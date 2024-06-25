cmake_minimum_required(VERSION 3.0.2 FATAL_ERROR)

get_filename_component(ProjectId ${CMAKE_CURRENT_SOURCE_DIR} NAME)
string(REPLACE " " "_" ProjectId ${ProjectId})
project(${ProjectId})
project (${ProjectId} LANGUAGES C)

set(CMAKE_C_STANDARD 99)
set(CMAKE_C_FLAGS "-std=gnu99")

cmake_policy(SET CMP0069 NEW)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

# find_package(LibEvent REQUIRED)

if(CMAKE_BUILD_TYPE STREQUAL "Release")
    set(CMAKE_C_FLAGS
        "${CMAKE_C_FLAGS} -s -Os -ffunction-sections -Wl,--gc-sections -DNDEBUG -Wextra -Wno-builtin-macro-redefined")
    add_definitions(-DLOG_LEVEL=2)
else()
    if(BUILD_SHARED_LIBS)
        set(CMAKE_C_FLAGS
            # "${CMAKE_C_FLAGS} -O1 -g -fsanitize=address -fno-omit-frame-pointer -Wno-builtin-macro-redefined")
            "${CMAKE_C_FLAGS} -O1 -g -fno-omit-frame-pointer -Wno-builtin-macro-redefined")
    endif()
    add_definitions(-DLOG_LEVEL=0 -DDEBUG)
endif()

add_definitions(
    -DBUILD_TYPE="${CMAKE_BUILD_TYPE}"
    -DPROJECT_NAME="${PROJECT_PRESENT_NAME}"
    -DPROJECT_FOLDER="${PROJECT}"
    -DTHREAD_SAFE
    -DLOG_USE_COLOR
    -DYAML
    -DVIDEO_FPS=25
)

if(NOT SKIP_VERSION)
    set(VERSION_SRC ${CMAKE_CURRENT_BINARY_DIR}/version.c)
    # Add a custom command that produces version.c, plus a dummy output that's not
    # actually produced, in order to force version.cmake to always be re-run
    # before the build
    add_custom_command(
        OUTPUT ${VERSION_SRC} ${CMAKE_CURRENT_BINARY_DIR}/_version.c
        COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_SOURCE_DIR}/cmake/version.cmake)
else()
    add_definitions(-DSKIP_VERSION)
endif()

include(FetchContent)

FetchContent_Declare(
    log.c
    URL https://github.com/SSharshunov/log.c/archive/refs/tags/latest.tar.gz
)

FetchContent_Declare(
    libhl
    URL https://github.com/xant/libhl/archive/refs/tags/libhl-3.1.tar.gz
)

FetchContent_Declare(
    smolrtsp
    # URL https://github.com/OpenIPC/smolrtsp/archive/refs/tags/v0.1.3.tar.gz
    GIT_REPOSITORY https://github.com/OpenIPC/smolrtsp.git
    GIT_TAG 9f708a4
)

FetchContent_Declare(
    smolrtsp-libevent
    GIT_REPOSITORY https://github.com/OpenIPC/smolrtsp-libevent
    GIT_TAG 7451e74
)

FetchContent_Declare(
    pm
    GIT_REPOSITORY https://github.com/SSharshunov/pm.git
    GIT_TAG main
    # URL https://github.com/SSharshunov/pm/archive/refs/tags/latest.tar.gz
)

FetchContent_MakeAvailable(log.c libhl pm smolrtsp smolrtsp-libevent)

