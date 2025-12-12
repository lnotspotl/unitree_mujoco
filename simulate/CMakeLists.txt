cmake_minimum_required(VERSION 3.16)
project(unitree_mujoco)

include(FetchContent)

if (CMAKE_BUILD_TYPE MATCHES "Debug")
  message("Debug mode")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17 -g -O0 -fPIC")
else()
  message("Release mode")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17 -O3 -DNDEBUG -fPIC")
endif()

# Fetch MuJoCo pre-built release
set(MUJOCO_VERSION "3.3.6")
set(MUJOCO_URL "https://github.com/google-deepmind/mujoco/releases/download/${MUJOCO_VERSION}/mujoco-${MUJOCO_VERSION}-linux-x86_64.tar.gz")

FetchContent_Declare(
  mujoco
  URL ${MUJOCO_URL}
  DOWNLOAD_EXTRACT_TIMESTAMP TRUE
)

FetchContent_MakeAvailable(mujoco)

# Set MuJoCo paths from fetched content
set(MUJOCO_DIR ${mujoco_SOURCE_DIR})
set(MUJOCO_INCLUDE_DIR ${MUJOCO_DIR}/include)
set(MUJOCO_LIB_DIR ${MUJOCO_DIR}/lib)
set(MUJOCO_LIB ${MUJOCO_LIB_DIR}/libmujoco.so.${MUJOCO_VERSION})

message(STATUS "MuJoCo source dir: ${MUJOCO_DIR}")
message(STATUS "MuJoCo include dir: ${MUJOCO_INCLUDE_DIR}")
message(STATUS "MuJoCo lib: ${MUJOCO_LIB}")

list(APPEND CMAKE_PREFIX_PATH "/opt/unitree_robotics/lib/cmake")
find_package(unitree_sdk2 REQUIRED)
find_package(Boost REQUIRED COMPONENTS program_options)

include_directories(
  ${MUJOCO_INCLUDE_DIR}
  ${MUJOCO_INCLUDE_DIR}/mujoco
  ${MUJOCO_DIR}/simulate
  src/lodepng
)

file(GLOB SIM_SRC
  src/joystick/joystick.cc

  # exclude mujoco/simulate/main.cc
  ${MUJOCO_DIR}/simulate/glfw_*.cc
  ${MUJOCO_DIR}/simulate/platform_*.cc
  ${MUJOCO_DIR}/simulate/simulate.cc
  src/lodepng/lodepng.cpp
)

link_libraries(
  pthread
  glfw
  yaml-cpp
  unitree_sdk2
  boost_program_options
  fmt
)

add_executable(unitree_mujoco ${SIM_SRC} src/main.cc)
target_link_libraries(unitree_mujoco ${MUJOCO_LIB})

add_executable(jstest src/joystick/jstest.cc src/joystick/joystick.cc)