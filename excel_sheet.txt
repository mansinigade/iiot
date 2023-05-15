#include <Arduino.h>

#include <Adafruit_Sensor.h>
#include <DHT.h>
#include <DHT_U.h>

#include <stdint.h>
// https://www.microsoft.com/en-us/download/confirmation.aspx?id=56976
constexpr uint8_t sensorPin{15};
constexpr uint8_t channelsInExcel{2};
constexpr uint16_t serialInterval{50};

DHT_Unified dht(sensorPin, DHT11);
uint32_t serialPreviousTime;
int8_t *sensorData[channelsInExcel];

void setup()
{
  Serial.begin(9600);
  dht.begin();
}

void loop()
{
  // Reading data from sensors
  sensor_t temperature;
  dht.temperature().getSensor(&temperature);
  sensor_t humidity;
  dht.humidity().getSensor(&humidity);

  uint8_t serialData[64]{};
  while (Serial.available() != 0) {
    Serial.readBytesUntil('\n', &serialData[0], 64);
  }

  // Tokenizing and adding readings to an array
  uint8_t *token = strtok(&serialData[0], ",");
  uint8_t index{};
  while (token != nullptr) {
    sensorData[index] = token;
    token = strtok(nullptr, ",");
    ++index;
  }

  // Print data to spreadsheet
  if ((millis() - serialPreviousTime) > serialInterval) {
    serialPreviousTime = millis();

    sensors_event_t event;
    dht.temperature().getEvent(&event);
    Serial.print(event.temperature);
    Serial.print(',');
    dht.humidity().getEvent(&event);
    Serial.print(event.relative_humidity);
    Serial.print(',');

    Serial.println();
  }
}
