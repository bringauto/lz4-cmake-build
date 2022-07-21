#
# CMake LZ4 Build.
#
# Cmd line options:
#   CMAKE_BUILD_TYPE in { Debug, Release }
#   LZ4_VERSION - valid lz4 version (Example: 1.76.0)
#
#

PROJECT(CMAKE_LZ4 CXX)
CMAKE_MINIMUM_REQUIRED(VERSION 3.16)

IF(NOT LZ4_VERSION)
	MESSAGE(FATAL_ERROR "LZ4 version variable LZ4_VERSION not specified!!")
ENDIF()

IF(NOT CMAKE_BUILD_TYPE)
	MESSAGE(FATAL_ERROR "Please set CMAKE_BUILD_TYPE!!")
ENDIF()

IF(NOT "${CMAKE_BUILD_TYPE}" STREQUAL "Debug" AND
		NOT "${CMAKE_BUILD_TYPE}" STREQUAL "Release")
	MESSAGE(FATAL_ERROR "Unsupported build type: ${CMAKE_BUILD_TYPE}")
ENDIF()

STRING(REGEX MATCH "^[0-9]\.[0-9]+\.[0-9]+$" version_ok "${LZ4_VERSION}")
IF(NOT version_ok)
	MESSAGE(FATAL_ERROR "LZ4 version ${LZ4_VERSION} is not valid!")
ENDIF()

SET(LZ4_DOWNLOAD_URL "https://github.com/lz4/lz4/archive/refs/tags/v${LZ4_VERSION}.tar.gz")
SET(LZ4_INSTALL_DIR "${CMAKE_CURRENT_BINARY_DIR}/lz4_install")

SET(archive_path           ${CMAKE_CURRENT_BINARY_DIR}/lz4_${LZ4_VERSION}.tar.gz)
SET(extracted_archive_path ${CMAKE_CURRENT_BINARY_DIR}/lz4_${LZ4_VERSION}_${CMAKE_BUILD_TYPE})

IF(NOT EXISTS "${archive_path}")
	FILE(DOWNLOAD "${LZ4_DOWNLOAD_URL}" "${archive_path}" SHOW_PROGRESS)
ENDIF()
FILE(MAKE_DIRECTORY  "${extracted_archive_path}")
EXECUTE_PROCESS(
		COMMAND ${CMAKE_COMMAND} -E tar xjf ${archive_path}
		WORKING_DIRECTORY ${extracted_archive_path}
)

SET(lz4_sources "${extracted_archive_path}/lz4-${LZ4_VERSION}")

MESSAGE(STATUS ${lz4_sources})

SET(exe_permissions WORLD_EXECUTE WORLD_READ GROUP_EXECUTE GROUP_READ OWNER_EXECUTE OWNER_READ OWNER_WRITE)

ADD_CUSTOM_COMMAND(
		OUTPUT "${LZ4_INSTALL_DIR}"
		MAIN_DEPENDENCY "${lz4_sources}"
		COMMAND make DESTDIR="${LZ4_INSTALL_DIR}" -j 16 install
		WORKING_DIRECTORY "${lz4_sources}"
		COMMENT "Build lz4 from sources"
)

ADD_CUSTOM_TARGET(lz4_build ALL
	DEPENDS "${LZ4_INSTALL_DIR}"
)

INSTALL(DIRECTORY ${LZ4_INSTALL_DIR}/usr/local/include/
	DESTINATION "include"
)

INSTALL(DIRECTORY ${LZ4_INSTALL_DIR}/usr/local/bin/
	DESTINATION "bin"
)
INSTALL(PROGRAMS ${LZ4_INSTALL_DIR}/usr/local/bin/lz4
	DESTINATION "bin"
	PERMISSIONS ${exe_permissions}
)

INSTALL(DIRECTORY ${LZ4_INSTALL_DIR}/usr/local/lib/
	DESTINATION "lib"
)
INSTALL(FILES ${LZ4_INSTALL_DIR}/usr/local/lib/liblz4.so.${LZ4_VERSION}
	DESTINATION "lib"
	PERMISSIONS ${exe_permissions}
)


INSTALL(DIRECTORY ${LZ4_INSTALL_DIR}/usr/local/share/
	DESTINATION "share"
)
