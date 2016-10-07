project(yara_tests)

include_directories(${CMAKE_CURRENT_SOURCE_DIR} ${YARA_INCLUDES})

add_executable(test_alignment test-alignment.c)
target_link_libraries(test_alignment libyara)

add_test(NAME test_alignment COMMAND $<TARGET_FILE:test_alignment>)

add_executable(test_rules test-rules.c util.c)
target_link_libraries(test_rules libyara)
add_test(NAME test_rules COMMAND $<TARGET_FILE:test_rules>
         WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/..)

add_executable(test_pe test-pe.c util.c)
target_link_libraries(test_pe libyara)
add_test(NAME test_pe COMMAND $<TARGET_FILE:test_pe>
         WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/..)
