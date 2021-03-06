project(pilight_hardware)

file(GLOB ${PROJECT_NAME}_sources "${PROJECT_SOURCE_DIR}/*.c" "${PROJECT_SOURCE_DIR}/*.cpp")
file(GLOB ${PROJECT_NAME}_headers "${PROJECT_SOURCE_DIR}/*.h")

if(${MODULESPACK} MATCHES "OFF" OR WIN32)
	include(CMakeExclude.txt)

	if(WIN32)
		list(REMOVE_ITEM ${PROJECT_NAME}_sources "${PROJECT_SOURCE_DIR}/433gpio.c")
		list(REMOVE_ITEM ${PROJECT_NAME}_headers "${PROJECT_SOURCE_DIR}/433gpio.h")
		list(REMOVE_ITEM ${PROJECT_NAME}_sources "${PROJECT_SOURCE_DIR}/433lirc.c")
		list(REMOVE_ITEM ${PROJECT_NAME}_headers "${PROJECT_SOURCE_DIR}/433lirc.h")
	endif()
	
	if(${ZWAVE} MATCHES "OFF")
		list(REMOVE_ITEM ${PROJECT_NAME}_sources "${PROJECT_SOURCE_DIR}/zwave.cpp")
		list(REMOVE_ITEM ${PROJECT_NAME}_headers "${PROJECT_SOURCE_DIR}/zwave.h")
	endif()

	include(CMakeInclude.txt)
	
	set(${PROJECT_NAME}_files ${${PROJECT_NAME}_sources} ${${PROJECT_NAME}_headers} CACHE INTERNAL "")
else()
	foreach(file ${${PROJECT_NAME}_sources})
		get_filename_component(name ${file} NAME_WE)
		add_library(${name}_module MODULE ${file})
		target_link_libraries(${name}_module ${CMAKE_THREAD_LIBS_INIT})
		target_link_libraries(${name}_module m)
		if(${CMAKE_SYSTEM_NAME} MATCHES "FreeBSD")
			target_link_libraries(${name}_module execinfo)
		endif()
		target_link_libraries(${name}_module ${CMAKE_DL_LIBS})
		set_target_properties(${name}_module PROPERTIES OUTPUT_NAME ${name})
		string(REPLACE "_" "-" debname ${name})
		install(FILES ${CMAKE_BINARY_DIR}/libs/pilight/hardware/lib${name}.so DESTINATION lib/pilight/hardware RENAME ${name}.so COMPONENT pilight-hardware-${debname})
	endforeach()
endif()
