cmake_minimum_required(VERSION 3.24)
project(pcmanager_smibios_proxy LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if (MSVC)
  add_definitions(-DUNICODE -D_UNICODE)
endif()

add_library(version SHARED
  src/version_proxy.cpp
  src/smbios_spoof.cpp
  src/smbios_spoof.hpp
  src/version.def
)

set_target_properties(version PROPERTIES
  OUTPUT_NAME "version"
)

find_package(minhook CONFIG REQUIRED)
target_link_libraries(version PRIVATE MinHook::minhook)

if (MSVC)
  target_compile_options(version PRIVATE /W4 /WX-)
endif()

add_executable(smbios_check tools/smbios_check.cpp)
target_link_libraries(smbios_check PRIVATE kernel32)
if (MSVC)
  target_compile_options(smbios_check PRIVATE /W4 /WX-)
endif()

foreach(tgt IN ITEMS version smbios_check)
  set_target_properties(${tgt} PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin"
    RUNTIME_OUTPUT_DIRECTORY_RELEASE "${CMAKE_BINARY_DIR}/bin/Release"
    RUNTIME_OUTPUT_DIRECTORY_DEBUG "${CMAKE_BINARY_DIR}/bin/Debug")
endforeach()
