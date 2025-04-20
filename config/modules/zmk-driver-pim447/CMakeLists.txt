# Copyright (c) 2023-2025 The ZMK Contributors
# SPDX-License-Identifier: MIT

if(CONFIG_ZMK_TRACKBALL_PIM447)
  zephyr_include_directories(include)
  add_subdirectory(src/drivers/sensor/trackball_pim447)
endif()