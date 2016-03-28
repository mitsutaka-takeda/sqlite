cmake_minimum_required(VERSION 3.3)

project(sqlite C)

find_package(Git REQUIRED)

file(GENERATE 
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/generate_manifest_and_uuid.sh
  CONTENT
"#!/bin/sh\n\
${GIT_EXECUTABLE} -C ${CMAKE_CURRENT_SOURCE_DIR} log -1 --format=format:%ci%n | sed -e 's/ [-+].*$//;s/ /T/;s/^/D /' > ${CMAKE_CURRENT_SOURCE_DIR}/manifest\n\
${CMAKE_COMMAND} -E echo $(${GIT_EXECUTABLE} -C ${CMAKE_CURRENT_SOURCE_DIR} log -1 --format=format:%H) > ${CMAKE_CURRENT_SOURCE_DIR}/manifest.uuid\n\
"
)

add_custom_command(
  OUTPUT 
  ${CMAKE_CURRENT_SOURCE_DIR}/manifest
  ${CMAKE_CURRENT_SOURCE_DIR}/manifest.uuid
  ${CMAKE_CURRENT_BINARY_DIR}/sqlite3.c
  ${CMAKE_CURRENT_BINARY_DIR}/include/sqlite3.h
  COMMAND chmod 764 ${CMAKE_CURRENT_BINARY_DIR}/generate_manifest_and_uuid.sh && ${CMAKE_CURRENT_BINARY_DIR}/generate_manifest_and_uuid.sh
  COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/configure --with-pic
  COMMAND make .target_source
  COMMAND chmod 764 ${CMAKE_CURRENT_SOURCE_DIR}/tool/mksqlite3c.tcl && ${CMAKE_CURRENT_SOURCE_DIR}/tool/mksqlite3c.tcl
  COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_CURRENT_BINARY_DIR}/include
  COMMAND ${CMAKE_COMMAND} -E copy_if_different ${CMAKE_CURRENT_BINARY_DIR}/sqlite3.h ${CMAKE_CURRENT_BINARY_DIR}/include/
  )

add_library(
  sqlite3
  STATIC
  ${CMAKE_CURRENT_BINARY_DIR}/sqlite3.c
  ${CMAKE_CURRENT_BINARY_DIR}/include/sqlite3.h
  )

target_include_directories(
  sqlite3
  PUBLIC
  ${CMAKE_CURRENT_BINARY_DIR}/include
  )
