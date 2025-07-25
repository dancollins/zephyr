# SPDX-License-Identifier: Apache-2.0

zephyr_include_directories(${ZEPHYR_BASE}/drivers)
zephyr_sources(
  soc.c
  )

zephyr_include_directories(.)

set(SOC_LINKER_SCRIPT ${ZEPHYR_BASE}/include/zephyr/arch/arm/cortex_m/scripts/linker.ld CACHE INTERNAL "")

if(${CMAKE_HOST_SYSTEM_NAME} STREQUAL Windows)
  set(SIGNING_TOOL STM32_SigningTool_CLI.exe)
else()
  set(SIGNING_TOOL STM32_SigningTool_CLI)
endif()

find_file(SIGNING_TOOL_FIND ${SIGNING_TOOL})
if(SIGNING_TOOL_FIND STREQUAL SIGNING_TOOL_FIND-NOTFOUND)
  message(WARNING "
  Signing Image tool (${SIGNING_TOOL}) is not available.
  Signed image will not be generated.
  You won't be able to run application on the board.
  Refer to board documentation for more information")
else()
  set_property(GLOBAL APPEND PROPERTY extra_post_build_commands
    COMMAND ${SIGNING_TOOL}
    -in ${PROJECT_BINARY_DIR}/${CONFIG_KERNEL_BIN_NAME}.bin
    -nk -t fsbl -hv 2.3 --silent
    -o ${PROJECT_BINARY_DIR}/${CONFIG_KERNEL_BIN_NAME}.signed.bin
    -dump ${PROJECT_BINARY_DIR}/${CONFIG_KERNEL_BIN_NAME}.signed.bin
    WORKING_DIRECTORY ${PROJECT_BINARY_DIR}
  )

  set_property(TARGET runners_yaml_props_target PROPERTY bin_file ${CONFIG_KERNEL_BIN_NAME}.signed.bin)
endif()
