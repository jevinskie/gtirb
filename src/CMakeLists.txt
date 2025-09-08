set(PROJECT_NAME gtirb)

# add_subdirectory(gtirb/proto)

set(GTIRB_PROTO_FILES
  AuxData.proto
  ByteInterval.proto
  CFG.proto
  CMakeLists.txt
  CodeBlock.proto
  DataBlock.proto
  IR.proto
  Module.proto
  Offset.proto
  ProxyBlock.proto
  Section.proto
  Symbol.proto
  SymbolicExpression.proto
  v0/AuxData.proto
  v0/AuxDataContainer.proto
  v0/Block.proto
  v0/ByteMap.proto
  v0/CFG.proto
  v0/CMakeLists.txt
  v0/DataObject.proto
  v0/IR.proto
  v0/ImageByteMap.proto
  v0/Module.proto
  v0/Offset.proto
  v0/ProxyBlock.proto
  v0/Section.proto
  v0/Symbol.proto
  v0/SymbolicExpression.proto
)

set(GTIRB_PROTO_FILE_PATHS)
foreach(PB ${GTIRB_PROTO_FILES})
    set(PB "${CMAKE_CURRENT_SOURCE_DIR}/../proto/${PB}")
    list(APPEND GTIRB_PROTO_FILE_PATHS ${PB})
endforeach()

message(WARNING "GTIRB_PROTO_FILE_PATHS: ${GTIRB_PROTO_FILE_PATHS}")

add_library(gtirb_proto OBJECT ${GTIRB_PROTO_FILE_PATHS})
target_link_libraries(gtirb_proto PUBLIC protobuf::libprotobuf)
set(PROTO_BINARY_DIR "${CMAKE_CURRENT_BINARY_DIR}/gtirb/proto")
target_include_directories(gtirb_proto PUBLIC "$<BUILD_INTERFACE:${PROTO_BINARY_DIR}>")


protobuf_generate(
    TARGET gtirb_proto
    IMPORT_DIRS "${CMAKE_CURRENT_SOURCE_DIR}/../proto"
    PROTOC_OUT_DIR "${PROTO_BINARY_DIR}")

# protobuf_generate(
#     TARGET gtirb_proto
#     IMPORT_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/../proto
#     PROTOC_OUT_DIR ${CMAKE_CURRENT_BINARY_DIR}/proto)

install(
  TARGETS gtirb_proto
  # COMPONENT proto_library
  EXPORT gtirbTargets
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
)

if(GTIRB_RUN_CLANG_TIDY)
  find_program(
    CLANG_TIDY_EXE
    NAMES "clang-tidy"
    DOC "Path to clang-tidy executable"
  )
  if(NOT CLANG_TIDY_EXE)
    message(STATUS "clang-tidy not found.")
  else()
    message(STATUS "clang-tidy found: ${CLANG_TIDY_EXE}")
  endif()
else()
  message(STATUS "clang-tidy disabled.")
endif()

if(${CMAKE_CXX_COMPILER_ID} STREQUAL MSVC)
  # add_compile_options(-wd4251)  # Non-exportable template classes.
  add_compile_options(-DBOOST_UUID_RANDOM_PROVIDER_FORCE_WINCRYPT)
elseif(${CMAKE_CXX_COMPILER_ID} STREQUAL GNU)
  # add_compile_options(-Wno-unused-function)
  add_compile_options(-mtune=generic)
  add_compile_options(-pthread)
elseif(${CMAKE_CXX_COMPILER_ID} STREQUAL Clang)
  # add_compile_options(-Wno-unused-function)
  add_compile_options(-mtune=generic)
  add_compile_options(-pthread)
endif()

# generate version.h from version.h.in
configure_file(
  "${GTIRB_SOURCE_DIR}/include/gtirb/version.h.in"
  "${GTIRB_BINARY_DIR}/include/gtirb/version.h" @ONLY
)

