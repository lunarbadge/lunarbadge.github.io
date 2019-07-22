# Lunar Lander Badge

The software was written using Arduino, with the help of some esp32 native libraries where needed. The esp32 libraries are based on FreeRTOS.

## Hardware

The hardware schematic is here:(link)

It is similar to an Adafruit Feather, used in early prototyping. The additions are a TFT LCD screen, five buttons, seven LEDs and the SAO port for expansion. The esp32 communicates with the screen using SPI. The board incorporates a charging circuit for a LIPO battery (provided with the badge).

The code uses BLE (Bluetooth Low Energy) from the esp32, but not WiFi (though it's not blocked).

The hardware pinouts are defined in hardware.h. Four of the defined pins are not currently in use but are connected to the SAO port. The I2C connections (SDA / SCL) used for the SAO do not have additional pull-up resistors on the board, so may work best with SAOs that include their own resistors. Since the current software doesn't use I2C it is available for use with SAOs that support serial. There are two additional GPIO pins connected to the SAO.

## Software
The software is mostly written in C++. We pre-compute values where possible to avoid complex calculations.

### Building and Loading the code

Download and install Arduino. You will also need the esp32 packages. The BLE libraries on the esp32 use a considerable amount of memory, so the code _will not_ load with the standard Adafruit Feather board definition. You need to edit boards.txt, and repeat it when you upgrade Arduino or the esp32 libraries. The changes below add a new partition scheme to a standard Adafruit Feather. The partition disables OTA (over the air update) which is not used by this code.

    ~/Library/Arduino15/packages/esp32/hardware/esp32/1.0.2/boards.txt

    $ diff boards.txt orig_boards.txt
    1406,1412d1405
    < featheresp32.menu.PartitionScheme.default=Default 4MB with spiffs (1.2MB APP/1.5MB SPIFFS)
    < featheresp32.menu.PartitionScheme.default.build.partitions=default
    < featheresp32.menu.PartitionScheme.huge_app=Huge APP (3MB No OTA/1MB SPIFFS)
    < featheresp32.menu.PartitionScheme.huge_app.build.partitions=huge_app
    < featheresp32.menu.PartitionScheme.huge_app.upload.maximum_size=3145728

When you build using the Arduino IDE, select the "Adafruit ESP32 Feather" as the board, and "Huge APP (3MB No OTA/1MB SPIFFS)" as the Partition Scheme (if you build using other tools, hopefully you know what you are doing).

### Software Components

#### lunarLander.ino
The main Arduino code, consisting of setup() and loop() functions. Setup creates all the apps and initializes everything. The main event loop doesn't use a delay(); instead it first checks for button presses and processes them, then if it's been a while since the last action, it will call the currently-active app. There is too much app-specific code here, but that's the general idea. It uses the bitmap stored in landerImage. It also displays the credits.

#### App
Defines the interface that should be implemented by any apps or games running on the badge. SettingsApp, LunarLanderGame, FriendsApp and CommanderGame all implement App.

#### BLELander
The code that interacts with bluetooth. If you want to listen for results from scanning for friends, implement BleFriendObserver.

#### Idler
The Idler keeps the lights on (literally) while the badge is doing other things. It runs as a separate task, pinned to CPU0. Arduino code usually runs on CPU1.

#### LunarLanderGame
The reason we are here. This is a clean-room re-implementation of the Lunar Lander game. Enjoy! It uses LunarLanderConstants.h for the moonscape and the pre-calculated Lunar Lander.

#### Settings
The settings screen allows you to change the name on the badge, turn bluetooth on or off, and adjust the brightness (and colors, if unlocked).

#### CommanderGame
Where would the Lunar Lander be without the command module?  Don't overlook this one.

It uses a greyscale image of the moon, greyMoon.cpp.

#### FriendsApp
The friends app is where you can connect with other badges and share your score.

