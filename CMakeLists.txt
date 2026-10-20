cmake_minimum_required(VERSION 3.8)
project(amiga_description)
find_package(ament_cmake REQUIRED)
install(DIRECTORY urdf meshes launch rviz DESTINATION share/${PROJECT_NAME})
# Gazebo resolves package://amiga_description/... through GZ_SIM_RESOURCE_PATH
ament_environment_hooks("${CMAKE_CURRENT_SOURCE_DIR}/hooks/${PROJECT_NAME}.dsv.in")
ament_package()
