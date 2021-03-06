FILE(GLOB veyoncore_INCLUDES
						${CMAKE_CURRENT_SOURCE_DIR}/include/*.h
						${CMAKE_CURRENT_SOURCE_DIR}/include/Configuration/*.h
)
FILE(GLOB veyoncore_SOURCES
						${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp
						${CMAKE_CURRENT_SOURCE_DIR}/src/*.ui
						${CMAKE_CURRENT_SOURCE_DIR}/src/Configuration/*.cpp
)

SET(core_RESOURCES ${CMAKE_CURRENT_SOURCE_DIR}/resources/core.qrc ${CMAKE_CURRENT_BINARY_DIR}/builddata.qrc)

# find Qt's translation directory
IF(NOT VEYON_BUILD_WIN32)
	GET_TARGET_PROPERTY(QT_QMAKE_EXECUTABLE Qt5::qmake IMPORTED_LOCATION)
	EXECUTE_PROCESS(COMMAND "${QT_QMAKE_EXECUTABLE}" -query QT_INSTALL_TRANSLATIONS
						OUTPUT_STRIP_TRAILING_WHITESPACE
						OUTPUT_VARIABLE QT_TRANSLATIONS_DIR)
	ADD_DEFINITIONS(-D'QT_TRANSLATIONS_DIR="${QT_TRANSLATIONS_DIR}"')
ENDIF()


SET(libvncclient_SOURCES
	${libvncserver_DIR}/libvncclient/cursor.c
	${libvncserver_DIR}/libvncclient/listen.c
	${libvncserver_DIR}/libvncclient/rfbproto.c
	${libvncserver_DIR}/libvncclient/sockets.c
	${libvncserver_DIR}/libvncclient/tls_none.c
	${libvncserver_DIR}/libvncclient/vncviewer.c
	${libvncserver_DIR}/common/d3des.c
	${libvncserver_DIR}/common/turbojpeg.c)


ADD_DEFINITIONS(-DBUILD_VEYON_CORE_LIBRARY)

INCLUDE_DIRECTORIES(${CMAKE_BINARY_DIR} ${CMAKE_CURRENT_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/include ${libvncserver_DIR}/common/)
ADD_LIBRARY(veyon-core SHARED ${veyoncore_SOURCES} ${veyoncore_INCLUDES} ${core_RESOURCES} ${veyoncore_qt_qm_out} ${libvncclient_SOURCES})

SET_SOURCE_FILES_PROPERTIES(${libvncclient_SOURCES} PROPERTIES COMPILE_FLAGS "-Wno-unused-variable -Wno-unused-label -Wno-maybe-uninitialized -Wno-misleading-indentation -Wno-pointer-sign -Wno-int-in-bool-context")

TARGET_LINK_LIBRARIES(veyon-core
	Qt5::Concurrent
	Qt5::Gui
	Qt5::Network
	Qt5::Widgets
	${VEYON_DEBUG_LIBRARIES}
	${ZLIB_LIBRARIES}
	${JPEG_LIBRARIES}
	${PNG_LIBRARIES}
	${LZO_LIBRARIES}
	${QCA_LIBRARY}
	${OPENSSL_LIBRARIES}
	)

IF(VEYON_BUILD_WIN32)
	# add Windows Socket library required by libvncclient
	TARGET_LINK_LIBRARIES(veyon-core -lws2_32)
	SET_TARGET_PROPERTIES(veyon-core PROPERTIES PREFIX "")
	INSTALL(TARGETS veyon-core RUNTIME DESTINATION ${VEYON_LIB_DIR})
ELSE()
	INSTALL(TARGETS veyon-core LIBRARY DESTINATION ${VEYON_LIB_DIR})
ENDIF(VEYON_BUILD_WIN32)

COTIRE_VEYON(veyon-core)