# specify header files that need to be installed
set(${PROJECT_NAME}_H
    "${GTIRB_SOURCE_DIR}/include/gtirb/Addr.hpp"
    "${GTIRB_SOURCE_DIR}/include/gtirb/Allocator.hpp"
    "${GTIRB_SOURCE_DIR}/include/gtirb/AuxData.hpp"
    "${GTIRB_SOURCE_DIR}/include/gtirb/AuxDataContainer.hpp"
    "${GTIRB_SOURCE_DIR}/include/gtirb/AuxDataSchema.hpp"
    "${GTIRB_SOURCE_DIR}/include/gtirb/ByteInterval.hpp"
    "${GTIRB_SOURCE_DIR}/include/gtirb/CFG.hpp"
    "${GTIRB_SOURCE_DIR}/include/gtirb/Casting.hpp"
    "${GTIRB_SOURCE_DIR}/include/gtirb/CfgNode.hpp"
    "${GTIRB_SOURCE_DIR}/include/gtirb/CodeBlock.hpp"
    "${GTIRB_SOURCE_DIR}/include/gtirb/Context.hpp"
    "${GTIRB_SOURCE_DIR}/include/gtirb/DataBlock.hpp"
    "${GTIRB_SOURCE_DIR}/include/gtirb/DecodeMode.hpp"
    "${GTIRB_SOURCE_DIR}/include/gtirb/ErrorOr.hpp"
    "${GTIRB_SOURCE_DIR}/include/gtirb/Export.hpp"
    "${GTIRB_SOURCE_DIR}/include/gtirb/IR.hpp"
    "${GTIRB_SOURCE_DIR}/include/gtirb/Module.hpp"
    "${GTIRB_SOURCE_DIR}/include/gtirb/Node.hpp"
    "${GTIRB_SOURCE_DIR}/include/gtirb/Observer.hpp"
    "${GTIRB_SOURCE_DIR}/include/gtirb/Offset.hpp"
    "${GTIRB_SOURCE_DIR}/include/gtirb/ProxyBlock.hpp"
    "${GTIRB_SOURCE_DIR}/include/gtirb/Section.hpp"
    "${GTIRB_SOURCE_DIR}/include/gtirb/Symbol.hpp"
    "${GTIRB_SOURCE_DIR}/include/gtirb/SymbolicExpression.hpp"
    "${GTIRB_SOURCE_DIR}/include/gtirb/Utility.hpp"
    "${GTIRB_SOURCE_DIR}/include/gtirb/gtirb.hpp"
)

# specify source files
set(${PROJECT_NAME}_SRC
    AuxData.cpp
    AuxDataContainer.cpp
    ByteInterval.cpp
    CodeBlock.cpp
    Context.cpp
    CFG.cpp
    DataBlock.cpp
    ErrorOr.cpp
    IR.cpp
    Module.cpp
    Node.cpp
    Offset.cpp
    ProxyBlock.cpp
    Section.cpp
    Serialization.cpp
    Symbol.cpp
    SymbolicExpression.cpp
    Utility.cpp
)

# file(GLOB ProtoFiles "${CMAKE_CURRENT_SOURCE_DIR}/gtirb/proto/*.proto")
# set(${PROJECT_NAME}_PROTO ${ProtoFiles})
# source_group("proto" FILES ${ProtoFiles})

if(UNIX AND NOT WIN32)
  set(SYSLIBS ${CMAKE_DL_LIBS})
else()
  set(SYSLIBS)
endif()

gtirb_add_library()
# FIXME We should be setting SOVERSION with an ABI version number once the ABI
# stabilizes. For now, since most patches affect the ABI anyway, we just use the
# GTIRB_VERSION as the SOVERSION.
set_target_properties(${PROJECT_NAME} PROPERTIES VERSION ${GTIRB_VERSION})
if(WIN32)
  # FIXME: We need to figure out how to handle cutting a release build. For the
  # moment, treat everything as a pre-release build.
  set(GTIRB_IS_PRERELEASE 1)
  configure_file(
    ${GTIRB_SOURCE_DIR}/resources/windows_version_resource.rc.in
    ${GTIRB_BINARY_DIR}/version.rc @ONLY
  )
  target_sources(${PROJECT_NAME} PRIVATE ${GTIRB_BINARY_DIR}/version.rc)
  source_group(Resources FILES ${GTIRB_BINARY_DIR}/version.rc)
endif()

# Find our headers
target_include_directories(
  ${PROJECT_NAME} PUBLIC $<BUILD_INTERFACE:${GTIRB_SOURCE_DIR}/include>
)
target_include_directories(
  ${PROJECT_NAME} PUBLIC $<BUILD_INTERFACE:${GTIRB_BINARY_DIR}/include>
)
target_include_directories(
  ${PROJECT_NAME} PRIVATE $<BUILD_INTERFACE:${GTIRB_SOURCE_DIR}/include/gtirb>
)
target_include_directories(
  ${PROJECT_NAME} PRIVATE $<BUILD_INTERFACE:${GTIRB_BINARY_DIR}/src/>
)

