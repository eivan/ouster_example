# ==== Requirements ====
find_package(Pcap REQUIRED)
find_package(libtins REQUIRED)

include(Coverage)

# ==== Libraries ====
add_library(ouster_pcap src/pcap.cpp src/os_pcap.cpp src/indexed_pcap_reader.cpp src/ip_reassembler.cpp)
target_include_directories(ouster_pcap SYSTEM PRIVATE
  ${PCAP_INCLUDE_DIR})
target_include_directories(ouster_pcap PUBLIC
  $<INSTALL_INTERFACE:include>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>)
CodeCoverageFunctionality(ouster_pcap)

if(WIN32)
  target_link_libraries(ouster_pcap PUBLIC ws2_32)
endif()
target_link_libraries(ouster_pcap
  PUBLIC
    OusterSDK::ouster_client
  PRIVATE libpcap::libpcap libtins::libtins)
add_library(OusterSDK::ouster_pcap ALIAS ouster_pcap)

if(WIN32)
  set_property (TARGET ouster_pcap PROPERTY WINDOWS_EXPORT_ALL_SYMBOLS ON)
endif()

# ==== Install ====
install(TARGETS ouster_pcap
  EXPORT ouster-sdk-targets
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include)

install(DIRECTORY include/ouster DESTINATION include)
