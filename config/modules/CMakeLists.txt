# Add the module directory to the Zephyr module search path
list(APPEND ZEPHYR_EXTRA_MODULES ${CMAKE_CURRENT_SOURCE_DIR}/zmk-driver-pim447)

# Make sure this CMake variable is visible outside this directory
set(ZEPHYR_EXTRA_MODULES ${ZEPHYR_EXTRA_MODULES} PARENT_SCOPE)