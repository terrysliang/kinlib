cmake_minimum_required(VERSION 3.0.2)
project(kinlib)

## Compile as C++17
add_compile_options(-std=c++17)

set(KINLIB_TEST_RESOURCES_DIR "${CMAKE_CURRENT_LIST_DIR}/files")
configure_file("include/${PROJECT_NAME}_test/config.h.in" "${CATKIN_DEVEL_PREFIX}/include/${PROJECT_NAME}_test/config.h")
set(KINLIB_TEST_INCLUDE_DIR "${CATKIN_DEVEL_PREFIX}/include/")

set(assimp_DIR "/usr/lib/aarch64-linux-gnu/cmake/assimp-5.0/assimp-config.cmake")

find_package(PkgConfig REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(Assimp REQUIRED)
find_package(fcl REQUIRED)
find_package(Boost REQUIRED COMPONENTS system filesystem)
find_package(catkin REQUIRED COMPONENTS
  roscpp
  std_msgs
  geometry_msgs
  trajectory_msgs
  urdf
  kdl_parser
  eigen_conversions
)

pkg_check_modules(CCD REQUIRED ccd)
include_directories(${CCD_INCLUDE_DIRS})
link_directories(${CCD_LIBRARY_DIRS})
add_definitions(${CCD_CFLAGS_OTHER})


include_directories(
  include
  ${catkin_INCLUDE_DIRS}
  ${Eigen3_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS}
  ${FCL_INCLUDE_DIRS}
  ${CCD_INCLUDE_DIRS}
  ${ASSIMP_INCLUDE_DIRS}
)

add_library(${PROJECT_NAME}
  src/DualQuat.cpp
  src/manipulator.cpp
  src/kinlib_kinematics.cpp
  src/lemke.cpp
  src/collision_utils.cpp
)

target_link_libraries(${PROJECT_NAME}
  ${catkin_LIBRARIES}
  ${Boost_LIBRARIES}
  ${FCL_LIBRARIES} 
  ${CCD_LIBRARIES}
  ${ASSIMP_LIBRARIES}
)

catkin_package(
  INCLUDE_DIRS include
  LIBRARIES kinlib
  CATKIN_DEPENDS roscpp std_msgs geometry_msgs trajectory_msgs urdf kdl_parser eigen_conversions
  DEPENDS Eigen3 Boost fcl ccd
)

install(TARGETS ${PROJECT_NAME}
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
)

if(CATKIN_ENABLE_TESTING)
  include_directories(${KINLIB_TEST_INCLUDE_DIR})
  catkin_add_gtest(kinlib_test test/kinlib_test.cpp)
  target_link_libraries(kinlib_test ${PROJECT_NAME} ${catkin_LIBRARIES} armadillo)
  
  add_executable(kinlib_optime_test test/kinlib_optime_test.cpp)
  target_link_libraries(kinlib_optime_test ${PROJECT_NAME} ${catkin_LIBRARIES} armadillo)
endif()
