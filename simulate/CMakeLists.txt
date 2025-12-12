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

# Fetch unitree_sdk2
set(BUILD_EXAMPLES OFF CACHE BOOL "Build examples" FORCE)
FetchContent_Declare(
  unitree_sdk2
  GIT_REPOSITORY https://github.com/unitreerobotics/unitree_sdk2.git
  GIT_TAG 4f8aca7bbc269d275d45a7917d23b0386f74fb1c
  CMAKE_ARGS -DCMAKE_BUILD_TYPE=Release -DBUILD_EXAMPLES=OFF
)
FetchContent_MakeAvailable(unitree_sdk2)
message(STATUS "Unitree SDK2 source dir: ${unitree_sdk2_SOURCE_DIR}")

# Install unitree_sdk2
install(DIRECTORY ${unitree_sdk2_SOURCE_DIR}/lib/
    DESTINATION ${CMAKE_INSTALL_LIBDIR}
    FILES_MATCHING PATTERN "*.so*" PATTERN "*.a"
)

install(DIRECTORY ${unitree_sdk2_SOURCE_DIR}/include/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(DIRECTORY ${unitree_sdk2_SOURCE_DIR}/thirdparty/include/ddscxx/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)
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
