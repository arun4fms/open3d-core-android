# setting
ln -s /home/{userid}/Android/Sdk/ndk/20.1.5948944/ /home/{userid}/Android/Sdk/ndk-bundle

# Change 
1. build.bat ( use default ndk path)

# Fork updates

The update that made in this fork: 
1. Open3D version for 0.9.0. 
For 0.9.0, it has PossionRecon-android.patch for the submodule PossionRecon.
2. The original repo removes Tools and Visualization, in this fork IO module is removed since some external libraries are hard to compile (turbojpeg).
3. The abi includes armeabi-v7a and arm64-v8a. 

# Building Open3D core for Android

CMake scripts for cross-compiling the core library of [Open3D](http://www.open3d.org/) for Android and linking to it with Android Studio.

Tested with:

- Windows 10 1809
- CMake 3.14.1
- Android Studio 3.4.1 with Gradle plugin 3.4.1
- Bundled Android NDK r20
- Open3D 0.7.0

Should work on Linux hosts, too, but it's untested.

## Patches

A lot of Open3D revolves around visualisation. Since that code relies on OpenGL and X11 (both of which don't exist on Android) I removed it from being compiled and included. The patches to its CMakeLists.txt can be found in [open3d-android.patch](open3d-android.patch). Essentially it:

- prevents GLEW and GLFW from being built and linked
- removes the Visualization and Tools modules

The patch file itself was created with `git diff > open3d-android.patch`.

## Compiling

You will need [Ninja](https://ninja-build.org/) on your system (and in the PATH variable).

To cross-compile Open3D, simply:
```
mkdir build & cd build
cmake -G Ninja -DCMAKE_INSTALL_PREFIX=../install .. && cmake --build .
```
This downloads a specific revision of Open3D and compiles it for all supported NDK ABIs.

- To change the version of Open3D change the `GIT_TAG` in the call to `ExternalProject_Add` for *open3d-fetch*. You may have to change Open3D's CMakeLists.txt's and create an updated patch file.
- To specify an NDK path, call CMake with -DANDROID_NDK=path/to/ndk. By default it locates the NDK bundled with Android Studio.

When it's finished, you'll find compiled versions for each Android ABI in the install folder.

## Linking

To use the library in Android Studio, take a look at [the template CMakeLists.txt](android-studio/CMakeLists.txt).
It defines a sample library that finds and links to Open3D. It requires two additional steps:

1. Set `OPEN3D_PATH` to the Open3D install. This can be done by [specifying an extra parameter to CMake](https://developer.android.com/ndk/guides/cmake#variables) in your build.gradle:
```groovy
android {
  ...
  defaultConfig {
    ...
    externalNativeBuild {
      cmake {
        ...
        arguments "-DOPEN3D_PATH=path/to/Open3D/install"
      }
    }
  }
}
```

2. Add the shared libraries. The template CMakeLists.txt copies the appropriate shared libraries to their own ABI folders under libs/ in the main source directory. To make Gradle find and package them, [add them to the source set](https://developer.android.com/studio/projects/gradle-external-native-builds#jniLibs):
```groovy
android {
  ...
  sourceSets {
    main {
      jniLibs.srcDirs = ['libs']
    }
  }
}
```
