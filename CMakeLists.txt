cmake_minimum_required (VERSION 3.1)
project (OPENCL_ICD_LOADER VERSION 1.2 LANGUAGES C)
include (GNUInstallDirs)

# The option below allows building the ICD Loader library as a shared library
# (ON, default) or a static library (OFF).
#
# Khronos OpenCL Working Group strongly recommends building and using the ICD
# loader as a shared library due to the following benefits:
#
# 1. The shared library can be updated independent of the application. This
#    allows releasing new fixes and features in the ICD loader without updating
#    the application.
#
#    In rare cases when there are backward-incompatible changes to the ICD
#    loader (due to platform requirements, for instance), using a shared
#    library allows updating the library to make the transition seamless to
#    installed applications.
#
# 2. On platforms that require the ICD mechanism there are multiple vendors
#    shipping their OpenCL implementations. The vendor installers collaborate
#    to make sure that the installed ICD shared library version is suitable for
#    working with all vendor implementations installed on the system.
#
#    If applications statically link to ICD Loader then that version of the ICD
#    loader may not work with one or more installed vendor implementations.
#
# Using the OpenCL ICD loader as a static library is NOT recommended for
# end-user installations in general. However in some controlled environments it
# may be useful to simplify the build and distribution of the application. E.g.
# in test farms, or in cases where the end-user system configs are known in
# advance. Use it with discretion.
option (BUILD_SHARED_LIBS "Build shared libs" ON)

# This option enables support for OpenCL 3.0 Provisional in the ICD loader.  It
# is currently off by default while the specification is provisional, as it may
# change.
option (ENABLE_OPENCL30_PROVISIONAL "Enable 3.0 provisional entry points" OFF)

set (CMAKE_C_STANDARD 99)
set (CMAKE_C_STANDARD_REQUIRED ON)
set (CMAKE_C_EXTENSIONS OFF)

set (OPENCL_ICD_LOADER_HEADERS_DIR ${CMAKE_CURRENT_SOURCE_DIR}/inc CACHE PATH "Path to OpenCL Headers")

if (WIN32 AND NOT USE_DYNAMIC_VCXX_RUNTIME)
    string(REPLACE "/MD" "/MT" CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE}")
    string(REPLACE "/MD" "/MT" CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE}")
    string(REPLACE "/MD" "/MT" CMAKE_C_FLAGS_MINSIZEREL "${CMAKE_C_FLAGS_MINSIZEREL}")
    string(REPLACE "/MD" "/MT" CMAKE_CXX_FLAGS_MINSIZEREL "${CMAKE_CXX_FLAGS_MINSIZEREL}")
    string(REPLACE "/MD" "/MT" CMAKE_C_FLAGS_RELWITHDEBINFO "${CMAKE_C_FLAGS_RELWITHDEBINFO}")
    string(REPLACE "/MD" "/MT" CMAKE_CXX_FLAGS_RELWITHDEBINFO "${CMAKE_CXX_FLAGS_RELWITHDEBINFO}")
    string(REPLACE "/MDd" "/MTd" CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG}")
    string(REPLACE "/MDd" "/MTd" CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG}")
endif ()

add_subdirectory (loader)

include (CTest)
if (BUILD_TESTING)
    add_subdirectory (test)
endif()

export(EXPORT OpenCLTargets
  FILE "${PROJECT_BINARY_DIR}/OpenCL/OpenCLTargets.cmake"
  NAMESPACE OpenCL::)
configure_file(cmake/OpenCLConfig.cmake
  "${PROJECT_BINARY_DIR}/OpenCL/OpenCLConfig.cmake"
  COPYONLY)

set(config_package_location ${CMAKE_INSTALL_LIBDIR}/cmake/OpenCL)
install(EXPORT OpenCLTargets
  FILE OpenCLTargets.cmake
  NAMESPACE OpenCL::
  DESTINATION ${config_package_location})
install(
  FILES ${CMAKE_CURRENT_BINARY_DIR}/OpenCL/OpenCLConfig.cmake
  DESTINATION ${config_package_location}
  COMPONENT Devel)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/OpenCL/OpenCLConfigVersion.cmake"
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY AnyNewerVersion)
install(
  FILES "${CMAKE_CURRENT_BINARY_DIR}/OpenCL/OpenCLConfigVersion.cmake"
  DESTINATION ${config_package_location}
  COMPONENT Devel)
