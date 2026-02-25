cmake_minimum_required(VERSION 3.16)
project(orbslam3_ros2_vendor LANGUAGES C CXX)

find_package(ament_cmake REQUIRED)
include(ExternalProject)

set(PANGOLIN_SRC_DIR "${CMAKE_CURRENT_SOURCE_DIR}/third_party/Pangolin")
set(ORB_SLAM3_SRC_DIR "${CMAKE_CURRENT_SOURCE_DIR}/third_party/ORB_SLAM3")
set(PATCH_HELPER "${CMAKE_CURRENT_SOURCE_DIR}/cmake/apply_git_patch_idempotent.cmake")
set(PANGOLIN_PATCH "${CMAKE_CURRENT_SOURCE_DIR}/patches/pangolin/0001-dedup-fonts.patch")
set(ORB_SLAM3_PATCH "${CMAKE_CURRENT_SOURCE_DIR}/patches/orbslam3/0001-add-install-rules-and-link-targets.patch")
set(VENDOR_INSTALL_PREFIX "${CMAKE_INSTALL_PREFIX}")

foreach(_required_file IN ITEMS
    "${PATCH_HELPER}"
    "${PANGOLIN_PATCH}"
    "${ORB_SLAM3_PATCH}")
  if(NOT EXISTS "${_required_file}")
    message(FATAL_ERROR "Missing required file: ${_required_file}")
  endif()
endforeach()

if(NOT EXISTS "${PANGOLIN_SRC_DIR}/CMakeLists.txt")
  message(FATAL_ERROR
    "Missing Pangolin source at ${PANGOLIN_SRC_DIR}. "
    "Initialize this repo submodules or vendor the source into third_party/.")
endif()
if(NOT EXISTS "${ORB_SLAM3_SRC_DIR}/CMakeLists.txt")
  message(FATAL_ERROR
    "Missing ORB_SLAM3 source at ${ORB_SLAM3_SRC_DIR}. "
    "Initialize this repo submodules or vendor the source into third_party/.")
endif()

set(_vendor_common_cmake_args
  -DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>
  -DCMAKE_BUILD_TYPE=Release
  -DCMAKE_C_FLAGS=
  -DCMAKE_CXX_FLAGS=
  -DCMAKE_SHARED_LINKER_FLAGS=
  -DCMAKE_EXE_LINKER_FLAGS=
  -DBUILD_SHARED_LIBS=ON
)

ExternalProject_Add(pangolin_ep
  PREFIX "${CMAKE_CURRENT_BINARY_DIR}/pangolin_ep"
  SOURCE_DIR "${PANGOLIN_SRC_DIR}"
  BINARY_DIR "${CMAKE_CURRENT_BINARY_DIR}/pangolin-build"
  INSTALL_DIR "${VENDOR_INSTALL_PREFIX}"
  UPDATE_COMMAND ""
  PATCH_COMMAND
    ${CMAKE_COMMAND}
      -DSOURCE_DIR=${PANGOLIN_SRC_DIR}
      -DPATCH_FILE=${PANGOLIN_PATCH}
      -DPATCH_LABEL=Pangolin-dedup-fonts
      -P ${PATCH_HELPER}
  CMAKE_ARGS
    ${_vendor_common_cmake_args}
  BUILD_COMMAND ${CMAKE_COMMAND} --build <BINARY_DIR> --parallel
  INSTALL_COMMAND ${CMAKE_COMMAND} --build <BINARY_DIR> --target install --parallel
)

ExternalProject_Add(orbslam3_ep
  PREFIX "${CMAKE_CURRENT_BINARY_DIR}/orbslam3_ep"
  SOURCE_DIR "${ORB_SLAM3_SRC_DIR}"
  BINARY_DIR "${CMAKE_CURRENT_BINARY_DIR}/orbslam3-build"
  INSTALL_DIR "${VENDOR_INSTALL_PREFIX}"
  UPDATE_COMMAND ""
  PATCH_COMMAND
    ${CMAKE_COMMAND}
      -DSOURCE_DIR=${ORB_SLAM3_SRC_DIR}
      -DPATCH_FILE=${ORB_SLAM3_PATCH}
      -DPATCH_LABEL=ORB_SLAM3-install-rules
      -P ${PATCH_HELPER}
  CMAKE_ARGS
    ${_vendor_common_cmake_args}
    -DCMAKE_PREFIX_PATH=<INSTALL_DIR>
    -DORBROS_BUILD_ORB_SLAM3_EXAMPLES=OFF
  BUILD_COMMAND ${CMAKE_COMMAND} --build <BINARY_DIR> --parallel
  INSTALL_COMMAND ${CMAKE_COMMAND} --build <BINARY_DIR> --target install --parallel
  DEPENDS pangolin_ep
)

add_custom_target(${PROJECT_NAME}_vendor ALL
  DEPENDS pangolin_ep orbslam3_ep
)

ament_package()
