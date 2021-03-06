GET_TARGET_PROPERTY(UMUNDONATIVEJAVA_LOCATION umundoNativeJava LOCATION)
if (UMUNDONATIVEJAVA_LOCATION)
	add_dependencies(ALL_TESTS java)
	file(GLOB_RECURSE JAVA_TESTS "${PROJECT_SOURCE_DIR}/tests/*Test*.java")
	foreach(TEST ${JAVA_TESTS})
		get_filename_component(TEST_NAME ${TEST} NAME_WE)
		get_filename_component(TEST_DIR ${TEST} PATH)
		# strip java package structure
		STRING(REGEX REPLACE "org/umundo/.*" "" TEST_DIR_ROOT ${TEST_DIR})
		STRING(REGEX REPLACE "${PROJECT_SOURCE_DIR}/tests/.*/java/src/?" "" TEST_DIR_REST ${TEST_DIR})
		if (TEST_DIR_REST)
			STRING(REGEX REPLACE "/" "." TEST_PACKAGE ${TEST_DIR_REST})
		else()
			set(TEST_PACKAGE "")
		endif()
	
		add_test("java/${TEST_NAME}"
			"${ANT_EXECUTABLE}"
			-Dlib.dir=${JAR_JNI_ROOT_PATH}
			-Dsrc.dir=${PROJECT_SOURCE_DIR}
			-Dbuild.dir=${PROJECT_BINARY_DIR}
			-Dbuild.type=${CMAKE_BUILD_TYPE}
			-Dtest.class="${TEST_PACKAGE}.${TEST_NAME}"
			-Dtest.dir=${TEST_DIR_ROOT}
			-f ${PROJECT_SOURCE_DIR}/contrib/java/build-java.xml
			test)
		set_property(TEST "java/${TEST_NAME}" PROPERTY LABELS "java/${TEST_NAME}")
		set_property(TEST "java/${TEST_NAME}" PROPERTY REQUIRED_FILES "${PROJECT_BINARY_DIR}/lib/umundo.jar")
		
	endforeach()
endif()

add_executable(test-core-message-transmission test-message-transmission.cpp)
target_link_libraries(test-core-message-transmission umundo)
add_test(test-core-message-transmission ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/test-core-message-transmission)
set_target_properties(test-core-message-transmission PROPERTIES FOLDER "Tests")
add_dependencies(ALL_TESTS test-core-message-transmission)

add_executable(test-core-stress test-stress.cpp)
target_link_libraries(test-core-stress umundo)
# add_test(test-core-stress ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/test-core-stress)
set_target_properties(test-core-stress PROPERTIES FOLDER "Tests")
add_dependencies(ALL_TESTS test-core-stress)

add_executable(test-stack-api test-stack-api.cpp)
target_link_libraries(test-stack-api umundo)
add_test(test-stack-api ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/test-core-stress)
set_target_properties(test-stack-api PROPERTIES FOLDER "Tests")
add_dependencies(ALL_TESTS test-stack-api)

add_executable(test-core-greeter test-greeter.cpp)
target_link_libraries(test-core-greeter umundo)
add_test(test-stack-api ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/test-core-greeter)
set_target_properties(test-core-greeter PROPERTIES FOLDER "Tests")
add_dependencies(ALL_TESTS test-core-greeter)

# make sure all headers are self-reliant
set (UMUNDO_PUBLIC_HEADERS 
	"${PROJECT_SOURCE_DIR}/src/umundo.h"
)

if (APPLE)
	set (UMUNDO_PUBLIC_HEADERS 
		${UMUNDO_PUBLIC_HEADERS}
		"${PROJECT_SOURCE_DIR}/src/bindings/objc/umundo-objc.h"
	)
endif()

