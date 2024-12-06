add_subdirectory(src)
file(GLOB TEST_FILES src/*.c src/drivers/*.c)
set(FRAMEWORK_TEST_TARGET_NAME ${MBEDTLS_FRAMEWORK_TARGET_PREFIX}framework_test)
add_library(${FRAMEWORK_TEST_TARGET_NAME} OBJECT ${TEST_FILES})
if(GEN_FILES)
    add_custom_command(
        OUTPUT
            ${CMAKE_CURRENT_BINARY_DIR}/src/test_keys.h
        COMMAND
            "${MBEDTLS_FRAMEWORK_PYTHON_EXECUTABLE}"
            "${CMAKE_CURRENT_SOURCE_DIR}/../scripts/generate_test_keys.py"
            "--output"
            "${CMAKE_CURRENT_BINARY_DIR}/src/test_keys.h"
        DEPENDS
            ${CMAKE_CURRENT_SOURCE_DIR}/../scripts/generate_test_keys.py
    )
    add_custom_target(test_keys_header
        DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/src/test_keys.h)
    add_custom_command(
        OUTPUT
            ${CMAKE_CURRENT_BINARY_DIR}/src/test_certs.h
        COMMAND
            "${MBEDTLS_FRAMEWORK_PYTHON_EXECUTABLE}"
            "${CMAKE_CURRENT_SOURCE_DIR}/../scripts/generate_test_cert_macros.py"
            "--output"
            "${CMAKE_CURRENT_BINARY_DIR}/src/test_certs.h"
        DEPENDS
            ${CMAKE_CURRENT_SOURCE_DIR}/../scripts/generate_test_cert_macros.py
    )
    add_custom_target(test_certs_header
        DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/src/test_certs.h)
    add_dependencies(${FRAMEWORK_TEST_TARGET_NAME} test_keys_header test_certs_header)
endif()
target_include_directories(${FRAMEWORK_TEST_TARGET_NAME}
    PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/include)
# Request C11, needed for memory poisoning tests
set_target_properties(${FRAMEWORK_TEST_TARGET_NAME} PROPERTIES C_STANDARD 11)
