cmake_minimum_required(VERSION 3.14)
project(vikit_common)

# Default to C++14
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Build type
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

# Compiler flags
add_compile_options(
  -Wall -D_LINUX -D_REENTRANT -march=native
  -Wno-unused-variable -Wno-unused-but-set-variable -Wno-unknown-pragmas
)

if(CMAKE_SYSTEM_PROCESSOR MATCHES "(x86)|(X86)|(amd64)|(AMD64)")
  add_compile_options(-mmmx -msse -msse2 -msse3 -mssse3)
else()
  add_compile_options(-march=armv8-a)
endif()

# Dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(OpenCV REQUIRED)
find_package(Sophus REQUIRED)
find_package(fmt REQUIRED)

message(WARNING ${EIGEN3_INCLUDE_DIR})

# Source files
set(SOURCEFILES
  src/atan_camera.cpp
  src/omni_camera.cpp
  src/math_utils.cpp
  src/vision.cpp
  src/performance_monitor.cpp
  src/robust_cost.cpp
  src/user_input_thread.cpp
  src/pinhole_camera.cpp
  src/equidistant_camera.cpp
  src/polynomial_camera.cpp
  src/homography.cpp
  src/img_align.cpp
)

# Library
add_library(${PROJECT_NAME} SHARED ${SOURCEFILES})

ament_target_dependencies(${PROJECT_NAME}
  rclcpp
  Eigen3
  OpenCV
  Sophus
  fmt
)

target_include_directories(${PROJECT_NAME} PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>  # Solo per la build
  $<BUILD_INTERFACE:/opt/include/ros/jazzy>
  $<INSTALL_INTERFACE:include>  # Per l'installazione
)

ament_export_targets(export_${PROJECT_NAME} HAS_LIBRARY_TARGET)
ament_export_dependencies(rclcpp Eigen3 OpenCV Sophus fmt)

target_link_libraries(${PROJECT_NAME}
    Eigen3::Eigen
    Sophus::Sophus
)

# Executables (tests)
#add_executable(test_vk_common_camera test/test_camera.cpp)
#target_link_libraries(test_vk_common_camera ${PROJECT_NAME})
#
#add_executable(test_vk_common_triangulation test/test_triangulation.cpp)
#target_link_libraries(test_vk_common_triangulation ${PROJECT_NAME})
#
#add_executable(test_vk_common_patch_score test/test_patch_score.cpp)
#target_link_libraries(test_vk_common_patch_score ${PROJECT_NAME})
#
# Install headers and library
install(DIRECTORY include/
  DESTINATION include
)

install(TARGETS ${PROJECT_NAME}
  EXPORT export_${PROJECT_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

#install(TARGETS
#  test_vk_common_camera
#  test_vk_common_triangulation
#  test_vk_common_patch_score
#  DESTINATION lib/${PROJECT_NAME}
#)

# Export

ament_package()
