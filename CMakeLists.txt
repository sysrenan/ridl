cmake_minimum_required(VERSION 3.1)

include(CheckFunctionExists)

project(mdstool C CXX ASM)

find_package(PkgConfig)
find_package(Threads)
find_package(X11)

list(APPEND SOURCES
	source/bitmap.c
	source/human.c
	source/system.c
	source/topology.c)

list(APPEND CLI_SOURCES
	source/cli/l1tf.c
	source/cli/main.c
	source/cli/meltdown.c
	source/cli/ridl.c
	source/cli/spectre.c
	source/cli/ssb.c
	source/cli/system.c)

list(APPEND GUI_SOURCES
	source/ui/l1tf.c
	source/ui/main.c
	source/ui/meltdown.c
	source/ui/ridl.c
	source/ui/spectre.c
	source/ui/ssb.c
	source/ui/system.c)

if (${PKG_CONFIG_FOUND})
	pkg_check_modules(FREETYPE freetype2)
endif()

if (${X11_FOUND})
	list(APPEND GUI_SOURCES
		source/x11/canvas.c
		source/x11/display.c
		source/x11/events.c
		source/x11/font.c
		source/x11/image.c
		source/x11/window.c)
endif()

add_definitions(-DNK_INCLUDE_DEFAULT_ALLOCATOR)
add_definitions(-DNK_INCLUDE_COMMAND_USERDATA)

if ("${CMAKE_C_COMPILER_ID}" STREQUAL "GNU")
	list(APPEND SOURCES
		source/x86/gcc/cpuid.c)
elseif ("${CMAKE_C_COMPILER_ID}" STREQUAL "MSVC")
	add_definitions(-D_CRT_SECURE_NO_WARNINGS)
	list(APPEND SOURCES
		source/x86/msvc/cpuid.c)
endif()

if ("${CMAKE_SYSTEM_NAME}" STREQUAL "Windows")
	list(APPEND SOURCES
		source/msw/l1tf.c
		source/msw/meltdown.c
		source/msw/memory.c
		source/msw/microcode.c
		source/msw/os.c
		source/msw/spectre.c
		source/msw/topology.c)
	list(APPEND GUI_SOURCES
		source/msw/canvas.c
		source/msw/display.c
		source/msw/events.c
		source/msw/font.c
		source/msw/image.c
		source/msw/window.c)
elseif ("${CMAKE_SYSTEM_NAME}" STREQUAL "Linux")
	list(APPEND SOURCES
		source/linux/l1tf.c
		source/linux/meltdown.c
		source/linux/memory.c
		source/linux/microcode.c
		source/linux/spectre.c
		source/linux/ssb.c
		source/linux/topology.c
		source/linux/vuln.c)
	list(APPEND SOURCES
		source/unix/os.c)
endif()

list(APPEND SOURCES
	source/x86/cpuid.c
	source/x86/ridl.c)

check_function_exists(asprintf HAVE_ASPRINTF)
check_function_exists(clock_nanosleep HAVE_CLOCK_NANOSLEEP)
check_function_exists(nanosleep HAVE_NANOSLEEP)
check_function_exists(usleep HAVE_USLEEP)

if(HAVE_CLOCK_NANOSLEEP)
	add_definitions(-DHAVE_CLOCK_NANOSLEEP)
endif()

if (NOT HAVE_ASPRINTF)
	list(APPEND SOURCES
		source/compat/asprintf.c)
endif()

if (${CMAKE_SYSTEM_NAME} STREQUAL Windows)
	add_definitions(-D_WIN32_WINNT=0x601)
	add_definitions(-DUNICODE)
else ()
	add_definitions(-D_GNU_SOURCE)
endif()

add_executable(mdstool-cli ${SOURCES} ${CLI_SOURCES})

if (${CMAKE_SYSTEM_NAME} STREQUAL Windows)
	set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /SUBSYSTEM:WINDOWS /ENTRY:mainCRTStartup")
endif()

add_definitions(-DHAVE_GUI)

add_executable(mdstool ${SOURCES} ${GUI_SOURCES})

include_directories(nuklear)
include_directories(stb)
include_directories(include)
include_directories(${X11_INCLUDE_DIR} ${X11_Xft_INCLUDE_PATH} ${X11_Xrender_INCLUDE_PATH} ${FREETYPE_INCLUDE_DIRS})
target_link_libraries(mdstool ${CMAKE_DL_LIBS} ${CMAKE_THREAD_LIBS_INIT} ${X11_LIBRARIES} ${X11_Xft_LIB} ${X11_Xrender_LIB} ${FREETYPE_LIBRARIES})

if (${CMAKE_SYSTEM_NAME} STREQUAL Windows)
	target_link_libraries(mdstool gdiplus ntdll shlwapi)
else()
	target_link_libraries(mdstool m)
endif()
