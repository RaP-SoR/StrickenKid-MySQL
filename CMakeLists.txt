cmake_minimum_required(VERSION 3.20)
project(StrickenKid-MySQL LANGUAGES C CXX)

# SA-MP plugins are 32-bit only.
if(CMAKE_SIZEOF_VOID_P EQUAL 8)
  message(FATAL_ERROR
    "This project must be configured as 32-bit (Win32 / -m32). "
    "Example: cmake -S . -B build -G \"Visual Studio 17 2022\" -A Win32")
endif()

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)

set(PLUGIN_OUTPUT_NAME "mysql")

# --- MariaDB Connector/C 3.4.5 (built as subproject) ---
# Connector/C only adds these when not a subproject; without them Winsock1/2 collide.
if(WIN32)
  add_compile_definitions(WIN32_LEAN_AND_MEAN NOGDI)
endif()

set(WITH_UNIT_TESTS OFF CACHE BOOL "" FORCE)
set(WITH_CURL OFF CACHE BOOL "" FORCE)
set(WITH_MYSQLCOMPAT OFF CACHE BOOL "" FORCE)
set(WITH_DYNCOL ON CACHE BOOL "" FORCE)
set(DEFAULT_SSL_VERIFY_SERVER_CERT OFF CACHE BOOL "" FORCE)
# Avoid empty INSTALL_PLUGINDIR when Connector/C is a subproject.
set(INSTALL_PLUGINDIR "lib/mariadb/plugin" CACHE STRING "" FORCE)
# Prefer static auth/IO plugins so the SA-MP plugin is self-contained.
set(CLIENT_PLUGIN_CACHING_SHA2_PASSWORD STATIC CACHE STRING "" FORCE)
set(CLIENT_PLUGIN_SHA256_PASSWORD STATIC CACHE STRING "" FORCE)
set(CLIENT_PLUGIN_CLIENT_ED25519 STATIC CACHE STRING "" FORCE)
set(CLIENT_PLUGIN_MYSQL_CLEAR_PASSWORD STATIC CACHE STRING "" FORCE)
set(CLIENT_PLUGIN_DIALOG OFF CACHE STRING "" FORCE)
set(CLIENT_PLUGIN_AUTH_GSSAPI_CLIENT OFF CACHE STRING "" FORCE)
set(CLIENT_PLUGIN_PARSEC OFF CACHE STRING "" FORCE)
set(CLIENT_PLUGIN_REMOTEIO OFF CACHE STRING "" FORCE)
set(CLIENT_PLUGIN_PVIO_SOCKET STATIC CACHE STRING "" FORCE)
set(CLIENT_PLUGIN_PVIO_NPIPE STATIC CACHE STRING "" FORCE)
set(CLIENT_PLUGIN_PVIO_SHMEM STATIC CACHE STRING "" FORCE)

add_subdirectory(deps/mariadb-connector-c EXCLUDE_FROM_ALL)

set(SAMP_SDK_DIR "${CMAKE_CURRENT_SOURCE_DIR}/deps/samp-plugin-sdk")

set(PLUGIN_SOURCES
  MySQL/Base.cpp
  MySQL/Main.cpp
  MySQL/Mysql.cpp
  MySQL/Natives.cpp
  MySQL/Thread.cpp
  ${SAMP_SDK_DIR}/amxplugin.cpp
  ${SAMP_SDK_DIR}/amxplugin2.cpp
)

add_library(mysql_plugin SHARED ${PLUGIN_SOURCES})
set_target_properties(mysql_plugin PROPERTIES
  OUTPUT_NAME "${PLUGIN_OUTPUT_NAME}"
  PREFIX ""
  RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin"
  LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin"
  ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin"
  RUNTIME_OUTPUT_DIRECTORY_DEBUG "${CMAKE_BINARY_DIR}/bin"
  RUNTIME_OUTPUT_DIRECTORY_RELEASE "${CMAKE_BINARY_DIR}/bin"
  RUNTIME_OUTPUT_DIRECTORY_RELWITHDEBINFO "${CMAKE_BINARY_DIR}/bin"
  RUNTIME_OUTPUT_DIRECTORY_MINSIZEREL "${CMAKE_BINARY_DIR}/bin"
  LIBRARY_OUTPUT_DIRECTORY_DEBUG "${CMAKE_BINARY_DIR}/bin"
  LIBRARY_OUTPUT_DIRECTORY_RELEASE "${CMAKE_BINARY_DIR}/bin"
  LIBRARY_OUTPUT_DIRECTORY_RELWITHDEBINFO "${CMAKE_BINARY_DIR}/bin"
  LIBRARY_OUTPUT_DIRECTORY_MINSIZEREL "${CMAKE_BINARY_DIR}/bin"
)

# Do not add MySQL/ to the include path: on Windows Mysql.h collides with
# MariaDB's mysql.h (case-insensitive filesystem).
target_include_directories(mysql_plugin PRIVATE
  "${SAMP_SDK_DIR}"
  "${CMAKE_CURRENT_SOURCE_DIR}/deps/mariadb-connector-c/include"
  "${CMAKE_CURRENT_BINARY_DIR}/deps/mariadb-connector-c/include"
)

target_compile_definitions(mysql_plugin PRIVATE
  HAVE_STDINT_H
  PAWN_CELL_SIZE=32
  $<$<PLATFORM_ID:Windows>:WIN32;_CRT_SECURE_NO_WARNINGS;_CRT_NONSTDC_NO_DEPRECATE>
  $<$<PLATFORM_ID:Linux>:LINUX;_GNU_SOURCE>
)

if(MSVC)
  target_compile_options(mysql_plugin PRIVATE /W3)
  target_sources(mysql_plugin PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/mysql.def")
  set_target_properties(mysql_plugin PROPERTIES
    LINK_FLAGS "/DEF:\"${CMAKE_CURRENT_SOURCE_DIR}/mysql.def\""
  )
else()
  target_compile_options(mysql_plugin PRIVATE -Wall -Wextra -fvisibility=hidden -fshort-wchar)
  target_link_options(mysql_plugin PRIVATE -Wl,--no-undefined)
endif()

# Static MariaDB client + system deps pulled in via INTERFACE.
target_link_libraries(mysql_plugin PRIVATE mariadbclient)

if(UNIX)
  find_package(Threads REQUIRED)
  target_link_libraries(mysql_plugin PRIVATE Threads::Threads ${CMAKE_DL_LIBS})
endif()

install(TARGETS mysql_plugin
  RUNTIME DESTINATION plugins
  LIBRARY DESTINATION plugins
)
install(FILES mysql.inc DESTINATION include)
