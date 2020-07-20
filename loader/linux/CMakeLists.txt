# By default don't include OpenCL 3.0 symbol definitions (i.e. comment them
# out), but include them for OpenCL 3.0 builds.  Once the symbols are no
# longer provisional then they may be included unconditionally.
set(ENABLE_OPENCL30_SYMBOLS_START "/*")
set(ENABLE_OPENCL30_SYMBOLS_END "*/")
if (ENABLE_OPENCL30_PROVISIONAL)
  set(ENABLE_OPENCL30_SYMBOLS_START "")
  set(ENABLE_OPENCL30_SYMBOLS_END "")
endif ()
configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/loader/linux/icd_exports.map.in
    ${CMAKE_CURRENT_BINARY_DIR}/loader/linux/icd_exports.map)

target_sources(OpenCL PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/icd_linux.c
    ${CMAKE_CURRENT_SOURCE_DIR}/icd_linux_envvars.c
    ${CMAKE_CURRENT_BINARY_DIR}/icd_exports.map
)

target_include_directories (OpenCL PRIVATE ${PROJECT_BINARY_DIR})

target_compile_definitions(OpenCL PRIVATE _DEFAULT_SOURCE)
set_property(TARGET OpenCL APPEND PROPERTY
    LINK_FLAGS -Wl,--version-script="${CMAKE_CURRENT_BINARY_DIR}/icd_exports.map"
)
set_property(TARGET OpenCL APPEND PROPERTY
    LINK_DEPENDS "${CMAKE_CURRENT_BINARY_DIR}/icd_exports.map"
)

