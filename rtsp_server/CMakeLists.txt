get_filename_component(ProjectId ${CMAKE_CURRENT_SOURCE_DIR} NAME)
string(REPLACE " " "_" ProjectId ${ProjectId})
project(${ProjectId})

message("ProjectId: '${ProjectId}'")

add_library(${ProjectId} SHARED
    src/rtsp_server.c
    ${libhl_SOURCE_DIR}/src/rbuf.c
    ${libhl_SOURCE_DIR}/src/rqueue.c
)

target_include_directories(${ProjectId}
    PUBLIC
    ${PROJECT_SOURCE_DIR}/src
    ${PROJECT_SOURCE_DIR}/src/plugins
    PRIVATE
    # "${json-c_BINARY_DIR}"
    # "${json-c_SOURCE_DIR}"
    "${pm_SOURCE_DIR}/src"
    "${libevent_BINARY_DIR}/include"
    "${log.c_SOURCE_DIR}/src"
    "${libevent_SOURCE_DIR}/include"
    "${smolrtsp_SOURCE_DIR}/include"
    "${smolrtsp-libevent_SOURCE_DIR}/include"
    ${LIBEVENT_INCLUDE_DIR}
    "${libhl_SOURCE_DIR}/src"
    "${interface99_SOURCE_DIR}"
    "${slice99_SOURCE_DIR}"
    "${datatype99_SOURCE_DIR}"
    "${metalang99_SOURCE_DIR}/include"
)
# target_link_libraries(rtsp_server smolrtsp smolrtsp-libevent ${LIBEVENT_LIBRARIES})
target_link_libraries(${ProjectId} smolrtsp smolrtsp-libevent event input_read_fifo)
set_target_properties(${ProjectId} PROPERTIES C_STANDARD 99 C_STANDARD_REQUIRED ON)
if(CMAKE_C_COMPILER_ID STREQUAL "Clang")
  target_compile_options(${ProjectId} PRIVATE -Wall -Wextra -fsanitize=address)
elseif(CMAKE_C_COMPILER_ID STREQUAL "GNU")
  # target_compile_options(rtsp_server PRIVATE -Wall -Wextra -fsanitize=address -Wno-misleading-indentation)
  target_compile_options(${ProjectId} PRIVATE  -O1 -g -fno-omit-frame-pointer -Wno-builtin-macro-redefined -Wall -Wextra -Wno-misleading-indentation)
endif()

target_link_options(${ProjectId} PRIVATE -fsanitize=address)
