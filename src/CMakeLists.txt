# The version number.
set(UHTTP_VERSION_MAJOR 0)
set(UHTTP_VERSION_MINOR 2)
set(UHTTP_VERSION_PATCH 1)

# Check the third party Libraries
find_package(Libev REQUIRED)
find_package(OpenSSL)
find_package(CyaSSL)

include_directories(${LIBEV_INCLUDE_DIRS})
set(EXTRA_LIBS ${LIBEV_LIBRARIES})
set(SOURCE_FILES uhttp.c log.c buf.c ssl.c parser.c)

set(UHTTP_DEBUG_CONFIG 0)
option(UHTTP_DEBUG "Turn on debug" OFF)
if(UHTTP_DEBUG)
    set(UHTTP_DEBUG_CONFIG 1)
endif()

option(UHTTP_DISABLE_SSL "Disable ssl support" OFF)
option(UHTTP_USE_OPENSSL "Explicit use OpenSSL as SSL backend" OFF)
option(UHTTP_USE_CYASSL "Explicit use CyaSSL as SSL backend" OFF)

set(UHTTP_SSL_ENABLED OFF)
set(UHTTP_SSL_ENABLED_CONFIG 0)
set(UHTTP_USE_OPENSSL_CONFIG 0)
set(UHTTP_USE_CYASSL_CONFIG 0)

if(NOT UHTTP_DISABLE_SSL)
    if(UHTTP_USE_OPENSSL)
        if(NOT OPENSSL_FOUND)
            message(FATAL_ERROR "Could NOT find OpenSSL")
        endif()

        set(UHTTP_SSL_ENABLED ON)
        set(UHTTP_SSL_ENABLED_CONFIG 1)
        set(UHTTP_USE_OPENSSL_CONFIG 1)

        list(APPEND EXTRA_LIBS ${OPENSSL_LIBRARIES})
        include_directories(${OPENSSL_INCLUDE_DIR})
    elseif(UHTTP_USE_CYASSL)
        if (NOT CYASSL_FOUND)
            message(FATAL_ERROR "Could NOT find CyaSSL")
        endif()

        set(UHTTP_SSL_ENABLED ON)
        set(UHTTP_SSL_ENABLED_CONFIG 1)
        set(UHTTP_USE_CYASSL_CONFIG 1)

        list(APPEND EXTRA_LIBS ${CYASSL_LIBRARIES})
        include_directories(${CYASSL_INCLUDE_DIR})
    endif()

    if(NOT UHTTP_SSL_ENABLED)
        if(OPENSSL_FOUND)
            set(UHTTP_SSL_ENABLED ON)
            set(UHTTP_SSL_ENABLED_CONFIG 1)
            set(UHTTP_USE_OPENSSL_CONFIG 1)

            list(APPEND EXTRA_LIBS ${OPENSSL_LIBRARIES})
            include_directories(${OPENSSL_INCLUDE_DIR})
        elseif(CYASSL_FOUND)
            set(UHTTP_SSL_ENABLED ON)
            set(UHTTP_SSL_ENABLED_CONFIG 1)
            set(UHTTP_USE_CYASSL_CONFIG 1)

            list(APPEND EXTRA_LIBS ${CYASSL_LIBRARIES})
            include_directories(${CYASSL_INCLUDE_DIR})
        endif()
    endif()
endif()

add_library(uhttp SHARED ${SOURCE_FILES})
set_target_properties(uhttp PROPERTIES VERSION ${UHTTP_VERSION_MAJOR}.${UHTTP_VERSION_MINOR}.${UHTTP_VERSION_PATCH})
target_link_libraries(uhttp ${EXTRA_LIBS})

# configure a header file to pass some of the CMake settings to the source code
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/include/uhttp/config.h.in ${CMAKE_CURRENT_BINARY_DIR}/include/uhttp/config.h)

install(
    FILES
	    ${CMAKE_CURRENT_BINARY_DIR}/include/uhttp/config.h
	    ${CMAKE_CURRENT_SOURCE_DIR}/include/uhttp/uhttp.h
	    ${CMAKE_CURRENT_SOURCE_DIR}/include/uhttp/log.h
	    ${CMAKE_CURRENT_SOURCE_DIR}/include/uhttp/buf.h
        ${CMAKE_CURRENT_SOURCE_DIR}/include/uhttp/str.h
	    ${CMAKE_CURRENT_SOURCE_DIR}/include/uhttp/parser.h
    DESTINATION
    	include/uhttp
)

install(
    FILES
	    ${CMAKE_CURRENT_SOURCE_DIR}/include/uhttp.h
    DESTINATION
    	include
)

install(
    TARGETS uhttp LIBRARY
    DESTINATION lib
)

message("")
message(STATUS "UHTTP_VERSION: ${UHTTP_VERSION_MAJOR}.${UHTTP_VERSION_MINOR}.${UHTTP_VERSION_PATCH}")
if(UHTTP_USE_OPENSSL_CONFIG)
message(STATUS "UHTTP_SSL_ENABLED: OpenSSL")
elseif(UHTTP_USE_CYASSL_CONFIG)
message(STATUS "UHTTP_SSL_ENABLED: CyaSSL")
else()
message(STATUS "UHTTP_SSL_ENABLED: None")
endif()
message(STATUS "DEBUG: ${UHTTP_DEBUG}")
message("")
