# Github Actions CI Arduino Init Script 

The purpose of this repo is to execute an automated compilation test of all examples in the WisBlock repository. It uses Github Actions.    
This repository is based on [ci-arduino](https://github.com/adafruit/ci-arduino) which was created by Adafruit. 

All required steps are defined in the `main.yml` action.  
The result of the tests are available in the [Actions overview](https://github.com/RAKWireless/WisBlock/actions).

## To be aware of

### This Github action is adjusted to test WisBlock examples with the RAK nRF Arduino BSP.

### When new examples are added, required libraries have to be added to the `extra libraries` section of `main.yml`  

### Many libraries have to be installed by cloning them and cannot be installed with the Arduino CLI library manager.

#### Example for library installations:
```yml  
    - name: extra libraries
      run: |
        /home/runner/work/WisBlock/WisBlock/bin/arduino-cli lib install SX126x-Arduino
        /home/runner/work/WisBlock/WisBlock/bin/arduino-cli lib install Arduino_LPS22HB
        /home/runner/work/WisBlock/WisBlock/bin/arduino-cli lib install ArduinoRS485
        /home/runner/work/WisBlock/WisBlock/bin/arduino-cli lib install ArduinoModbus
        git clone --quiet https://github.com/sparkfun/SparkFun_SHTC3_Arduino_Library
        mv SparkFun_SHTC3_Arduino_Library /home/runner/Arduino/libraries/SparkFun_SHTC3_Arduino_Library
        git clone --quiet https://github.com/closedcube/ClosedCube_OPT3001_Arduino
        mv ClosedCube_OPT3001_Arduino /home/runner/Arduino/libraries/ClosedCube_OPT3001_Arduino
        git clone --quiet https://github.com/sparkfun/SparkFun_LIS3DH_Arduino_Library
        mv SparkFun_LIS3DH_Arduino_Library /home/runner/Arduino/libraries/SparkFun_LIS3DH_Arduino_Library
        git clone --quiet https://github.com/adafruit/Adafruit_Sensor
        mv Adafruit_Sensor /home/runner/Arduino/libraries/Adafruit_Sensor
        git clone --quiet https://github.com/adafruit/Adafruit_BME680
        mv Adafruit_BME680 /home/runner/Arduino/libraries/Adafruit_BME680
        git clone --quiet https://github.com/BoschSensortec/BSEC-Arduino-library
        mv BSEC-Arduino-library /home/runner/Arduino/libraries/BSEC-Arduino-library
        git clone --quiet https://github.com/adafruit/Adafruit_TCS34725
        mv Adafruit_TCS34725 /home/runner/Arduino/libraries/Adafruit_TCS34725
        git clone --quiet https://github.com/sparkfun/SparkFun_TMP102_Arduino_Library
        mv SparkFun_TMP102_Arduino_Library /home/runner/Arduino/libraries/SparkFun_TMP102_Arduino_Library
        git clone --quiet https://github.com/sparkfun/SparkFun_SGP30_Arduino_Library
        mv SparkFun_SGP30_Arduino_Library /home/runner/Arduino/libraries/SparkFun_SGP30_Arduino_Library
        /home/runner/work/WisBlock/WisBlock/bin/arduino-cli lib install U8g2
        /home/runner/work/WisBlock/WisBlock/bin/arduino-cli lib install nRF52_OLED
```

### If additional platforms are added (ESP32 e.g.), the required BSP's and platforms have to be added in the `install.sh` and `build_platform.py` scripts.  
  
#### Lines to change for additional platform in `build_platform.py`:  
```sh  
ALL_PLATFORMS={
...
    "rak4631" : "raknrf:nrf52:WisCoreRAK4631Board:softdevice=s140v6,debug=l0",
...
}

BSP_URLS = "https://raw.githubusercontent.com/RAKWireless/RAKwireless-Arduino-BSP-Index/main/package_rakwireless_index.json,https://adafruit.github.io/arduino-board-index/package_adafruit_index.json,http://arduino.esp8266.com/stable/package_esp8266com_index.json,https://dl.espressif.com/dl/package_esp32_index.json,https://sandeepmistry.github.io/arduino-nRF5/package_nRF5_boards_index.json"


```

#### Lines to change for additional platform in `install.sh`  
```sh
...
# associative array for other platforms that can be called explicitly in .travis.yml configs
# this will be eval'd in the functions below because arrays can't be exported
...
export NRF5X_PLATFORMS='declare -A nrf5x_platforms=( [rak4631]="raknrf:nrf52:WisCoreRAK4631Board:softdevice=s140v6,debug=l0")'
...
# install the zero, esp8266, and adafruit board packages
echo -n "ADD PACKAGE INDEX: "
DEPENDENCY_OUTPUT=$(arduino --pref "boardsmanager.additional.urls=https://raw.githubusercontent.com/RAKWireless/RAKwireless-Arduino-BSP-Index/main/package_rakwireless_index.json,http://arduino.esp8266.com/stable/package_esp8266com_index.json,https://dl.espressif.com/dl/package_esp32_index.json" --save-prefs 2>&1)

...
INSTALL_NRF52=$([[ $INSTALL_PLATFORMS == *"raknrf"* || -z "$INSTALL_PLATFORMS" ]] && echo 1 || echo 0)
...
if [[ $INSTALL_NRF52 == 1 ]]; then
  echo -n "RAK nRF WisBlock: "
  pip3 install --user setuptools
  pip3 install --user adafruit-nrfutil
  pip3 install --user pyserial
  sudo pip3 install setuptools
  sudo pip3 install adafruit-nrfutil
  sudo pip3 install pyserial
  DEPENDENCY_OUTPUT=$(arduino --install-boards raknrf:nrf52 2>&1)
  if [ $? -ne 0 ]; then echo -e "\xe2\x9c\x96 OR CACHED"; else echo -e """$GREEN""\xe2\x9c\x93"; fi
fi
...
```

### If an example should not be tested, adding an empty file named `.rak4631.test.skip` will exclude the example from being tested against the RAK4631 board.

## Example output of a test:
```log
test platforms                             6m 39s
Run python3 ci/build_platform.py rak4631
build dir: /home/runner/work/WisBlock/WisBlock

########################################
INSTALLING ARDUINO BOARDS
########################################
arduino-cli core update-index --additional-urls https://raw.githubusercontent.com/RAKWireless/RAKwireless-Arduino-BSP-Index/main/package_rakwireless_index.json,https://adafruit.github.io/arduino-board-index/package_adafruit_index.json,http://arduino.esp8266.com/stable/package_esp8266com_index.json,https://dl.espressif.com/dl/package_esp32_index.json,https://sandeepmistry.github.io/arduino-nRF5/package_nRF5_boards_index.json > /dev/null


No library dep or properties found!
Libraries installed:  ['/home/runner/Arduino/libraries/SX126x-Arduino', '/home/runner/Arduino/libraries/ClosedCube_OPT3001_Arduino', '/home/runner/Arduino/libraries/Adafruit_Sensor', '/home/runner/Arduino/libraries/SparkFun_TMP102_Arduino_Library', '/home/runner/Arduino/libraries/Adafruit_Test_Library', '/home/runner/Arduino/libraries/ArduinoRS485', '/home/runner/Arduino/libraries/SparkFun_SGP30_Arduino_Library', '/home/runner/Arduino/libraries/nRF52_OLED', '/home/runner/Arduino/libraries/Adafruit_TCS34725', '/home/runner/Arduino/libraries/ArduinoModbus', '/home/runner/Arduino/libraries/BSEC-Arduino-library', '/home/runner/Arduino/libraries/U8g2', '/home/runner/Arduino/libraries/SparkFun_LIS3DH_Arduino_Library', '/home/runner/Arduino/libraries/SparkFun_SHTC3_Arduino_Library', '/home/runner/Arduino/libraries/Adafruit_BME680', '/home/runner/Arduino/libraries/Arduino_LPS22HB']
################################################################################
SWITCHING TO raknrf:nrf52:WisCoreRAK4631Board:softdevice=s140v6,debug=l0
Installing raknrf:nrf52 ✓
################################################################################
	ble_ota_dfu.ino ✓
	ble_proximity_sensing.ino ✓
	ble_uart.ino ✓
	BG77_Unvarnished_Transmission.ino ✓
	Cellular_Ping.ino ✓
	LoRaP2P_RX.ino ✓
	LoRaP2P_TX.ino ✓
	LoRaWAN_ABP.ino ✓
	LoRaWAN_OTAA.ino ✓
	RAK4631-DeepSleep-LoRaWan.ino ✓
	AT_Command_Test.ino ✓
	connect_ap.ino ✓
	RAK1901_Temperature_Humidity_SHTC3.ino ✓
	RAK1902_Pressure_LPS22HB.ino ✓
	RAK1903_Optical_OPT3001.ino ✓
	RAK1904_Accelerate_LIS3DH.ino ✓
	bme680_basic.ino ✓
	bme680_bsec.ino skipping
	RAK1910_GPS_UBLOX7.ino ✓
	RAK1920_Grove_Color_TCS3472.ino ✓
	RAK1920_Grove_PIR_AS312.ino ✓
	RAK1920_MikroBUS_Temperature_TMP102.ino ✓
	RAK1920_QWIIC_AirQuality_SGP30.ino ✓
	RAK1921_Jumping_Ball_SSD1306.ino ✓
	RAK1921_Moving_Logo_SSD1306.ino ✓
	RAK1921_OLED_SSD1306.ino ✓
	Read_Battery_Level.ino ✓
	ble.ino ✓
	display.ino ✓
	RAK5801_4-20mA.ino ✓
	Receiver.ino ✓
	Sender.ino ✓
	RAK5811_0-5V.ino ✓
	ble_environment_node.ino ✓
	ble_gateway.ino ✓
	Environment_Monitoring.ino ✓
	GPS_Tracker.ino ✓
	Hydraulic_Pressure_Monitoring.ino ✓
	Inteligence_Agriculture.ino ✓
	PAR_Monitoring.ino ✓
	Soil_Conductivity_Monitoring.ino ✓
	Soil_pH_Monitoring.ino ✓
	Water_Level_Monitoring.ino ✓
	Weather_Monitoring.ino ✓
	Wind_Speed_Monitoring.ino ✓
```
