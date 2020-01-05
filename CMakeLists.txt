cmake_minimum_required(VERSION 3.0 FATAL_ERROR)

project(redis VERSION 999.999.999)

if (NOT CMAKE_BUILD_TYPE)
    message(STATUS "No build type defined; defaulting to 'Debug'")
    set(CMAKE_BUILD_TYPE "Debug" CACHE STRING
            "The type of build. Possible values are: Debug, Release, RelWithDebInfo and MinSizeRel.")
endif ()

message(STATUS "Host is: ${CMAKE_HOST_SYSTEM}.  Build target is: ${CMAKE_SYSTEM}")
get_filename_component(REDIS_ROOT "${CMAKE_CURRENT_SOURCE_DIR}" ABSOLUTE)
message(STATUS "Project root directory is: ${REDIS_ROOT}")

# Just for debugging when handling a new platform.
if (false)
    message("C++ compiler supports these language features:")
    foreach (i ${CMAKE_CXX_COMPILE_FEATURES})
        message("  ${i}")
    endforeach ()
endif ()

if (${CMAKE_SYSTEM_NAME} MATCHES "Linux")
    #相当于代码中的#define
    add_definitions(-DUSE_JEMALLOC)
endif ()

message(STATUS "Generating release.h...")
execute_process(
        COMMAND sh -c ./mkreleasehdr.sh
        WORKING_DIRECTORY ${REDIS_ROOT}/src/
)
#add_subdirectory(deps)

set(SRC_SERVER
        src/adlist.c
        src/ae.c
        src/anet.c
        src/bio.c
        src/aof.c
        src/bitops.c
        src/blocked.c
        src/childinfo.c
        src/cluster.c
        src/config.c
        src/crc16.c
        src/crc64.c
        src/debug.c
        src/defrag.c
        src/dict.c
        src/server.c
        src/endianconv.c
        src/evict.c
        src/expire.c
        src/geo.c
        src/geohash.c
        src/geohash_helper.c
        src/hyperloglog.c
        src/intset.c
        src/latency.c
        src/lazyfree.c
        src/listpack.c
        src/listpack_malloc.h
        src/localtime.c
        src/lolwut.c
        src/lolwut5.c
        src/lzf_c.c
        src/lzf_d.c
        src/lzfP.h
        src/memtest.c
        src/module.c
        src/multi.c
        src/networking.c
        src/notify.c
        src/object.c
        src/db.c
        src/pqsort.c
        src/pubsub.c
        src/quicklist.c
        src/rand.c
        src/rax.c
        src/rdb.c
        src/redis-check-aof.c
        src/redis-check-rdb.c
        src/redismodule.h
        src/release.c
        src/replication.c
        src/rio.c
        src/scripting.c
        src/sds.c
        src/sdsalloc.h
        src/sentinel.c
        src/setproctitle.c
        src/sha1.c
        src/siphash.c
        src/slowlog.c
        src/solarisfixes.h
        src/sort.c
        src/sparkline.c
        src/syncio.c
        src/t_hash.c
        src/t_list.c
        src/t_set.c
        src/t_stream.c
        src/t_string.c
        src/t_zset.c
        src/util.c
        src/ziplist.c
        src/zipmap.c
        src/zmalloc.c
        )

set(SRC_CLI
        src/anet.c src/sds.c src/adlist.c src/redis-cli.c src/zmalloc.c
        src/release.c src/anet.c src/ae.c src/crc64.c src/dict.c
        src/crc16.c src/siphash.c
        )

set(SRC_REDIS_BENCHMARK
        src/redis-benchmark.c src/ae.c src/sds.c)


if (${CMAKE_SYSTEM_NAME} MATCHES "Linux")
    # better not to work with jemalloc
endif ()

add_executable(redis-server ${SRC_SERVER})
add_executable(redis-cli ${SRC_CLI})
#add_executable(redis-benchmark ${SRC_REDIS_BENCHMARK})

set_property(TARGET redis-server PROPERTY C_STANDARD 99)
set_property(TARGET redis-server PROPERTY CXX_STANDARD 11)
set_property(TARGET redis-server PROPERTY CXX_STANDARD_REQUIRED ON)

set_property(TARGET redis-cli PROPERTY C_STANDARD 99)
set_property(TARGET redis-cli PROPERTY CXX_STANDARD 11)
set_property(TARGET redis-cli PROPERTY CXX_STANDARD_REQUIRED ON)

#set_property(TARGET redis-benchmark PROPERTY C_STANDARD 99)
#set_property(TARGET redis-benchmark PROPERTY CXX_STANDARD 11)
#set_property(TARGET redis-benchmark PROPERTY CXX_STANDARD_REQUIRED ON)


target_include_directories(redis-server
        PRIVATE ${REDIS_ROOT}/deps/hiredis
        PRIVATE ${REDIS_ROOT}/deps/linenoise
        PRIVATE ${REDIS_ROOT}/deps/jemalloc/include
        PRIVATE ${REDIS_ROOT}/deps/lua/src
        )

target_include_directories(redis-cli
        PRIVATE ${REDIS_ROOT}/deps/hiredis
        PRIVATE ${REDIS_ROOT}/deps/linenoise
        PRIVATE ${REDIS_ROOT}/deps/jemalloc/include
        PRIVATE ${REDIS_ROOT}/deps/lua/src
        )


#target_include_directories(redis-benchmark
#        PRIVATE ${REDIS_ROOT}/deps/hiredis
#        PRIVATE ${REDIS_ROOT}/deps/linenoise
#        PRIVATE ${REDIS_ROOT}/deps/jemalloc/include
#        PRIVATE ${REDIS_ROOT}/deps/lua/src
#        )


target_link_libraries(redis-server
        PRIVATE pthread
        PRIVATE m
        PRIVATE lua
        PRIVATE linenoise
        PRIVATE hiredis
        PRIVATE jemalloc
        #由于jemalloc的静态链接包不是通过cmake编译的, 是单独编译的,所以这里需要指定静态包的绝对路径
        PRIVATE ${REDIS_ROOT}/deps/jemalloc/lib/libjemalloc.a
        #Name of library containing dlopen and dlclose.
        ${CMAKE_DL_LIBS}
        )

target_link_libraries(redis-cli
        PRIVATE pthread
        PRIVATE m
        PRIVATE linenoise
        PRIVATE hiredis
        PRIVATE ${REDIS_ROOT}/deps/jemalloc/lib/libjemalloc.a
        ${CMAKE_DL_LIBS}
        )

#target_link_libraries(redis-benchmark
#        PRIVATE pthread
#        PRIVATE m
#        PRIVATE lua
#        PRIVATE linenoise
#        PRIVATE hiredis
#        PRIVATE ${REDIS_ROOT}/deps/jemalloc/lib/libjemalloc.a
#        ${CMAKE_DL_LIBS}
#        )

link_directories(deps/hiredis/ deps/linenoise/ diredeps/lua/src)