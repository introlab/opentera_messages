message(STATUS "Processing messages")

if (MSVC)
    add_definitions(
      /wd4910 # Ignore export warnings (C4910) with MSVC
      /wd4661 # Ignore no suitable definition for explicit template (C4661)
      /wd4251 # Ignore needs to have dll-interface
      /wd4275 # Ignore non-dll interface class used as base for dll-interface class
    )
endif (MSVC)


set (MESSAGE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/../proto)
FILE(GLOB_RECURSE messages RELATIVE "${MESSAGE_PATH}" "${MESSAGE_PATH}/*.proto")
FILE(GLOB_RECURSE messages_sources RELATIVE "${CMAKE_CURRENT_SOURCE_DIR}" "${MESSAGE_PATH}/*.proto")
add_custom_target(proto_files SOURCES ${messages_sources})

#Message files output directory, will create directory
set (output_directory ${CMAKE_CURRENT_BINARY_DIR})
file(MAKE_DIRECTORY ${output_directory})

set (generated_headers "")
set (generated_srcs "")

# Include path
set(GOOGLE_PROTO_PATH ${PROJECT_SOURCE_DIR}/external/protobuf/src)

message(STATUS "GOOGLE_PROTO_PATH: ${GOOGLE_PROTO_PATH}")

#Generate code using protoc compiler
foreach (file ${messages})

    message(STATUS "Processing : ${file}")

    get_filename_component (name_without_extension ${file} NAME_WE)

    message(STATUS "name_without_extension : ${name_without_extension}")

    set(output_header_file ${output_directory}/${name_without_extension}.pb.h)
    set(output_src_file ${output_directory}/${name_without_extension}.pb.cc)

    #Complete files to be compiled...
    list(APPEND generated_headers ${output_header_file})
    list(APPEND generated_srcs ${output_src_file})

    #Should execute protoc on each .proto files
    #Ex: protoc -I=$SRC_DIR --cpp_out=$DST_DIR $SRC_DIR/addressbook.proto

    if (MSVC)
    add_custom_command(OUTPUT ${output_header_file} ${output_src_file}
                   COMMAND protoc -I=${MESSAGE_PATH} -I${GOOGLE_PROTO_PATH} --cpp_out=dllexport_decl=OPENTERAMESSAGES_EXPORT:${output_directory} ${file}
                   DEPENDS ${MESSAGE_PATH}/${file} protoc
                   WORKING_DIRECTORY ${MESSAGE_PATH}
                   VERBATIM)
     else(MSVC)
     add_custom_command(OUTPUT ${output_header_file} ${output_src_file}
                    COMMAND protoc -I=${MESSAGE_PATH} -I${GOOGLE_PROTO_PATH} --cpp_out=${output_directory} ${file}
                    DEPENDS ${MESSAGE_PATH}/${file} protoc
                    WORKING_DIRECTORY ${MESSAGE_PATH}
                    VERBATIM)
     endif(MSVC)
endforeach()


message (STATUS "Headers : ${generated_headers}")
message (STATUS "Sources : ${generated_srcs}")

include_directories(
    ${PROJECT_SOURCE_DIR}/submodules/protobuf/src
    ${output_directory}
)

if (MSVC)
add_definitions(-DOPENTERAMESSAGES_EXPORT=__declspec\(dllexport\))
endif(MSVC)

#compile library
add_library(opentera_messages SHARED ${generated_headers} ${generated_srcs})
#Link with protobuf
target_link_libraries(opentera_messages libprotobuf)
target_compile_definitions(opentera_messages PUBLIC OPENTERA_PROTOCOL_VERSION=1)

get_target_property(all_includes opentera_messages INCLUDE_DIRECTORIES)

message (STATUS "includes : ${all_includes}")

set(OPENTERA_MESSAGES_INCLUDES ${all_includes} CACHE INTERNAL "doc string")
set(OPENTERA_MESSAGES_LIBS "opentera_messages" CACHE INTERNAL "doc string")
