set(protoc_version 28.3)

if(CMAKE_HOST_SYSTEM_NAME STREQUAL "Linux")
    if(CMAKE_SYSTEM_PROCESSOR MATCHES "^(i[3-6]86|x86_64|x86)$")
        message(STATUS "Protobuf Intel Architecture")
        # Add Intel-specific configurations or settings here
        set(protoc_archive_name protoc-${protoc_version}-linux-x86_64.zip)
        set(protobuf_PROTOC_EXE ${CMAKE_CURRENT_BINARY_DIR}/protoc/bin/protoc)
    elseif(CMAKE_SYSTEM_PROCESSOR MATCHES "^(arm|aarch64)$")
        message(STATUS "Protobuf ARM Architecture")
        # Add ARM-specific configurations or settings here
        set(protoc_archive_name protoc-${protoc_version}-linux-aarch_64.zip)
        set(protobuf_PROTOC_EXE ${CMAKE_CURRENT_BINARY_DIR}/protoc/bin/protoc)
    else()
        message(STATUS "Unknown Architecture: ${CMAKE_SYSTEM_PROCESSOR}")
    endif()
elseif(CMAKE_HOST_SYSTEM_NAME STREQUAL "Windows")
    message(STATUS "Protobuf on Windows")
    set(protoc_archive_name protoc-${protoc_version}-win64.zip)
    set(protobuf_PROTOC_EXE ${CMAKE_CURRENT_BINARY_DIR}/protoc/bin/protoc.exe)
elseif(CMAKE_HOST_SYSTEM_NAME STREQUAL "Darwin")
    message(STATUS "Protobuf on macOS")
    set(protoc_archive_name protoc-${protoc_version}-osx-x86_64.zip)
    set(protobuf_PROTOC_EXE ${CMAKE_CURRENT_BINARY_DIR}/protoc/bin/protoc)
else()
    message(STATUS "Unknown host platform")
endif()

# FUll URL
set(protc_binaries_url https://github.com/protocolbuffers/protobuf/releases/download/v${protoc_version}/${protoc_archive_name})

# Download Archive
if(NOT EXISTS ${CMAKE_CURRENT_BINARY_DIR}/${protoc_archive_name})
    file(DOWNLOAD
        ${protc_binaries_url}
        ${CMAKE_CURRENT_BINARY_DIR}/${protoc_archive_name}
        TIMEOUT 300 #seconds
        TLS_VERIFY ON
        SHOW_PROGRESS)
endif()

# Extract Archive
# No protoc, will download native
file(ARCHIVE_EXTRACT INPUT ${CMAKE_CURRENT_BINARY_DIR}/${protoc_archive_name} DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/protoc)

# Add protoc to current path
set(CMAKE_PROGRAM_PATH ${CMAKE_CURRENT_BINARY_DIR}/protoc/bin ${CMAKE_PROGRAM_PATH})

set(MESSAGE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/../proto)

# Will get files relative to current directory meaning ../proto/*.proto
FILE(GLOB_RECURSE protofiles RELATIVE "${CMAKE_CURRENT_SOURCE_DIR}" "${MESSAGE_PATH}/*.proto")

message(STATUS "Protofiles: ${protofiles}")

add_custom_target(proto_files SOURCES ${protofiles})

# Include path
set(GOOGLE_PROTO_PATH ${CMAKE_CURRENT_BINARY_DIR}/protoc/include)

# Output directory for python/protobuf files
set(PROTOBUF_OUTPUT_PATH ${CMAKE_CURRENT_SOURCE_DIR})

# Targets
set(protobuf_python_targets)

foreach(file ${protofiles})
    message(STATUS "Processing ${file}")
    get_filename_component (name_without_extension ${file} NAME_WE)
    #Add target
    set(output_target opentera_messages_${name_without_extension}.proto.target)
    list(APPEND protobuf_python_targets ${output_target})
    #Add command to generate target
    add_custom_target(${output_target}
        COMMAND ${protobuf_PROTOC_EXE} ${file} -I=../proto/ -I${GOOGLE_PROTO_PATH} --python_out ${PROTOBUF_OUTPUT_PATH}
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        DEPENDS python-env ${file} )
endforeach(file)

add_custom_target(python-messages DEPENDS python-env ${protobuf_python_targets})
