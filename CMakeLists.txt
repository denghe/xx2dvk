cmake_minimum_required(VERSION 3.20)

set_property(GLOBAL PROPERTY USE_FOLDERS ON)

set(name xx2dvk)
project(${name})

set(CMAKE_CXX_STANDARD 20)

include_directories(
	xxlib
	src
	Vulkan-Hpp
	Vulkan-Hpp/RAII_Samples/utils
	Vulkan-Hpp/samples/utils
)

set(SRCS)
file(GLOB_RECURSE SRCS
	xxlib/*.h
	Vulkan-Hpp/vulkan/*.hpp
	Vulkan-Hpp/RAII_Samples/utils/*.hpp
	Vulkan-Hpp/samples/utils/*.hpp
	Vulkan-Hpp/samples/utils/*.cpp
	src/*.cpp
	src/*.h
	src/*.hpp
	src/*.inc
)
add_executable(${name} ${SRCS})

if(MSVC)
	add_definitions("-D_SECURE_SCL=0")
	add_definitions("-D_HAS_ITERATOR_DEBUGGING=0")

	# todo: precompile header

	set_target_properties(${name} PROPERTIES VS_DEBUGGER_WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}")

	SET_TARGET_PROPERTIES(${name} PROPERTIES COMPILE_FLAGS "/GL")
	SET_TARGET_PROPERTIES(${name} PROPERTIES LINK_FLAGS "/LTCG")
	target_compile_options(${name} 
		PRIVATE $<$<PLATFORM_ID:Windows>:/bigobj>
		PRIVATE $<$<PLATFORM_ID:Windows>:/MP${Biogears_BUILD_PARALLEL_LEVEL}>)
else()
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -flto")
endif()


find_package(Vulkan REQUIRED)
target_link_libraries(${name} ${Vulkan_LIBRARY})

include_directories(Vulkan-Hpp/Vulkan-Headers/include)

add_subdirectory(Vulkan-Hpp/glm)
include_directories(Vulkan-Hpp/glm)

add_subdirectory(Vulkan-Hpp/glslang)
include_directories(Vulkan-Hpp/glslang)
target_link_libraries(${name} glslang)
target_link_libraries(${name} SPIRV)
target_link_libraries(${name} glslang-default-resource-limits)


SET(GLFW_BUILD_DOCS OFF CACHE BOOL "GLFW_BUILD_DOCS" FORCE)
SET(GLFW_BUILD_TESTS OFF CACHE BOOL "GLFW_BUILD_TESTS" FORCE)
SET(GLFW_INSTALL OFF CACHE BOOL "GLFW_INSTALL" FORCE)
add_subdirectory(Vulkan-Hpp/glfw)
target_link_libraries(${name} glfw)


add_subdirectory(pugixml)
target_link_libraries(${name} pugixml)


SET(BUILD_TESTING OFF CACHE BOOL "BUILD_TESTING" FORCE)
SET(USE_MSVC_RUNTIME_LIBRARY_DLL OFF CACHE BOOL "USE_MSVC_RUNTIME_LIBRARY_DLL" FORCE)
SET(ZSTD_BUILD_PROGRAMS OFF CACHE BOOL "ZSTD_BUILD_PROGRAMS" FORCE)
SET(ZSTD_BUILD_SHARED OFF CACHE BOOL "ZSTD_BUILD_SHARED" FORCE)
SET(ZSTD_BUILD_TESTS OFF CACHE BOOL "ZSTD_BUILD_SHARED" FORCE)
SET(ZSTD_LEGACY_SUPPORT OFF CACHE BOOL "ZSTD_LEGACY_SUPPORT" FORCE)
add_subdirectory(zstd/build/cmake)
target_link_libraries(${name} libzstd_static)
include_directories(
	zstd/lib
)