# NOTE (2020-12): tried using protobuf::libprotobuf to get relocatable exports,
# but it is apparently treated inconsistently between static and shared builds.
# For simplicity, we revert to using ${Protobuf_LIBRARIES} for now.

target_link_libraries(
  ${PROJECT_NAME}
  PUBLIC ${SYSLIBS}
         ${Boost_LIBRARIES}
        #  protobuf::libprotobuf
         gtirb_proto
         # Link in this static lib, but don't make it a transitive dependency of
         # TestGTIRB, etc
  # PRIVATE gtirb_proto
)
target_compile_definitions(
  ${PROJECT_NAME} PRIVATE GTIRB_${PROJECT_NAME}_EXPORTS
  GTIRB_WRAP_UTILS_IN_NAMESPACE
)
target_include_directories(${PROJECT_NAME} PUBLIC "${PROTOBUF_INCLUDE_DIRS}")

if(${CMAKE_CXX_COMPILER_ID} STREQUAL MSVC)
  # These four warnings come from protobuf headers, disabling them this way
  # means that projects which link to gtirb via cmake won't have to deal with
  # them.
  target_compile_options(${PROJECT_NAME} PUBLIC -wd4100) # unreferenced formal
                                                         # parameter
  target_compile_options(
    ${PROJECT_NAME} PUBLIC -wd4127
  ) # conditional expression is constant
  target_compile_options(
    ${PROJECT_NAME} PUBLIC -wd4244
  ) # conversion from 'type1' to 'type2', possible loss of data
  target_compile_definitions(
    ${PROJECT_NAME} PUBLIC
    _SILENCE_CXX17_ITERATOR_BASE_CLASS_DEPRECATION_WARNING
  )

  target_compile_options(
    ${PROJECT_NAME} PUBLIC -wd4251
  ) # 'identifier' : class 'type' needs to have dll- interface to be used by
    # clients of class 'type2'
  target_compile_options(${PROJECT_NAME} PUBLIC -wd4275) # Non-dll interface
                                                         # base classes.
endif()

if(UNIX
   AND NOT CYGWIN
   AND ("${CMAKE_BUILD_TYPE}" STREQUAL "RelWithDebInfo" OR "${CMAKE_BUILD_TYPE}"
                                                           STREQUAL "Debug")
   AND ${GTIRB_STRIP_DEBUG_SYMBOLS}
)
  string(
    RANDOM
    LENGTH 32
    ALPHABET "abcdef0123456789" BUILD_ID
  )
  string(SUBSTRING "${BUILD_ID}" 0 2 BUILD_ID_PREFIX)
  string(SUBSTRING "${BUILD_ID}" 2 32 BUILD_ID_SUFFIX)
  target_link_libraries(${PROJECT_NAME} PRIVATE "-Wl,--build-id=0x${BUILD_ID}")
  add_custom_command(
    TARGET ${PROJECT_NAME}
    POST_BUILD
    COMMAND objcopy --only-keep-debug $<TARGET_FILE:${PROJECT_NAME}>
            ${GTIRB_BINARY_DIR}/bin/${BUILD_ID_SUFFIX}.debug
    COMMAND objcopy --strip-debug $<TARGET_FILE:${PROJECT_NAME}>
  )
  install(
    FILES "${GTIRB_BINARY_DIR}/bin/${BUILD_ID_SUFFIX}.debug"
    COMPONENT debug-file
    DESTINATION "lib/debug/.build-id/${BUILD_ID_PREFIX}"
  )
endif()

if(CLANG_TIDY_EXE)
  set_target_properties(
    ${PROJECT_NAME} PROPERTIES CXX_CLANG_TIDY "${CLANG_TIDY_EXE}"
  )
endif()

if(GTIRB_ENABLE_TESTS)
  add_subdirectory(test)
endif()

install(
  TARGETS ${PROJECT_NAME}
  COMPONENT library
  EXPORT gtirbTargets
  INCLUDES
  DESTINATION include
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
)
install(
  FILES ${${PROJECT_NAME}_H}
  COMPONENT headers
  DESTINATION include/gtirb
)
install(
  DIRECTORY "${GTIRB_BINARY_DIR}/include/gtirb/proto"
  COMPONENT headers
  DESTINATION include/gtirb
)
install(
  FILES "${GTIRB_BINARY_DIR}/include/gtirb/version.h"
  COMPONENT headers
  DESTINATION include/gtirb
)
