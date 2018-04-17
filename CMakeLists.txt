cmake_minimum_required(VERSION 3.0.0)

project(dslink-c-iec61850 VERSION 1.0.1 LANGUAGES C)


add_subdirectory("${CMAKE_CURRENT_LIST_DIR}/libiec61850")
include_directories("${CMAKE_CURRENT_BINARY_DIR}/libiec61850/config")
include_directories("${CMAKE_CURRENT_LIST_DIR}/libiec61850/src/iec61850/inc")
include_directories("${CMAKE_CURRENT_LIST_DIR}/libiec61850/src/common/inc")
include_directories("${CMAKE_CURRENT_LIST_DIR}/libiec61850/src/hal/inc")
include_directories("${CMAKE_CURRENT_LIST_DIR}/libiec61850/src/mms/inc")
include_directories("${CMAKE_CURRENT_LIST_DIR}/libiec61850/src/logging")

set(DSLINK_BUILD_BROKER OFF CACHE BOOL "Whether to build the broker" FORCE)
add_subdirectory("${CMAKE_CURRENT_LIST_DIR}/dslink")
include_directories("${CMAKE_CURRENT_BINARY_DIR}/dslink/include")
include_directories("${CMAKE_CURRENT_LIST_DIR}/dslink/sdk/include")
include_directories("${CMAKE_CURRENT_LIST_DIR}/dslink/deps/libuv/include")
include_directories("${CMAKE_CURRENT_LIST_DIR}/dslink/deps/mbedtls/include")
include_directories("${CMAKE_CURRENT_LIST_DIR}/dslink/deps/jansson/src")

include_directories("${CMAKE_CURRENT_LIST_DIR}/include")

include_directories(".")

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=gnu99")

set(SRC "src/main.c" "src/add_server.c" "src/client_handler.c" "src/server_status.c" "src/iec61850_server_defs.c" "src/time_iso8601.c" "src/dataset_handler.c" "src/report_handler.c")

add_executable(dslink-c-iec61850 ${SRC})
add_dependencies(dslink-c-iec61850 sdk_dslink_c-static)
target_link_libraries(dslink-c-iec61850 sdk_dslink_c-static)
target_link_libraries(dslink-c-iec61850 iec61850)

install(TARGETS dslink-c-iec61850 RUNTIME DESTINATION bin)

add_custom_command(OUTPUT dslink.json
                   COMMAND cp ${CMAKE_CURRENT_SOURCE_DIR}/resources/dslink.json.template dslink.json
                   COMMAND sed -i -e 's/@PROJECT_VERSION/${PROJECT_VERSION}/' dslink.json
                   DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/resources/dslink.json.template ${CMAKE_CURRENT_LIST_FILE}
                   )
# Create a custom target to trigger the custom build step and add the generated file to the ALL target
add_custom_target(package-config ALL DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/dslink.json)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/dslink.json DESTINATION . COMPONENT config)

# Create package information
set(CPACK_GENERATOR "ZIP")
set(CPACK_INCLUDE_TOPLEVEL_DIRECTORY 0)
set(CPACK_PACKAGE_FILE_NAME "${PROJECT_NAME}_${dslink-c-iec61850_VERSION}_${ARG_CPUARCH}")
include(CPack)
