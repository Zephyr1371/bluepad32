# Bluepad32 firmware for Arduino

## Supported boards

It works on **any ESP32 module** where the [Arduino Core for ESP32][arduino-core] runs.
In other words, if you already have Arduino working on ESP32 module, you can have Bluepad32 running on it as well.

But there is catch:

* It only works on ESP-IDF v4.4 or newer
* Arduino IDE is not supported yet, but you can use [Visual Studio Code + ESP-IDF plugin][vscode-plugin] (highly recommended!)

Are you still interested ? Good, then you must follow these instructions:

[arduino-core]: https://github.com/espressif/arduino-esp32
[vscode-plugin]: https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/vscode-setup.html

## Creating a Bluepad32 Arduino projects

There are three ways to setup a Bluepad32 Arduino project:

* Option A: Clone the template repo
  * Recommended!
* Option B: Starting a project from scratch
  * Useful to fine-tune your project
* Option C: Create a "Arduino Core for ESP32 + Bluepad32" library
  * Not ready... in fact, not even started. It will take time to have it ready.

## Option A: Clone the template project

This is the **RECOMMENDED OPTION**.

```sh
git clone --recursive https://gitlab.com/ricardoquesada/esp-idf-arduino-bluepad32-template.git my_project
```

... and that's it.

To test it just do:

```sh
# To compile it do:
idf.py build

# To flash it do:
idf.py flash monitor
```

## Option B: Create your a project from scratch

Use this option if you want to understand how the "template" project (from Option A) was
created.

It is split in 4 parts:

* Create an empty project
* Install the needed components
* Update configuration
* Copy the "main Arduino" files to your project

### Create an empty ESP-IDF project

Create an ESP-IDF project from template:

```sh
# One simiple way to start a new ESP-IDF project, is to clone the "template" project
git clone --recursive https://github.com/espressif/esp-idf-template my_project
```

### Installing components

Include the following components in your project's `components` folder:

* arduino
* bluepad32
* bluepad32_arduino
* btstack

Arduino component:

```sh
cd ${MYPROJECT}/components/
git clone --recursive https://github.com/espressif/arduino-esp32.git arduino
```

Bluepad32 / Bluepad32_arduino component:

```sh
# Just copy bluepad32 component to your project's component folder
cp -r ${BLUEPAD32}/src/components/bluepad32 ${MYPROJECT}/components
cp -r ${BLUEPAD32}/src/components/bluepad32_arduino ${MYPROJECT}/components
```

BTStack component:

```sh
cd ${BLUEPAD32}/external/btstack/port/esp32
# This will install BTstack as a component inside Bluepad32 source code (recommended).
# Remove "IDF_PATH=${BLUEPAD32}/src/" if you want it installed in the ESP-IDF folder
IDF_PATH=${BLUEPAD32}/src/ ./integrate_btstack.py
```

### Update configuration

And then do:

1. Add `set(ENV{BLUEPAD32_ARDUINO} TRUE)` to your main `CMakelists.txt`
2. `idf.py menuconfig`
3. Select "Components config" -> "Bluepad32" -> "Target platform" -> "Arduino"

   ![bluepad32-arduino](https://lh3.googleusercontent.com/pw/AM-JKLXm9ZyIvTKiTUlFBCT9QSaduKrhGZTXrWdR7G7F6krTHjkHJhpeGTXek_MCV3ZcXHCA8wnhxFAdDvQ_MbbGVMQY2AD58DK3DyK-_Cxua7BKHbvp8zkjtkcr87czftE7ySiCCUEcb6uSuMr9KY96JjQe-g=-no)

4. And set these Arduino options:
   * "Autostart Arduino setup and loop on boot" must be OFF
   * "Core on which Arduino's setup() and loop() are running" must be "Core 1"
     * Same for the remaining "Core" options
   * "Loop thread stack size": depends on what you do. 8192 is a good default value.
   * "Include only specific Arduino libraries" must ON
     * "Enable BLE" must be OFF
     * "Enable BluetoothSerial" must be OFF

    ![sdk-config](https://lh3.googleusercontent.com/pw/AM-JKLUC4p0Yf5fwxsmzBTqmisp09ElowiFvD06VZfVFeTe6qZZ7pavXZ3sOZ1qKe5wWvwCrnhZrvgOerIgb4XJcrX_fGQETiL2QObmE1u8KFn8wtRoO-vrLSJCRbQVgkC8_pnbyUQM4onrK6GXaaEf-Fuf4iQ=-no)

5. Set these Bluetooth options:
   * "Component Config" -> "Bluetooth" -> "Bluetooth Controller"
     * "Bluetooth Controller Mode": Bluetooth Dual Mode
     * "BLE Max Connections": 3
     * "BR/EDR ACL Max Connections": 7
     * "BR/EDR Sync (SCO/eSCO) Max Connections": 3
   * "Component Config" -> "Bluetooth" -> "Bluetooth Host"
     * "Controller only"

     ![sdkconfig-bluetooth](https://lh3.googleusercontent.com/pw/AM-JKLVOfishwCTAmGZN2owF0TNiTNVOlCR0DZf7PqUZprM0ujp_iM1e-tYMqDbhZKSe5zvJD4K4PCZJ-SuqO4IGnamgQL79vanzfvpItspvztGlsl0t_FlEkDYmif6q0WgbS6XCH7qrS0iM5LtqNxDySAWJhg=-no)

     ![sdkconfig-bluetooth2](https://lh3.googleusercontent.com/pw/AM-JKLUqEgrT5sF48hKUkmMsP2-9QzV6-JgyYyKwBfZA7GxjwOtQrDqYXvRE3R5tL7SQsAqRurXCiFqHoPU3k9noCtB-k_ZzJ4F_vqKqb9HVJXpI0ZkR5nJv8SzJ959LEmjjX9QaUteHpoJvbdHsiU-0TPoF8w=-no)

6. Set these ESP32 options:
   * "Component Config" -> "ESP32-specific"
      * "Main XTAL frequency": Autodetect

     ![sdkconfig-esp32](https://lh3.googleusercontent.com/pw/AM-JKLVvcfEonqhFDIWH98KajzMGSADBgaNoCI2QjGHaVFLPeRRAQMcIlXFwRmhvDSmNo6kIX_TGtKRr3V6EerW4ngPEiWbBtJYQPSOe2fixKC-rb16m3hhAVirbH7VnVmFwE1EXvRZk3MnNj7Yu2ydFn9f5Gg=-no)

7. Set Serial flasher config:
   * "Serial flasher cofnig"
      * "Flash size": 4MB (or choose the right for your module)

### Copy Bluepad32-Arduino API files

And in your *main* project, you must include the files that are located in `${BLUEPAD32}/src/main/*`

```sh
cd my_project/main
cp -r ${BLUEPAD32}/src/main/* .
```

Further reading:

* [Arduino as a ESP-IDF component][esp-idf-component]

[esp-idf-component]: https://docs.espressif.com/projects/arduino-esp32/en/latest/esp-idf_component.html

## Option C: Create an "Arduino Core for ESP32 + Bluepad32" library

TODO.

This option should allow you to use Arduino IDE.

In the meantime read: [ESP-IDF Library Builder][lib-builder]

[lib-builder]: https://docs.espressif.com/projects/arduino-esp32/en/latest/lib_builder.html
