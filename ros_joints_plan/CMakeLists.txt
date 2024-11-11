cmake_minimum_required(VERSION 3.8)
project(ros_joints_plan)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)
find_package(moveit_ros_planning_interface REQUIRED)
find_package(rclcpp REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(tf2 REQUIRED)
find_package(geometry_msgs REQUIRED)

set(ROS_JOINTS_PLAN_DEPENDENCIES
  moveit_ros_planning_interface
  rclcpp
)

if(UNIX)
  # Hack to link in missing/can't find libraries on linux
  execute_process(COMMAND bash "-c" "echo $(python3-config --ldflags --embed)" OUTPUT_VARIABLE BUILD_PYTHON_LINK_FLAGS)
  separate_arguments(FORCE_LINK_FLAGS UNIX_COMMAND ${BUILD_PYTHON_LINK_FLAGS})
  # list(APPEND FORCE_LINK_FLAGS
  # -static-libstdc++
  # )
  add_link_options(${FORCE_LINK_FLAGS})
endif()

add_executable(joints_plan src/joints_planning.cpp)
target_include_directories(joints_plan PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>)
target_compile_features(joints_plan PUBLIC c_std_99 cxx_std_17)  # Require C99 and C++17
ament_target_dependencies(joints_plan ${ROS_JOINTS_PLAN_DEPENDENCIES})

install(TARGETS joints_plan
  DESTINATION lib/${PROJECT_NAME})

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  # the following line skips the linter which checks for copyrights
  # comment the line when a copyright and license is added to all source files
  set(ament_cmake_copyright_FOUND TRUE)
  # the following line skips cpplint (only works in a git repo)
  # comment the line when this package is in a git repo and when
  # a copyright and license is added to all source files
  set(ament_cmake_cpplint_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()
endif()

install(DIRECTORY launch DESTINATION share/${PROJECT_NAME}
  PATTERN "setup_assistant.launch" EXCLUDE)

  ament_export_dependencies(${ROS_JOINTS_PLAN_DEPENDENCIES})
ament_package()
