project(libyara C)

set(MODULE_SOURCES modules/tests.c
                   modules/pe.c
                   modules/pe_utils.c
                   modules/elf.c
                   modules/math.c)

if(DOTNET_MODULE)
    list(APPEND MODULE_SOURCES modules/dotnet.c)
endif()

if(CUCKOO_MODULE)
    list(APPEND MODULE_SOURCES modules/cuckoo.c)
endif()

if(MAGIC_MODULE)
    list(APPEND MODULE_SOURCES modules/magic.c)
endif()

if(HASH_MODULE)
    list(APPEND MODULE_SOURCES modules/hash.c)
endif()

list(APPEND LIBYARA_INCLUDES include/yara.h
                             include/yara/ahocorasick.h
                             include/yara/atoms.h
                             include/yara/limits.h
                             include/yara/re.h
                             include/yara/arena.h
                             include/yara/sizedstr.h
                             include/yara/types.h
                             include/yara/hash.h
                             include/yara/exec.h
                             include/yara/scan.h
                             include/yara/rules.h
                             include/yara/error.h
                             include/yara/utils.h
                             include/yara/filemap.h
                             include/yara/compiler.h
                             include/yara/modules.h
                             include/yara/object.h
                             include/yara/strutils.h
                             include/yara/stream.h
                             include/yara/threading.h
                             include/yara/integers.h
                             include/yara/libyara.h)

set(BISON_FLAGS "-y")

file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/generated_files)

FLEX_TARGET(YARA_LEXER lexer.l ${CMAKE_CURRENT_BINARY_DIR}/generated_files/lexer.c)
BISON_TARGET(YARA_GRAMMAR grammar.y ${CMAKE_CURRENT_BINARY_DIR}/generated_files/grammar.c
             DEFINES_FILE ${CMAKE_CURRENT_BINARY_DIR}/generated_files/grammar.h
             COMPILE_FLAGS ${BISON_FLAGS})
ADD_FLEX_BISON_DEPENDENCY(YARA_LEXER YARA_GRAMMAR)

FLEX_TARGET(YARA_HEX_LEXER hex_lexer.l ${CMAKE_CURRENT_BINARY_DIR}/generated_files/hex_lexer.c)
BISON_TARGET(YARA_HEX_GRAMMAR hex_grammar.y ${CMAKE_CURRENT_BINARY_DIR}/generated_files/hex_grammar.c
             DEFINES_FILE ${CMAKE_CURRENT_BINARY_DIR}/generated_files/hex_grammar.h
             COMPILE_FLAGS ${BISON_FLAGS})
ADD_FLEX_BISON_DEPENDENCY(YARA_HEX_LEXER YARA_HEX_GRAMMAR)

FLEX_TARGET(YARA_RE_LEXER re_lexer.l ${CMAKE_CURRENT_BINARY_DIR}/generated_files/re_lexer.c)
BISON_TARGET(YARA_RE_GRAMMAR re_grammar.y ${CMAKE_CURRENT_BINARY_DIR}/generated_files/re_grammar.c
             DEFINES_FILE ${CMAKE_CURRENT_BINARY_DIR}/generated_files/re_grammar.h
             COMPILE_FLAGS ${BISON_FLAGS})
ADD_FLEX_BISON_DEPENDENCY(YARA_RE_LEXER YARA_RE_GRAMMAR)

include_directories(${CMAKE_CURRENT_BINARY_DIR}/generated_files)

set(LIBYARA_SOURCES ${MODULE_SOURCES}
                    grammar.y
                    ahocorasick.c
                    arena.c
                    atoms.c
                    compiler.c
                    exec.c
                    exefiles.c
                    filemap.c
                    hash.c
                    hex_grammar.y
                    hex_lexer.l
                    lexer.l
                    libyara.c
                    mem.c
                    modules.c
                    object.c
                    parser.c
                    proc.c
                    re.c
                    re_grammar.y
                    re_lexer.l
                    rules.c
                    scan.c
                    sizedstr.c
                    strutils.c
                    stream.c
                    threading.c
                    ${BISON_YARA_GRAMMAR_OUTPUTS}
                    ${BISON_YARA_HEX_GRAMMAR_OUTPUTS}
                    ${BISON_YARA_RE_GRAMMAR_OUTPUTS}
                    ${FLEX_YARA_LEXER_OUTPUTS}
                    ${FLEX_YARA_HEX_LEXER_OUTPUTS}
                    ${FLEX_YARA_RE_LEXER_OUTPUTS})

include_directories(${CMAKE_CURRENT_SOURCE_DIR}
                    ${CMAKE_CURRENT_SOURCE_DIR}/include)

if(LIBYARA_SHARED)
    add_library(libyara SHARED ${LIBYARA_INCLUDES} ${LIBYARA_SOURCES})
else()
    target_compile_definitions(libyara PRIVATE YARA_STATICLIB=1)
    add_library(libyara STATIC ${LIBYARA_INCLUDES} ${LIBYARA_SOURCES})
endif()
set_target_properties(libyara PROPERTIES PREFIX "")
message(STATUS "libyara libraries: ${LIBYARA_LIBRARIES}")
target_link_libraries(libyara ${LIBYARA_LIBRARIES})
install(TARGETS libyara DESTINATION lib)
install(DIRECTORY include/ DESTINATION include FILES_MATCHING PATTERN "*.h")
