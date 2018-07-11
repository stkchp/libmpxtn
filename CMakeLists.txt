cmake_minimum_required(VERSION 3.0)

project(LIBMPXTN VERSION 0.0.1 LANGUAGES C)

set(MPXTN_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)
set(TEST_DIR  ${CMAKE_CURRENT_SOURCE_DIR}/test)

file(GLOB MPXTN_SRC "${MPXTN_DIR}/*.h" "${MPXTN_DIR}/*.c")
set(TEST_SRC ${TEST_DIR}/dummydump.c)

set(VERSION_SRC ${MPXTN_DIR}/libmpxtn.map)

# option
option(USE_OGGVORBIS "Woice support ogg vorbis" ON)

# Debug True!!
set(CMAKE_BUILD_TYPE Debug)

set(CMAKE_VERBOSE_MAKEFILE ON)
add_compile_options(-std=c11)
if (CMAKE_BUILD_TYPE STREQUAL "Debug")
	if (CMAKE_C_COMPILER_ID MATCHES "Clang")
		add_compile_options(-Weverything)
		add_compile_options(-Wno-padded)
		add_compile_options(-Wno-missing-field-initializers)
	endif ()
	add_compile_options(-Wall)
	add_compile_options(-g3 -O0)
	add_compile_options(-D_DEBUG)
else ()
	add_compile_options(-Wall)
	add_compile_options(-march=native)
	add_compile_options(-O2 -pipe -g0)
	add_compile_options(-fvisibility=hidden)

	if(CMAKE_SYSTEM_NAME MATCHES "Darwin")
	else()
	set(CUSTOM_LDFLAGS "${CUSTOM_LDFLAGS} -Wl,-s")
	set(CUSTOM_LDFLAGS "${CUSTOM_LDFLAGS} -Wl,--as-needed")
	set(CUSTOM_LDFLAGS "${CUSTOM_LDFLAGS} -Wl,--version-script=${VERSION_SRC}")
	endif()

	set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} ${CUSTOM_LDFLAGS}")
endif ()

if(USE_OGGVORBIS)
	add_compile_options(-DMPXTN_OGGVORBIS)
endif()

# shared library
add_library(mpxtn SHARED ${MPXTN_SRC})

target_link_libraries(mpxtn m)

if(USE_OGGVORBIS)
	target_link_libraries(mpxtn vorbisfile)
endif()

set_property(TARGET mpxtn PROPERTY VERSION ${LIBMPXTN_VERSION})
set_property(TARGET mpxtn PROPERTY SOVERSION ${LIBMPXTN_VERSION_MAJOR})

# static library
# add_library(mpxtn_static STATIC ${MPXTN_SRC})

# test executable
add_executable(dummydump ${TEST_SRC})

target_link_libraries(dummydump mpxtn)
target_include_directories(dummydump PUBLIC ${MPXTN_DIR})

# install headers
install(FILES ${MPXTN_DIR}/mpxtn.h DESTINATION include/mpxtn)


message (STATUS "USE_OGGVORBIS=${USE_OGGVORBIS}")
