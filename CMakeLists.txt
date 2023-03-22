cmake_minimum_required(VERSION 3.25)
project(OIYolo)

if (NOT CMAKE_INSTALL_PREFIX)
	set(CMAKE_INSTALL_PREFIX ${CMAKE_CURRENT_BINARY_DIR})
endif()

set(CMAKE_CXX_STANDARD 14)

option(OIYolo_INSTALL "install ncnn library and headers" ON)
option(OIYolo_OpenCV "use OpenCV library for inference" ON)
option(OIYolo_NCNN "use NCNN library for inference" ON)

include(cmake/Dependencies.cmake)

add_library(${PROJECT_NAME}
		src/ncnn_port/V8.cpp)

add_dependencies(${PROJECT_NAME} ${DEPENDENCIES_LIST})

target_include_directories(${PROJECT_NAME}
		PUBLIC
		${CMAKE_CURRENT_LIST_DIR}/include)

if(CMAKE_SYSTEM_NAME STREQUAL "Darwin" OR CMAKE_SYSTEM_NAME STREQUAL "Linux")
	if (OIYolo_OpenCV)
		find_package(OpenCV REQUIRED)
		target_include_directories(${PROJECT_NAME}
				PUBLIC
				${OpenCV_INCLUDE_DIRS})
		target_link_libraries(${PROJECT_NAME}
				PUBLIC
				${OpenCV_LIBS})
	endif()

	if (OIYolo_NCNN)
		message(STATUS NCNN_ROOT=${NCNN_ROOT})
		target_include_directories(${PROJECT_NAME}
				PUBLIC
				${NCNN_ROOT}/include)

		target_link_libraries(${PROJECT_NAME}
				PUBLIC
				${NCNN_ROOT}/lib/libncnn.a)
	endif()
else()
	if (OIYolo_NCNN)
		target_include_directories(${PROJECT_NAME}
				PRIVATE
				C:\\ncnn_test_inference\\ncnn\\build_embarc\\install\\include)

		target_link_libraries(${PROJECT_NAME}
				PRIVATE
				-LC:\\ncnn_test_inference\\ncnn\\build_embarc\\install\\lib ncnn.lib)
	else()
		message(FATAL_ERROR "OpenCV supported currently only on Darwin/Linux systems")
	endif()
endif()

include(GNUInstallDirs)
message(STATUS CMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX})
message(STATUS CMAKE_INSTALL_LIBDIR=${CMAKE_INSTALL_LIBDIR})
message(STATUS CMAKE_INSTALL_INCLUDEDIR=${CMAKE_INSTALL_INCLUDEDIR})
install(TARGETS ${PROJECT_NAME} EXPORT OIYolo
		ARCHIVE DESTINATION ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR}
		LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR}
		RUNTIME DESTINATION ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_BINDIR}
		)
file(GLOB OIYolo_PUBLIC_INCLUDES ${CMAKE_CURRENT_LIST_DIR}/include/OIYolo/*.hpp)
install(FILES
		${OIYolo_PUBLIC_INCLUDES}
		DESTINATION ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_INCLUDEDIR}/OIYolo/)

add_subdirectory(test)