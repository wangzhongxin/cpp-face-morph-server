cmake_minimum_required (VERSION 2.8)
project (face-morph)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin/)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/modules/")
set(CMAKE_CONFIG_DIR etc/face-morph)
set(CONFIG_DIR "${CMAKE_INSTALL_PREFIX}/${CMAKE_CONFIG_DIR}")
add_definitions(-DCONFIG_DIR="${CONFIG_DIR}")

find_package( OpenCV REQUIRED )

MESSAGE("OpenCV information:") 
MESSAGE("  OpenCV_INCLUDE_DIRS: ${OpenCV_INCLUDE_DIRS}") 
MESSAGE("  OpenCV_LIBRARIES: ${OpenCV_LIBRARIES}") 
MESSAGE("  OpenCV_LIBRARY_DIRS: ${OpenCV_LIBRARY_DIRS}") 
include_directories( ${OpenCV_INCLUDE_DIRS} )

find_package( Boost 1.5.9 REQUIRED COMPONENTS filesystem system)
MESSAGE("Boost information:") 
MESSAGE("  Boost_INCLUDE_DIRS: ${Boost_INCLUDE_DIRS}") 
MESSAGE("  Boost_LIBRARIES: ${Boost_LIBRARIES}") 
MESSAGE("  Boost_LIBRARY_DIRS: ${Boost_LIBRARY_DIRS}") 

INCLUDE_DIRECTORIES(${Boost_INCLUDE_DIRS})
INCLUDE_DIRECTORIES(${Boost_INCLUDE_DIRS}/boost)
LINK_DIRECTORIES(${Boost_LIBRARY_DIRS})	

find_package(TBB REQUIRED)

# Move fld-core model
file(GLOB files "lib/local/fld-core/model/*.txt")
foreach(file ${files})
	if (MSVC)
		file(COPY ${file} DESTINATION ${CMAKE_BINARY_DIR}/bin/Debug/model)
		file(COPY ${file} DESTINATION ${CMAKE_BINARY_DIR}/bin/Release/model)
	else(MSVC)
		file(COPY ${file} DESTINATION ${CMAKE_BINARY_DIR}/bin/model)
	endif(MSVC)

	install(FILES ${file} DESTINATION ${CMAKE_CONFIG_DIR}/model)
endforeach()

# Move the hierarchical fld-core models
file(GLOB files "lib/local/fld-core/model/model*")
foreach(file ${files})
	if (MSVC)
		file(COPY ${file} DESTINATION ${CMAKE_BINARY_DIR}/bin/Debug/model)
		file(COPY ${file} DESTINATION ${CMAKE_BINARY_DIR}/bin/Release/model)
	else(MSVC)
		file(COPY ${file} DESTINATION ${CMAKE_BINARY_DIR}/bin/model)
	endif(MSVC)

	install(DIRECTORY ${file} DESTINATION ${CMAKE_CONFIG_DIR}/model)
endforeach()

file(GLOB files "lib/local/fld-core/model/detection_validation/*.txt")
foreach(file ${files})
	if (MSVC)
		file(COPY ${file} DESTINATION ${CMAKE_BINARY_DIR}/bin/Debug/model/detection_validation)
		file(COPY ${file} DESTINATION ${CMAKE_BINARY_DIR}/bin/Release/model/detection_validation)
	else(MSVC)
		file(COPY ${file} DESTINATION ${CMAKE_BINARY_DIR}/bin/model/detection_validation)
	endif(MSVC)

	install(FILES ${file} DESTINATION ${CMAKE_CONFIG_DIR}/model/detection_validation)
endforeach()

file(GLOB files "lib/local/fld-core/model/patch_experts/*.txt")
foreach(file ${files})
	if (MSVC)
		file(COPY ${file} DESTINATION ${CMAKE_BINARY_DIR}/bin/Debug/model/patch_experts)
		file(COPY ${file} DESTINATION ${CMAKE_BINARY_DIR}/bin/Release/model/patch_experts)
	else(MSVC)
		file(COPY ${file} DESTINATION ${CMAKE_BINARY_DIR}/bin/model/patch_experts)
	endif(MSVC)

	install(FILES ${file} DESTINATION ${CMAKE_CONFIG_DIR}/model/patch_experts)
endforeach()

file(GLOB files "lib/local/fld-core/model/pdms/*.txt")
foreach(file ${files})
	if (MSVC)
		file(COPY ${file} DESTINATION ${CMAKE_BINARY_DIR}/bin/Debug/model/pdms)
		file(COPY ${file} DESTINATION ${CMAKE_BINARY_DIR}/bin/Release/model/pdms)
	else(MSVC)
		file(COPY ${file} DESTINATION ${CMAKE_BINARY_DIR}/bin/model/pdms)
	endif(MSVC)

	install(FILES ${file} DESTINATION ${CMAKE_CONFIG_DIR}/model/pdms)
endforeach()

# Move OpenCV classifiers
file(GLOB files "lib/3rdParty/OpenCV3.1/classifiers/*.xml")
foreach(file ${files})
	if (MSVC)
		file(COPY ${file} DESTINATION ${CMAKE_BINARY_DIR}/bin/Debug/classifiers)
		file(COPY ${file} DESTINATION ${CMAKE_BINARY_DIR}/bin/Release/classifiers)
	else(MSVC)
		file(COPY ${file} DESTINATION ${CMAKE_BINARY_DIR}/bin/classifiers)
	endif(MSVC)

	install(FILES ${file} DESTINATION ${CMAKE_CONFIG_DIR}/classifiers)
endforeach()

if (${CMAKE_CXX_COMPILER_ID} STREQUAL "GNU")
    execute_process(COMMAND ${CMAKE_CXX_COMPILER} -dumpversion OUTPUT_VARIABLE GCC_VERSION)
    if (GCC_VERSION VERSION_LESS 4.7)
        set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x -msse -msse2 -msse3")
    else ()
        set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -msse -msse2 -msse3")
    endif ()
else ()
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -msse -msse2 -msse3")
endif ()

# Boost
if(WIN32)
	include_directories(lib/3rdParty/boost)
	include_directories(lib/3rdParty/boost/boost)
	link_directories( ${PROJECT_SOURCE_DIR}/lib/3rdParty/boost/lib )
else()
	INCLUDE_DIRECTORIES(${BOOST_INCLUDE_DIR})
endif()
set(CMAKE_EXE_LINKER_FLAGS "-lboost_thread -lpthread")

# dlib
include_directories(lib/3rdParty/dlib/include)

# dlib library
add_subdirectory(lib/3rdParty/dlib)

# base64
include_directories(lib/3rdParty/base64)

# json
include_directories(lib/3rdParty/json)

# fld-core library
add_subdirectory(lib/local/fld-core)

# MORPHER library
include_directories(lib/local/morpher)
add_subdirectory(lib/local/morpher)

# fld-server library
add_subdirectory(lib/local/fld-server)

#### executables
add_subdirectory(exe/face-morph-server)
