cmake_minimum_required(VERSION 3.5)
project(bytetrack)

find_package(autoware_cmake REQUIRED)
autoware_package()

find_package(OpenCV REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(OpenMP REQUIRED)
if(OpenMP_FOUND)
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif()
find_package(Boost REQUIRED)

#
# Core library
#
file(GLOB bytetrack_lib_src
  "lib/src/*.cpp"
)

ament_auto_add_library(bytetrack_lib SHARED
  ${bytetrack_lib_src}
)

target_include_directories(bytetrack_lib
  PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/lib/include>
  $<INSTALL_INTERFACE:include>
)

#
# ROS node
#
ament_auto_add_library(${PROJECT_NAME} SHARED
  src/bytetrack.cpp
)

ament_target_dependencies(${PROJECT_NAME}
  OpenCV
)

target_include_directories(${PROJECT_NAME}
  PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/lib/include>
  $<INSTALL_INTERFACE:include>
)

target_compile_definitions(${PROJECT_NAME} PRIVATE
  TENSORRT_VERSION_MAJOR=${TENSORRT_VERSION_MAJOR}
)

ament_auto_add_library(${PROJECT_NAME}_node SHARED
  src/bytetrack_node.cpp
)

ament_target_dependencies(${PROJECT_NAME}_node
  OpenCV
)

target_link_libraries(${PROJECT_NAME}_node
  ${PROJECT_NAME}
)

target_compile_definitions(${PROJECT_NAME}_node PRIVATE
  TENSORRT_VERSION_MAJOR=${TENSORRT_VERSION_MAJOR}
)

rclcpp_components_register_node(${PROJECT_NAME}_node
  PLUGIN "bytetrack::ByteTrackNode"
  EXECUTABLE ${PROJECT_NAME}_node_exe
)

#
# Visualizer
#
ament_auto_add_library(${PROJECT_NAME}_visualizer SHARED
  src/bytetrack_visualizer_node.cpp
)

ament_target_dependencies(${PROJECT_NAME}_visualizer
  OpenCV
)

rclcpp_components_register_node(${PROJECT_NAME}_visualizer
  PLUGIN "bytetrack::ByteTrackVisualizerNode"
  EXECUTABLE ${PROJECT_NAME}_visualizer_node_exe
)

ament_auto_package(INSTALL_TO_SHARE
  launch
)
