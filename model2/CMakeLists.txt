cmake_minimum_required(VERSION 2.6)
project (model2)

set(MODEL_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)

set(SOURCES
  ${MODEL_SOURCE_DIR}/main.cpp
)

add_executable(model2 ${SOURCES})
target_link_libraries(model2 ${MLPACK_LIBRARIES})
