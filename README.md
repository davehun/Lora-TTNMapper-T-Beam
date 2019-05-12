## Intro

This is a simple sketch demonstrating the capability of the [TTGO T-Beam](https://www.aliexpress.com/store/product/TTGO-T-Beam-ESP32-433-868-915Mhz-WiFi-wireless-Bluetooth-Module-ESP-32-GPS-NEO-6M/2090076_32875743018.html) as a [TTN Mapper](https://ttnmapper.org/) Node on [The Things Network](https://www.thethingsnetwork.org/) LoraWAN.

Derived from [sbiermann/Lora-TTNMapper-ESP32](https://github.com/sbiermann/Lora-TTNMapper-ESP32) and with some information/inspiration from [cyberman54/ESP32-Paxcounter](https://github.com/cyberman54/ESP32-Paxcounter) and [Edzelf/LoRa](https://github.com/Edzelf/LoRa).

## Software dependencies

[platfomIO](https://platformio.org/)

[TinyGPS++](http://arduiniana.org/libraries/tinygpsplus/)

[LMIC-Arduino](https://github.com/matthijskooijman/arduino-lmic)

## Instructions

You need to connect the [T-Beam](https://github.com/LilyGO/TTGO-T-Beam) DIO1 pin marked *Lora1* to the *pin 33* - So that the ESP32 can read that output from the Lora module.
Optionally you can also connect the *Lora2* output to *GPIO 32*, but this is not needed here.

Change config.h to match your settings then run

``` bash
pio run -t upload
```
This will fetch the required libs and install the code onto the board

On The Things Network side, the settings needed are available [here](https://www.thethingsnetwork.org/docs/applications/ttnmapper/).

Configure the Payload decoder with:
```javascript
function Decoder(bytes, port) {
    var decoded = {};

    decoded.latitude = ((bytes[0]<<16)>>>0) + ((bytes[1]<<8)>>>0) + bytes[2];
    decoded.latitude = (decoded.latitude / 16777215.0 * 180) - 90;

    decoded.longitude = ((bytes[3]<<16)>>>0) + ((bytes[4]<<8)>>>0) + bytes[5];
    decoded.longitude = (decoded.longitude / 16777215.0 * 360) - 180;

    var altValue = ((bytes[6]<<8)>>>0) + bytes[7];
    var sign = bytes[6] & (1 << 7);
    if(sign)
    {
        decoded.altitude = 0xFFFF0000 | altValue;
    }
    else
    {
        decoded.altitude = altValue;
    }

    decoded.hdop = bytes[8] / 10.0;

    return decoded;
}
```



## Todolist

* ~~Stop sending data to TTN until the GPS get a fix.~~ <== Done thanks to [@Roeland54](https://github.com/Roeland54)
* ~~Switch to [the maintained version](https://github.com/mcci-catena/arduino-lmic)~~ of the arduino-lmic library. <==Done
* Manage and document the different T-Beam revisions/versions.
* Switch to OTAA auth method for TTN and save the 'credentials' for reboot use.
* SPIFFS FS
* Save and reload the frame counter somewhere - GPS RTC data ? SPIFFS ? EEPROM ? - so I can check the "Frame Counter Checks" box as recommended on TTN.
* Also save the GPS 'status' so that on next boot it gets a fix faster.
* Reduce the power needed ! That thing is a power hog currently, we need to make it sleep most of the time as possible.
* Adapt the data send frequency based on current velocity : When not moving, an update per hour should be enough.
