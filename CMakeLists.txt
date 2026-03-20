cmake_minimum_required(VERSION 3.16)
project(rip_checker VERSION 1.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# -----------------------------------------------------------------------
# vcpkg integration (optional but recommended)
# If using vcpkg, pass: -DCMAKE_TOOLCHAIN_FILE=<vcpkg>/scripts/buildsystems/vcpkg.cmake
# Then install curl: vcpkg install curl
# -----------------------------------------------------------------------

find_package(CURL REQUIRED)

add_executable(rip_checker
    src/main.cpp
    src/csv_reader.cpp
    src/rip_scraper.cpp
    src/result_writer.cpp
)

target_include_directories(rip_checker PRIVATE src)
target_link_libraries(rip_checker PRIVATE CURL::libcurl)

# Windows: link to ws2_32 (Winsock) required by curl
if(WIN32)
    target_link_libraries(rip_checker PRIVATE ws2_32 wldap32 crypt32)
endif()

# Optional: enable warnings
if(MSVC)
    target_compile_options(rip_checker PRIVATE /W4)
else()
    target_compile_options(rip_checker PRIVATE -Wall -Wextra -Wpedantic)
endif()

# Copy executable and required DLLs to project root after build for convenience
add_custom_command(TARGET rip_checker POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:rip_checker> ${CMAKE_SOURCE_DIR}
    COMMENT "Copying rip_checker to project root"
)

if(WIN32)
    # Copy runtime DLLs (libcurl, zlib) so the exe runs without PATH setup
    add_custom_command(TARGET rip_checker POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
            $<TARGET_RUNTIME_DLLS:rip_checker>
            ${CMAKE_SOURCE_DIR}
        COMMAND_EXPAND_LISTS
        COMMENT "Copying runtime DLLs to project root"
    )
endif()
