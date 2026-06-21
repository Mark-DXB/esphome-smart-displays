# Waveshare 5-Inch RGB Display (ESPHome)

This configuration drives a custom ESPHome LVGL Media Player on the Waveshare 5-Inch ESP32-S3 RGB display.

## Key Community Findings & Solutions

This board is notoriously tricky to get running perfectly in ESPHome. We solved several major challenges during development that are worth sharing:

### 1. I2C IO Expander (CH422G)
The backlight and the touchscreen reset pin are NOT directly connected to the ESP32-S3. They are routed through a CH422G I2C IO expander. 
We successfully mapped the `mipi_rgb` display component and the `gt911` touchscreen component to this expander, ensuring stable initialization.

### 2. GDMA Starvation & Display Offset
When using `power_save_mode: LIGHT` to save battery, the GDMA channel driving the MIPI RGB display stutters during WiFi sleep cycles, causing a permanent horizontal and vertical pixel shift.
* **Attempted Fix:** We tested injecting `CONFIG_LCD_RGB_RESTART_IN_VSYNC=y` into the ESP-IDF compiler.
* **Result:** While it prevented the permanent offset, it introduced unacceptable screen tearing and jitter on this specific panel. 
* **Final Solution:** We removed the VSYNC restart flag. To deal with the occasional soft-boot offset, we added a hidden 80x80 pixel "Secret Button" in the top-right corner of the screen that forces a soft reboot when tapped.

### 3. Sleep Shield
To prevent accidental button presses when waking the screen up from sleep, we implemented a transparent `sleep_shield` object that sits on top of all interactive elements. When tapped, it resets the sleep timer and hides itself, letting the next tap interact with the actual UI.

### 4. Dynamic LVGL Speaker Grouping
We built a dynamic multi-room media player interface entirely inside LVGL without relying on HA dashboards. It queries Home Assistant for `group_members` and conditionally updates UI switches to show active speaker groups.
