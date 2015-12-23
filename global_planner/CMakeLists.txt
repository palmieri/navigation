cmake_minimum_required(VERSION 2.8.3)
project(global_planner)


# Adding definitions for maximum optimization by gcc
MESSAGE(STATUS "Compiling with maximum optimization, will only work on Core i7 processors! (-mavx -march=corei7-avx -msse4.2 -O3 -funroll-loops -fomit-frame-pointer -ftree-vectorize ...)")
ADD_DEFINITIONS(-march=corei7-avx)
ADD_DEFINITIONS(-mtune=corei7-avx)
ADD_DEFINITIONS(-msse)
ADD_DEFINITIONS(-msse2)
ADD_DEFINITIONS(-msse3)
ADD_DEFINITIONS(-mssse3)
ADD_DEFINITIONS(-msse4.1)
ADD_DEFINITIONS(-msse4.2)
IF($ENV{AVX})
  ADD_DEFINITIONS(-mavx)
  MESSAGE(STATUS "Compiling with AVX on")
ELSE()
  MESSAGE(WARNING "Compiling with AVX off")
  ADD_DEFINITIONS(-mno-avx)
ENDIF()
ADD_DEFINITIONS(-mno-aes)
ADD_DEFINITIONS(-O3)
ADD_DEFINITIONS(-funroll-loops)
ADD_DEFINITIONS(-fomit-frame-pointer)
ADD_DEFINITIONS(-pipe)
ADD_DEFINITIONS(-ftree-vectorize)
ADD_DEFINITIONS(-DNDEBUG)
# set(CMAKE_BUILD_TYPE RelWithDebInfo)
#######


find_package(catkin REQUIRED
  COMPONENTS
    angles
    costmap_2d
    dynamic_reconfigure
    geometry_msgs
    nav_core
    navfn
    nav_msgs
    pluginlib
    roscpp
    tf
)

generate_dynamic_reconfigure_options(
  cfg/GlobalPlanner.cfg
)

catkin_package(
  INCLUDE_DIRS include
  LIBRARIES ${PROJECT_NAME}
  CATKIN_DEPENDS
    costmap_2d
    dynamic_reconfigure
    geometry_msgs
    nav_core
    navfn
    nav_msgs
    pluginlib
    roscpp
    tf
)

include_directories(
  include
  ${catkin_INCLUDE_DIRS}
)

add_library(${PROJECT_NAME}
  src/quadratic_calculator.cpp
  src/dijkstra.cpp
  src/astar.cpp
  src/grid_path.cpp
  src/gradient_path.cpp
  src/orientation_filter.cpp
  src/planner_core.cpp
)
target_link_libraries(${PROJECT_NAME} ${catkin_LIBRARIES})

add_dependencies(${PROJECT_NAME} ${PROJECT_NAME}_gencfg)

add_executable(planner
  src/plan_node.cpp
)
target_link_libraries(planner
  ${PROJECT_NAME}
  ${catkin_LIBRARIES}
)

install(TARGETS ${PROJECT_NAME} planner
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})

install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  PATTERN ".svn" EXCLUDE)

install(FILES bgp_plugin.xml
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})
