
cmake_minimum_required(VERSION 3.20)
project(glew C)

set(OpenGL_GL_PREFERENCE GLVND)
find_package(OpenGL REQUIRED)

if (NOT WIN32 AND NOT APPLE)
	find_package(X11 REQUIRED)
endif()

if(CMAKE_SIZEOF_VOID_P EQUAL 8)
	set(ARCH_POSTFIX 64)
else()
	set(ARCH_POSTFIX "")
endif()

set(CMAKE_DEBUG_POSTFIX d)

message("Configuration: ARCH_POSTFIX:${ARCH_POSTFIX} CMAKE_DEBUG_POSTFIX:${CMAKE_DEBUG_POSTFIX}")

# glew
set (GLEW_PUBLIC_HEADERS_FILES include/GL/wglew.h include/GL/glew.h include/GL/glxew.h)
set (GLEW_SRC_FILES src/glew.c)

if (WIN32)
	list (APPEND GLEW_SRC_FILES build/glew.rc)
endif ()

foreach(type SHARED STATIC)
	if (${type} MATCHES "SHARED")
		set(suffix ${ARCH_POSTFIX}s)
	else()
		set(suffix ${ARCH_POSTFIX})
	endif()

	set(target glew${suffix})

	add_library(${target} ${type} ${GLEW_PUBLIC_HEADERS_FILES} ${GLEW_SRC_FILES})

	target_compile_definitions(${target} PRIVATE "VC_EXTRALEAN")

	if (${type} MATCHES "SHARED")
		target_compile_definitions(${target} PRIVATE "GLEW_BUILD")
	else()
		target_compile_definitions(${target} PRIVATE "GLEW_STATIC")
	endif()

	target_include_directories(${target} PRIVATE include)
	target_link_libraries(${target} PRIVATE OpenGL::GL)
endforeach()

# glewinfo
set (GLEWINFO_SRC_FILES src/glewinfo.c)

if (WIN32)
	list (APPEND GLEWINFO_SRC_FILES build/glewinfo.rc)
endif ()

add_executable(glewinfo ${GLEWINFO_SRC_FILES})
target_include_directories(glewinfo PRIVATE include)
target_link_libraries(glewinfo PRIVATE glew${ARCH_POSTFIX}s OpenGL::GL)

if (NOT WIN32 AND NOT APPLE)
	target_link_libraries(glewinfo PRIVATE ${X11_LIBRARIES})
endif()

# visualinfo
set (VISUALINFO_SRC_FILES src/visualinfo.c)

if (WIN32)
	list (APPEND VISUALINFO_SRC_FILES build/visualinfo.rc)
endif ()

add_executable(visualinfo ${VISUALINFO_SRC_FILES})
target_include_directories(visualinfo PRIVATE include)
target_link_libraries(visualinfo PRIVATE glew${ARCH_POSTFIX}s OpenGL::GL OpenGL::GLU)

if (NOT WIN32 AND NOT APPLE)
	target_link_libraries(visualinfo PRIVATE ${X11_LIBRARIES})
endif()
