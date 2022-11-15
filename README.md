# Home Assistant 

<img src="https://www.home-assistant.io/images/home-assistant-logo.svg" width=30% height=30%> 
# esphome </br>
Hardware </br>
   Gravity: MEMS Gas Sensor (CO, Alcohol, NO2 & NH3) - I2C - MiCS-4514<br/>
   https://www.dfrobot.com/product-2417.html
<img src="https://dfimg.dfrobot.com/store/data/SEN0377/SEN0377.jpg" width=50% height=50%>

espHome YAML 
```
esphome:
  name: esp32-test
  platform: ESP32
  board: firebeetle32
  
  
  includes:
    - my_custom_sensor.h
    
  libraries:
    - Wire
    - "https://github.com/makertut/esphome-air-sensors/tree/main/DFRobot_MICS"
  
# Enable logging
logger:

# Enable Home Assistant API
api:

ota:
  password: "c5ae2eaebedcf9fe5a73e0b405cb8241"

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Esp32-Test Fallback Hotspot"
    password: "kr8ry0mV9xzD"

captive_portal:

# Example configuration entry
sensor:
- platform: custom
  lambda: |-
    auto my_sensor = new MyCustomSensor();
    App.register_component(my_sensor);
    return {my_sensor->Carbon_Monoxide_sensor, 
            my_sensor->Methane_sensor, 
            my_sensor->Ethanol_sensor, 
            my_sensor->Hydrogen_sensor, 
            my_sensor->Ammonia_sensor, 
            my_sensor->Nitrogen_Dioxide_sensor
           };

  sensors:
  - name: "Carbon Monoxide"    
    unit_of_measurement: PPM
    accuracy_decimals: 0
  - name: "Methane"    
    unit_of_measurement: PPM
    accuracy_decimals: 0
  - name: "Ethanol"    
    unit_of_measurement: PPM
    accuracy_decimals: 0
  - name: "Hydrogen"    
    unit_of_measurement: PPM
    accuracy_decimals: 0
  - name: "Ammonia"    
    unit_of_measurement: PPM
    accuracy_decimals: 0  
  - name: "Nitrogen Dioxide"    
    unit_of_measurement: PPM
    accuracy_decimals: 2  
```
my_custom_sensor.h
```
#include "esphome.h"
#include "DFRobot_MICS.h"

#define CALIBRATION_TIME   3 

class MyCustomSensor : public PollingComponent, public Sensor {
 public:
  // constructor
  DFRobot_MICS_I2C mics;
  
  Sensor *Carbon_Monoxide_sensor = new Sensor(); // CO
  Sensor *Methane_sensor = new Sensor();         // CH4
  Sensor *Ethanol_sensor = new Sensor();         // C2H5OH 
  Sensor *Hydrogen_sensor = new Sensor();        // H2
  Sensor *Ammonia_sensor = new Sensor();         // NH3
  Sensor *Nitrogen_Dioxide_sensor = new Sensor();       // NO2
  
  MyCustomSensor() : PollingComponent(15000)  {}

  float get_setup_priority() const override { return esphome::setup_priority::AFTER_WIFI; }

  void setup() override {
    
    // This will be called by App.setup()
    while(!mics.begin()){
      ESP_LOGD("custom","NO Deivces !");
      delay(1000);
    } 
    
    
    ESP_LOGD("custom","Device connected successfully !");
    
    uint8_t mode = mics.getPowerState();
    if(mode == SLEEP_MODE){
       mics.wakeUpMode();
       ESP_LOGD("custom","wake up sensor success!");
    }else{
       ESP_LOGD("custom","The sensor is wake up mode");
    }

    while(!mics.warmUpTime(CALIBRATION_TIME)){
      ESP_LOGD("custom","Please wait until the warm-up time is over!");
      delay(1000);
    }
    
  }
  void update() override {
    // This will be called every "update_interval" milliseconds.
    float gasdata = mics.getGasData(CO);
    
    Carbon_Monoxide_sensor->publish_state(gasdata);
    
    gasdata = mics.getGasData(CH4);
    Methane_sensor->publish_state(gasdata);
    
    gasdata = mics.getGasData(C2H5OH);
    Ethanol_sensor->publish_state(gasdata);
    
    gasdata = mics.getGasData(H2);
    Hydrogen_sensor->publish_state(gasdata);
    
    gasdata = mics.getGasData(NH3);
    Ammonia_sensor->publish_state(gasdata);
    
    gasdata = mics.getGasData(NO2);
    Nitrogen_Dioxide_sensor->publish_state(gasdata);
  }
};
```
