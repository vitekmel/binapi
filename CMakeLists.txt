cmake_minimum_required(VERSION 3.2)
project(binapi)

set(CMAKE_CXX_STANDARD 20)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -fsanitize=address")

add_definitions(
    -UNDEBUG
    -DDTF_HEADER_ONLY
)

include_directories(
    ./include
)

find_package(Boost REQUIRED COMPONENTS system filesystem)
include_directories(${Boost_INCLUDE_DIRS})
link_directories(${Boost_LIBRARY_DIRS})

#find_package(OpenSSL REQUIRED)

find_package(OpenSSL QUIET)
if(NOT OpenSSL_FOUND)
    message(STATUS "Looking for OpenSSL in common locations")
    find_path(OPENSSL_ROOT_DIR
            NAMES include/openssl/ssl.h
            PATHS "/usr/local/opt/openssl" "/opt/homebrew/opt/openssl@3"
            )
    if(OPENSSL_ROOT_DIR)
        message(STATUS "Found OpenSSL in ${OPENSSL_ROOT_DIR}")
        find_package(OpenSSL REQUIRED)
    else()
        message(FATAL_ERROR "OpenSSL not found")
    endif()
endif()

include_directories(${OPENSSL_INCLUDE_DIR})
message(${OPENSSL_FOUND})

get_filename_component(OPENSSL_LIB_DIR "${OPENSSL_CRYPTO_LIBRARY}" DIRECTORY)
link_directories(${OPENSSL_LIB_DIR})

set(BINAPI_HEADERS
    binapi/api.hpp
    binapi/flatjson.hpp
    binapi/dtf.hpp
    binapi/double_type.hpp
    binapi/enums.hpp
    binapi/errors.hpp
    binapi/invoker.hpp
    binapi/message.hpp
    binapi/pairslist.hpp
    binapi/reports.hpp
    binapi/tools.hpp
    binapi/types.hpp
    binapi/websocket.hpp
)

set(BINAPI_SOURCES
    src/api.cpp
    src/enums.cpp
    src/errors.cpp
    src/pairslist.cpp
    src/reports.cpp
    src/tools.cpp
    src/types.cpp
    src/websocket.cpp
)

add_executable(${PROJECT_NAME} main.cpp ${BINAPI_SOURCES})

target_link_libraries(${PROJECT_NAME} ${Boost_LIBRARIES})
target_link_libraries(${PROJECT_NAME} OpenSSL::SSL OpenSSL::Crypto)
target_link_libraries(${PROJECT_NAME} z crypto ssl pthread)
