cmake_minimum_required(VERSION 2.8)

project(jq)

find_package(BISON)
find_package(FLEX)

set(CMAKE_BUILD_TYPE "Release")
set(CMAKE_C_FLAGS "-Wextra -Wall -Wno-missing-field-initializers -Wno-unused-parameter -std=gnu99 -ggdb -Wno-unused-function")

add_custom_target(gen_utf8_tables python ${CMAKE_SOURCE_DIR}/gen_utf8_tables.py > ${CMAKE_BINARY_DIR}/jv_utf8_tables.gen.h)

set(JQ_VERSION 1.1)

CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/version.cmake ${CMAKE_CURRENT_BINARY_DIR}/version.gen.h)

bison_target(parser parser.y ${CMAKE_CURRENT_BINARY_DIR}/parser.gen.c )
flex_target(lexer lexer.l ${CMAKE_CURRENT_BINARY_DIR}/lexer.gen.c COMPILE_FLAGS "--header-file=${CMAKE_BINARY_DIR}/lexer.gen.h")

include_directories(${CMAKE_BINARY_DIR})
include_directories(${CMAKE_CURRENT_SOURCE_DIR})

set(JQ_SRC
    opcode.c 
    bytecode.c 
    compile.c 
    execute.c 
    builtin.c 
    jv.c 
    jv_parse.c 
    jv_print.c 
    jv_dtoa.c 
    jv_unicode.c 
    jv_aux.c)


add_executable(jq ${JQ_SRC} main.c ${BISON_parser_OUTPUTS} ${FLEX_lexer_OUTPUTS})
add_dependencies(jq gen_utf8_tables gen_version)

add_executable(jq_test ${JQ_SRC} jq_test.c ${BISON_parser_OUTPUTS} ${FLEX_lexer_OUTPUTS})
install_programs(FILES jq)
