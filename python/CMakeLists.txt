# Protobuf files
#SET(protofiles
#    sample.proto
#)

set(MESSAGE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/../proto)

# Will get files relative to current directory meaning ../proto/*.proto
FILE(GLOB_RECURSE protofiles RELATIVE "${CMAKE_CURRENT_SOURCE_DIR}" "${MESSAGE_PATH}/*.proto")

message(STATUS "Protofiles: ${protofiles}")

add_custom_target(proto_files SOURCES ${protofiles})

# Include path
if(WIN32)
    set(GOOGLE_PROTO_PATH ${PYTHON_ENV_DIRECTORY}/Library/include/google/protobuf/)
else(WIN32)
    set(GOOGLE_PROTO_PATH ${PYTHON_ENV_DIRECTORY}/include/google/protobuf/)
endif(WIN32)


# Output directory for python/protobuf files
set(PROTOBUF_OUTPUT_PATH ${CMAKE_CURRENT_SOURCE_DIR})

# Targets
set(protobuf_python_targets)

foreach(file ${protofiles})
    message(STATUS "Processing ${file}")
    get_filename_component (name_without_extension ${file} NAME_WE)
    # message(STATUS "Without extension ${name_without_extension}")
    #Add target
    set(output_target opentera_messages_${name_without_extension}.proto.target)
    list(APPEND protobuf_python_targets ${output_target})
    #Add command to generate target
    add_custom_target(${output_target}
        COMMAND ${PROTOC_EXECUTABLE} ${file} -I=../proto/ -I${GOOGLE_PROTO_PATH} --python_out ${PROTOBUF_OUTPUT_PATH}
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        DEPENDS python-env ${file} )
endforeach(file)

add_custom_target(python-messages DEPENDS python-env ${protobuf_python_targets})
