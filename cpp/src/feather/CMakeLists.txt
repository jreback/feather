# Copyright 2012 Feather Developers
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

set(LIBFEATHER_SRCS
  buffer.cc
  io.cc
  metadata.cc
  reader.cc
  status.cc
  types.cc
  writer.cc
)

set(LIBFEATHER_LINK_LIBS
)

set(LIBFEATHER_LINKAGE "SHARED")

add_library(feather
  ${LIBFEATHER_LINKAGE}
  ${LIBFEATHER_SRCS}
)

set_target_properties(feather
  PROPERTIES
  LIBRARY_OUTPUT_DIRECTORY "${BUILD_OUTPUT_ROOT_DIRECTORY}")
target_link_libraries(feather ${LIBFEATHER_LINK_LIBS})
set_target_properties(feather PROPERTIES LINKER_LANGUAGE CXX)

if(APPLE)
    set_target_properties(feather PROPERTIES LINK_FLAGS "-undefined dynamic_lookup")
endif()

add_library(feather_test_main
  tests/test_main.cc)

if (APPLE)
  target_link_libraries(feather_test_main
    gtest
    dl)
  set_target_properties(feather_test_main
        PROPERTIES LINK_FLAGS "-undefined dynamic_lookup")
else()
  target_link_libraries(feather_test_main
    dl
    gtest
    pthread
  )
endif()

ADD_FEATHER_TEST(tests/io-test)
ADD_FEATHER_TEST(tests/metadata-test)
ADD_FEATHER_TEST(tests/writer-test)

# make clean will delete the generated file
set_source_files_properties(metadata_generated.h PROPERTIES GENERATED TRUE)

set(OUTPUT_DIR ${CMAKE_SOURCE_DIR}/src/feather)
set(FBS_OUTPUT_FILES "${OUTPUT_DIR}/metadata_generated.h")

set(FBS_SRC metadata.fbs)
get_filename_component(ABS_FBS_SRC ${FBS_SRC} ABSOLUTE)

add_custom_command(
  OUTPUT ${FBS_OUTPUT_FILES}
  COMMAND ${FLATBUFFERS_COMPILER} -c -o ${OUTPUT_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/${FBS_SRC}
  DEPENDS ${ABS_FBA_SRC}
  COMMENT "Running flatc compiler on ${FBS_SRC}"
  VERBATIM
)

add_custom_target(metadata_fbs DEPENDS ${FBS_OUTPUT_FILES})
add_dependencies(feather metadata_fbs)

# installation
install(FILES
  api.h
  buffer.h
  common.h
  io.h
  metadata.h
  reader.h
  status.h
  types.h
  writer.h
  DESTINATION include/feather)

install(TARGETS feather
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib)
