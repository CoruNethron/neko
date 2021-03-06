include(FindPkgConfig)

######################
# zlib.ndll

add_library(zlib.ndll MODULE zlib.c)


if (STATIC_ZLIB)
	set(ZLIB_CMAKE_ARGS
		-DCMAKE_INSTALL_PREFIX=${CMAKE_BINARY_DIR}/libs/src/install-prefix
		-Wno-dev
	)
	if (UNIX)
		list(APPEND ZLIB_CMAKE_ARGS
			-DCMAKE_OSX_ARCHITECTURES=${CMAKE_OSX_ARCHITECTURES}
			${ARG_PIC}
		)
	endif()
	ExternalProject_Add(Zlib
		${EP_CONFIGS}
		URL http://zlib.net/zlib-1.2.11.tar.gz
		URL_MD5 1c9f62f0778697a09d36121ead88e08e
		CMAKE_ARGS ${ZLIB_CMAKE_ARGS}
	)
	set_target_properties(Zlib PROPERTIES ${EP_PROPS})
	set(ZLIB_INCLUDE_DIRS ${CMAKE_BINARY_DIR}/libs/src/install-prefix/include)
	if (WIN32)
		set(ZLIB_LIBRARIES
			optimized ${CMAKE_BINARY_DIR}/libs/src/install-prefix/lib/zlibstatic.lib
			debug ${CMAKE_BINARY_DIR}/libs/src/install-prefix/lib/zlibstaticd.lib
		)
	else()
		set(ZLIB_LIBRARIES ${CMAKE_BINARY_DIR}/libs/src/install-prefix/lib/libz.a)
	endif()
	add_dependencies(zlib.ndll Zlib)
	target_include_directories(zlib.ndll PRIVATE ${ZLIB_INCLUDE_DIRS})
	# Download project for fat source archive
	add_dependencies(download_static_deps Zlib-download)
else()
	pkg_check_modules(ZLIB REQUIRED zlib)
	target_include_directories(zlib.ndll PRIVATE ${ZLIB_INCLUDEDIR} ${ZLIB_INCLUDE_DIRS})
endif()

target_link_libraries(zlib.ndll libneko ${ZLIB_LIBRARIES})

set_target_properties(zlib.ndll
	PROPERTIES
	PREFIX ""
	OUTPUT_NAME zlib
	SUFFIX .ndll
)

install (
	TARGETS zlib.ndll
	DESTINATION ${DEST_NDLL}
)

install(SCRIPT ${NEKO_FLATTEN_SCRIPT})
