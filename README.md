# orbslam3_ros2_vendor

ROS 2 `ament_cmake` vendor package that builds Pangolin and ORB_SLAM3 into this package install prefix.

## What it installs (minimal SDK for wrapper)

- `${prefix}/include/**` (Pangolin + ORB_SLAM3 public headers + Sophus headers required by ORB headers)
- `${prefix}/lib/libpangolin*.so` + `${prefix}/lib/cmake/Pangolin/*`
- `${prefix}/lib/libORB_SLAM3.so`
- `${prefix}/lib/libDBoW2.so`
- `${prefix}/lib/libg2o.so`

## Expected source layout

Populate:

- `third_party/Pangolin`
- `third_party/ORB_SLAM3`

You can do this with git submodules in this repo, or by vendoring the source trees directly.

## Build flags isolation

Both ExternalProjects force:

- `Release`
- empty `CMAKE_C_FLAGS`
- empty `CMAKE_CXX_FLAGS`
- empty `CMAKE_SHARED_LINKER_FLAGS`
- empty `CMAKE_EXE_LINKER_FLAGS`

This is intentional to prevent sanitizer flag leakage from the wrapper build into vendor binaries.

## System packages (Ubuntu)

Install OpenGL/X11 deps with apt (intentionally not listed in `package.xml`):

```bash
sudo apt update
sudo apt install -y \
  libeigen3-dev \
  libopencv-dev \
  libepoxy-dev \
  libglew-dev \
  libgl1-mesa-dev \
  libx11-dev \
  libxext-dev \
  libxrandr-dev \
  libxi-dev \
  libxinerama-dev \
  libxcursor-dev
```
