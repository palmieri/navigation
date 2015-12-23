cmake_minimum_required(VERSION 2.8.3)
project(move_base)

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
        cmake_modules
        roscpp
        pluginlib
        actionlib
        dynamic_reconfigure
        message_generation
        nav_core
        tf
)
find_package(Eigen REQUIRED)
add_definitions(${EIGEN_DEFINITIONS})

# dynamic reconfigure
generate_dynamic_reconfigure_options(
    cfg/MoveBase.cfg
)

catkin_package(
    CATKIN_DEPENDS
        roscpp
        dynamic_reconfigure
)

include_directories(
    include
    ${catkin_INCLUDE_DIRS}
    ${EIGEN_INCLUDE_DIRS}
)

# move_base
add_library(move_base
  src/move_base.cpp
)
target_link_libraries(move_base
    ${Boost_LIBRARIES}
    ${catkin_LIBRARIES}
    )
add_dependencies(move_base move_base_gencfg)
add_dependencies(move_base geometry_msgs_gencpp)

add_executable(move_base_node
  src/move_base_node.cpp
)
target_link_libraries(move_base_node move_base)
set_target_properties(move_base_node PROPERTIES OUTPUT_NAME move_base)

install(DIRECTORY launch
    DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
    USE_SOURCE_PERMISSIONS
    )

install(
    PROGRAMS
       scripts/subtopic_forwarder.py
       scripts/subtopic_forwarder_node.py
       scripts/warner.py
    DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(
    TARGETS
        move_base
        move_base_node
    ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

## Mark cpp header files for installation
install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.h"
)
