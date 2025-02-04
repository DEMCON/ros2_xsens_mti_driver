cmake_minimum_required(VERSION 3.5)
project(ros2_xsens_mti_driver)

option(XSENS_MTI_RVIZ "Install rviz and urdf files" ON)

cmake_policy(SET CMP0057 NEW)

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

# Find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(tf2 REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(std_msgs REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(diagnostic_msgs REQUIRED)
find_package(diagnostic_updater REQUIRED)

###########
## Build ##
###########

add_subdirectory(lib/xspublic)

add_library(
  xsens_mti_utils
  STATIC
  src/xdautils.cpp
)

target_include_directories(
  xsens_mti_utils PUBLIC
  src
)

target_link_libraries(
	xsens_mti_utils
	xstypes
)

add_executable(
	xsens_mti_node
	src/main.cpp
  src/additional_logger_ros.cpp
	src/xdainterface.cpp
	src/xdacallback.cpp
)

target_include_directories(xsens_mti_node PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/lib/xspublic>
  $<INSTALL_INTERFACE:include>)

ament_target_dependencies(xsens_mti_node
  rclcpp
  tf2
  tf2_ros
  std_msgs
  geometry_msgs
  sensor_msgs
  diagnostic_msgs
  diagnostic_updater
)

target_link_libraries(
	xsens_mti_node
  xsens_mti_utils
	xscontroller
	xscommon
	xstypes
	pthread
	dl
)

install(TARGETS xsens_mti_node
  EXPORT export_${PROJECT_NAME}
  DESTINATION lib/${PROJECT_NAME}
)

install(DIRECTORY launch param
  DESTINATION share/${PROJECT_NAME}
)

if(XSENS_MTI_RVIZ)
  install(DIRECTORY rviz urdf
    DESTINATION share/${PROJECT_NAME}
  )
endif()

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
  add_subdirectory(test)
endif()

ament_package()