foreach(UMUNDO_HEADER ${UMUNDO_PUBLIC_HEADERS} )
	STRING(REGEX REPLACE "${PROJECT_SOURCE_DIR}/src/" "" UMUNDO_REL_HEADER ${UMUNDO_HEADER})

	set(HEADER_TEST)
	set(HEADER_TEST "${HEADER_TEST}#include \"${UMUNDO_HEADER}\"\n")
	set(HEADER_TEST "${HEADER_TEST}int main(int argc, char** argv) {}")
	FILE(WRITE ${CMAKE_CURRENT_BINARY_DIR}/headers/${UMUNDO_REL_HEADER}.cpp ${HEADER_TEST})

	add_test(NAME "header/${UMUNDO_REL_HEADER}"
		COMMAND ${CMAKE_COMMAND}
			-DTESTFILE:FILEPATH=${CMAKE_CURRENT_BINARY_DIR}/headers/${UMUNDO_REL_HEADER}.cpp
			-DCC_BIN:FILEPATH=${CMAKE_C_COMPILER}
			-DCXX_BIN:FILEPATH=${CMAKE_CXX_COMPILER}
			-DPROJECT_SOURCE_DIR=${PROJECT_SOURCE_DIR}
			-DUMUNDO_PLATFORM_ID=${UMUNDO_PLATFORM_ID}
			-DCMAKE_BINARY_DIR=${CMAKE_BINARY_DIR}
			-DPROJECT_BINARY_DIR=${PROJECT_BINARY_DIR}
			-DCMAKE_LIBRARY_OUTPUT_DIRECTORY=${CMAKE_LIBRARY_OUTPUT_DIRECTORY}
		-P ${CMAKE_CURRENT_SOURCE_DIR}/scripts/run_header_compiles.cmake)
	set_property(TEST "header/${UMUNDO_REL_HEADER}" PROPERTY LABELS "header/${UMUNDO_REL_HEADER}")
	
endforeach()

if(NOT WIN32)
	# can't be bothered at the moment - go get an actual operating system .. *sigh*
	add_executable(test-core-threads test-threads.cpp)
	target_link_libraries(test-core-threads umundo)
	set_target_properties(test-core-threads PROPERTIES FOLDER "Tests")
	# add_test(test-core-threads ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/test-core-threads)
	add_dependencies(ALL_TESTS test-core-threads)
	
	add_executable(test-core-domains test-domains.cpp)
	target_link_libraries(test-core-domains umundo)
	# add_test(test-core-domains ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/test-core-domains)
	set_target_properties(test-core-domains PROPERTIES FOLDER "Tests")
	add_dependencies(ALL_TESTS test-core-domains)
	
	add_executable(test-core-discovery test-discovery.cpp)
	target_link_libraries(test-core-discovery umundo)
	add_test(test-core-discovery ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/test-core-discovery)
	set_target_properties(test-core-discovery PROPERTIES FOLDER "Tests")
	add_dependencies(ALL_TESTS test-core-discovery)

	add_executable(test-lz4-dict 
		test-lz4-dict.cpp 
		../contrib/src/lz4/xxhash.c 
		../contrib/src/lz4/lz4.c 
		../contrib/src/lz4/lz4hc.c)
	target_link_libraries(test-lz4-dict umundo)
	add_test(test-core-discovery ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/test-lz4-dict)
	set_target_properties(test-lz4-dict PROPERTIES FOLDER "Tests")
	add_dependencies(ALL_TESTS test-lz4-dict)
	
	add_executable(test-zeromq-disconnect test-zeromq-disconnect.cpp)
	target_link_libraries(test-zeromq-disconnect umundo)
	set_target_properties(test-zeromq-disconnect PROPERTIES FOLDER "Tests")
	add_dependencies(ALL_TESTS test-zeromq-disconnect)

	add_executable(test-zeromq-nodecomm test-zeromq-nodecomm.cpp)
	target_link_libraries(test-zeromq-nodecomm umundo)
	set_target_properties(test-zeromq-nodecomm PROPERTIES FOLDER "Tests")
	add_dependencies(ALL_TESTS test-zeromq-nodecomm)
endif()

if(DISC_AVAHI)
	add_executable(test-avahi-stress test-avahi-stress.cpp)
	target_link_libraries(test-avahi-stress umundo)
	set_target_properties(test-avahi-stress PROPERTIES FOLDER "Tests")
	add_dependencies(ALL_TESTS test-avahi-stress)
endif()

if (NET_RTP)
	add_executable(test-rtp-pubsub test-rtp-pubsub.cpp)
	target_link_libraries(test-rtp-pubsub umundo)
	# add_test(test-rtp-pubsub ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/test-rtp-pubsub)
	set_target_properties(test-rtp-pubsub PROPERTIES FOLDER "Tests")
	add_dependencies(ALL_TESTS test-rtp-pubsub)
endif()