MACRO(SUBDIRLIST result curdir)
    FILE(GLOB children RELATIVE ${curdir} ${curdir}/*)
    SET(dirlist "")
    FOREACH(child ${children})
        IF(IS_DIRECTORY ${curdir}/${child})
            IF(EXISTS "${curdir}/${child}/CMakeLists.txt")
                LIST(APPEND dirlist ${child})
            ENDIF()
        ENDIF()
    ENDFOREACH()
    SET(${result} ${dirlist})
ENDMACRO()

SET (MY_CURRENT_DIR ${CMAKE_CURRENT_SOURCE_DIR})
SUBDIRLIST(SUBDIRS ${MY_CURRENT_DIR})
# add_subdirectory(lib1)
FOREACH(subdir ${SUBDIRS})
  ADD_SUBDIRECTORY(${subdir})
ENDFOREACH()

# get_filename_component(ProjectId ${CMAKE_CURRENT_SOURCE_DIR} NAME)
# string(REPLACE " " "_" ProjectId ${ProjectId})
# project(${ProjectId})

# function(add_plugin plugin_name plugin_source plugin_incs)
#     add_library(${plugin_name} SHARED ${plugin_source})
#     target_include_directories(ping
#         PUBLIC
#         ${PROJECT_SOURCE_DIR}/src
#         ${PROJECT_SOURCE_DIR}/src/plugins
#         PRIVATE
#         # "${json-c_BINARY_DIR}"
#         # "${json-c_SOURCE_DIR}"
#         "${libevent_BINARY_DIR}/include"
#         "${libevent_SOURCE_DIR}/include"
#         "${libhl_SOURCE_DIR}/src"
#         # "${argparse_SOURCE_DIR}"
#         "${plugin_incs}"
#     )
#     set_target_properties(${plugin_name} PROPERTIES C_STANDARD 99 C_STANDARD_REQUIRED ON)
# endfunction()

# SET (PL_SRC src/plugins/test_read_fifo.c ${libhl_SOURCE_DIR}/src/rqueue.c )
# SET (PL_INC "${smolrtsp_SOURCE_DIR}/include"
#     "${interface99_SOURCE_DIR}"
#     "${slice99_SOURCE_DIR}"
#     "${datatype99_SOURCE_DIR}"
#     "${metalang99_SOURCE_DIR}/include" )
# add_plugin(input_read_fifo ${PL_SRC} ${PL_INC})

# add_library(input_read_fifo SHARED
#     src/plugins/test_read_fifo.c
#     ${libhl_SOURCE_DIR}/src/rqueue.c
# )

# target_include_directories(input_read_fifo
#     PUBLIC
#     ${PROJECT_SOURCE_DIR}/src
#     ${PROJECT_SOURCE_DIR}/src/plugins
#     PRIVATE
#     "${json-c_BINARY_DIR}"
#     "${json-c_SOURCE_DIR}"
#     "${libevent_BINARY_DIR}/include"
#     "${libevent_SOURCE_DIR}/include"
#     "${libhl_SOURCE_DIR}/src"
#     "${argparse_SOURCE_DIR}"
#     "${smolrtsp_SOURCE_DIR}/include"
#     "${interface99_SOURCE_DIR}"
#     "${slice99_SOURCE_DIR}"
#     "${datatype99_SOURCE_DIR}"
#     "${metalang99_SOURCE_DIR}/include"
# )
# set_target_properties(input_read_fifo PROPERTIES C_STANDARD 99 C_STANDARD_REQUIRED ON)
# # target_link_libraries(input_read_fifo smolrtsp smolrtsp-libevent event)
# target_link_libraries(input_read_fifo smolrtsp)

# add_library(http_server SHARED src/plugins/http_server.c)
# target_include_directories(http_server
#     PUBLIC
#     ${PROJECT_SOURCE_DIR}/src
#     ${PROJECT_SOURCE_DIR}/src/plugins
#     PRIVATE
#     "${json-c_BINARY_DIR}"
#     "${json-c_SOURCE_DIR}"
#     "${libevent_BINARY_DIR}/include"
#     "${libevent_SOURCE_DIR}/include"
#     "${libhl_SOURCE_DIR}/src"
#     "${argparse_SOURCE_DIR}"
# )
# set_target_properties(http_server PROPERTIES C_STANDARD 99 C_STANDARD_REQUIRED ON)

# add_library(ping SHARED src/plugins/ping.c)
# target_include_directories(ping
#     PUBLIC
#     ${PROJECT_SOURCE_DIR}/src
#     ${PROJECT_SOURCE_DIR}/src/plugins
#     PRIVATE
#     "${json-c_BINARY_DIR}"
#     "${json-c_SOURCE_DIR}"
#     "${libevent_BINARY_DIR}/include"
#     "${libevent_SOURCE_DIR}/include"
#     "${libhl_SOURCE_DIR}/src"
#     "${argparse_SOURCE_DIR}"
# )
# set_target_properties(ping PROPERTIES C_STANDARD 99 C_STANDARD_REQUIRED ON)

# execute_process(COMMAND pwd)
# execute_process(COMMAND pwd)
# execute_process(COMMAND xxd -i src/plugins/media/audio.g711a src/plugins/media/audio.g711a.h)
# execute_process(COMMAND xxd -i src/plugins/media/video.h264 src/plugins/media/video.h264.h)

# add_plugin(ping src/plugins/ping.c)
SET(INCS
    "${smolrtsp_SOURCE_DIR}/include"
    "${smolrtsp-libevent_SOURCE_DIR}/include"
    ${LIBEVENT_INCLUDE_DIR}
    "${interface99_SOURCE_DIR}"
    "${slice99_SOURCE_DIR}"
    "${datatype99_SOURCE_DIR}"
    "${metalang99_SOURCE_DIR}/include"
)
# add_plugin(rtsp_server
#     src/plugins/rtsp_server.c
#     ${libhl_SOURCE_DIR}/src/rbuf.c
#     ${libhl_SOURCE_DIR}/src/rqueue.c
#     INCS
# )

# add_library(rtsp_server SHARED
#     src/plugins/rtsp_server.c
#     ${libhl_SOURCE_DIR}/src/rbuf.c
#     ${libhl_SOURCE_DIR}/src/rqueue.c
# )
# target_include_directories(rtsp_server
#     PUBLIC
#     ${PROJECT_SOURCE_DIR}/src
#     ${PROJECT_SOURCE_DIR}/src/plugins
#     PRIVATE
#     # "${json-c_BINARY_DIR}"
#     # "${json-c_SOURCE_DIR}"
#     "${libevent_BINARY_DIR}/include"
#     "${libevent_SOURCE_DIR}/include"
#     "${smolrtsp_SOURCE_DIR}/include"
#     "${smolrtsp-libevent_SOURCE_DIR}/include"
#     ${LIBEVENT_INCLUDE_DIR}
#     "${libhl_SOURCE_DIR}/src"
#     "${argparse_SOURCE_DIR}"
#     "${interface99_SOURCE_DIR}"
#     "${slice99_SOURCE_DIR}"
#     "${datatype99_SOURCE_DIR}"
#     "${metalang99_SOURCE_DIR}/include"
# )
# # target_link_libraries(rtsp_server smolrtsp smolrtsp-libevent ${LIBEVENT_LIBRARIES})
# target_link_libraries(rtsp_server smolrtsp smolrtsp-libevent event input_read_fifo)
# set_target_properties(rtsp_server PROPERTIES C_STANDARD 99 C_STANDARD_REQUIRED ON)
# if(CMAKE_C_COMPILER_ID STREQUAL "Clang")
#   target_compile_options(rtsp_server PRIVATE -Wall -Wextra -fsanitize=address)
# elseif(CMAKE_C_COMPILER_ID STREQUAL "GNU")
#   # target_compile_options(rtsp_server PRIVATE -Wall -Wextra -fsanitize=address -Wno-misleading-indentation)
#   target_compile_options(rtsp_server PRIVATE  -O1 -g -fno-omit-frame-pointer -Wno-builtin-macro-redefined -Wall -Wextra -Wno-misleading-indentation)
# endif()

# target_link_options(rtsp_server PRIVATE -fsanitize=address)

# get_cmake_property(_variableNames VARIABLES)
# list (SORT _variableNames)
# foreach (_variableName ${_variableNames})
#     message(STATUS "${_variableName}=${${_variableName}}")
# endforeach()

# # if(NOT (CMAKE_SYSTEM_PROCESSOR STRLESS "mips"))
# IF(NOT ${CMAKE_SYSTEM_PROCESSOR} MATCHES "mips")
#     set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-builtin-macro-redefined -D'__FILE__=\"$(subst $(realpath ${CMAKE_SOURCE_DIR})/,,$(abspath $<))\"'")
#     set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-builtin-macro-redefined -D'__FILE__=\"$(subst $(realpath ${CMAKE_SOURCE_DIR})/,,$(abspath $<))\"'")

# else()
# endif()
