# ==== Requirements ====
find_package(Eigen3 REQUIRED)
find_package(jsoncpp REQUIRED)
find_package(CURL REQUIRED)
find_package(spdlog REQUIRED)
include(Coverage)
include(GenerateExportHeader)

# ==== Libraries ====
add_library(ouster_client src/client.cpp src/types.cpp src/sensor_info.cpp src/netcompat.cpp src/lidar_scan.cpp
  src/image_processing.cpp src/udp_packet_source.cpp src/parsing.cpp
  src/sensor_http.cpp src/sensor_http_imp.cpp src/sensor_tcp_imp.cpp src/logging.cpp
  src/field.cpp src/profile_extension.cpp src/util.cpp)
target_link_libraries(ouster_client
  PUBLIC
    Eigen3::Eigen
    jsoncpp_lib
    $<BUILD_INTERFACE:ouster_build>
    spdlog::spdlog
  PRIVATE
    CURL::libcurl)
target_compile_definitions(ouster_client PRIVATE EIGEN_MPL2_ONLY)
CodeCoverageFunctionality(ouster_client)

add_library(OusterSDK::ouster_client ALIAS ouster_client)

# If ouster_client is built as >=c++17, the nonstd::optional backport
# will just be an alias for std::optional. In that case, client code
# must also build as c++17 to use the same implementation of optional
get_target_property(OUSTER_CLIENT_CXX_STANDARD ouster_client CXX_STANDARD)
if(OUSTER_CLIENT_CXX_STANDARD GREATER_EQUAL 17)
  target_compile_features(ouster_client INTERFACE cxx_std_17)
endif()

if(WIN32)
  target_link_libraries(ouster_client PUBLIC ws2_32)
endif()
target_include_directories(ouster_client PUBLIC
  $<INSTALL_INTERFACE:include>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>)
target_include_directories(ouster_client SYSTEM PUBLIC
  $<INSTALL_INTERFACE:include/optional-lite>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/optional-lite>)

# ==== Export header ====

set(ouster_client_export_filename
  ouster_client/ouster/ouster_client_export.h)

generate_export_header(ouster_client 
  EXPORT_FILE_NAME ${PROJECT_BINARY_DIR}/${ouster_client_export_filename})

target_include_directories(ouster_client PUBLIC
  $<BUILD_INTERFACE:${PROJECT_BINARY_DIR}/ouster_client>)

install(FILES ${PROJECT_BINARY_DIR}/${ouster_client_export_filename} DESTINATION include/ouster)
  
# ==== Install ====
install(TARGETS ouster_client
        EXPORT ouster-sdk-targets
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
        RUNTIME DESTINATION bin
        INCLUDES DESTINATION include)

install(DIRECTORY include/ouster include/optional-lite DESTINATION include)
