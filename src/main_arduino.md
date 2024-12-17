---

## Main_arduino cod 설명

이 코드는 **토양 수분 센서**, **조도 센서**, 그리고 **DHT11 온습도 센서**를 사용해 환경 데이터를 수집하고 시리얼 모니터를 통해 출력하는 코드입니다. 
센서 데이터를 안정적으로 읽고 노이즈를 줄이기 위한 평균화 처리도 포함되어 있습니다.

---

### 사용된 센서 및 핀 정의

1. **토양 수분 센서**  
   - **핀 연결**: 아날로그 핀 `A1`  
   - **역할**: 토양의 수분 값을 읽어 백분율(0~100%)로 변환합니다.  
   - **보정 값**:  
      - 건조한 상태: `520`  
      - 습한 상태: `260`  

2. **조도 센서**  
   - **핀 연결**: 아날로그 핀 `A0`  
   - **역할**: 빛의 세기를 측정하고, 노이즈 제거를 위해 샘플 평균과 극단값 제외 처리를 합니다.  

3. **DHT11 온습도 센서**  
   - **핀 연결**: 디지털 핀 `D2`  
   - **역할**: 온도와 습도 값을 읽어 출력합니다.  
   - **라이브러리**: `SimpleDHT.h`  

---

### 라이브러리 설치

DHT11 센서의 데이터를 읽기 위해 **`SimpleDHT.h` 라이브러리**가 필요합니다.  
Arduino IDE에서 아래 방법으로 설치하세요:

1. Arduino IDE를 열고 **Sketch -> Include Library -> Manage Libraries**를 선택합니다.  
2. 검색창에 **"SimpleDHT"**를 입력하고 설치합니다.  

> **참고**: SimpleDHT 라이브러리는 DHT11 센서와 호환됩니다.

---

### 주요 기능

1. **토양 수분 데이터 수집**  
   - 센서 값 평균화 처리 (10회 측정 후 평균)  
   - 백분율 변환 및 값 보정  

2. **조도 센서 데이터 수집**  
   - 10회 측정 후 극단값 제거  
   - 안정적인 평균값 계산  

3. **온습도 데이터 수집**  
   - DHT11 센서를 통해 온도 및 습도 값 출력  

4. **시리얼 출력**  
   - 모든 센서 값을 시리얼 모니터에 2초 간격으로 출력합니다.  

---

### 하드웨어 연결

| 센서             | VCC    | GND    | 데이터 핀     |
|------------------|--------|--------|---------------|
| 토양 수분 센서   | VCC    | GND    | A1 (Analog)   |
| 조도 센서        | VCC    | GND    | A0 (Analog)   |
| DHT11 센서       | VCC    | GND    | D2 (Digital)  |

---

### 아두이노 센서 측정 코드

```bash
#include <SimpleDHT.h>

// Pin definitions
const int soilMoisturePin = A1; // Soil moisture sensor connected to analog pin
const int lightSensorPin = A0;  // Light sensor connected to analog pin
const int dhtPin = 2;           // DHT11 connected to digital pin

// Constants for soil moisture sensor
const int dryValue = 520;  // Calibrated dry soil value
const int wetValue = 260;  // Calibrated wet soil value

// DHT11 sensor initialization
SimpleDHT11 dht11;

// Function to get average sensor value for soil moisture
int getAverageSensorValue(int pin, int samples) {
  long total = 0;
  for (int i = 0; i < samples; i++) {
    total += analogRead(pin);
    delay(10); // Small delay between readings
  }
  return total / samples;
}

// Function to get stable and averaged light sensor reading
int getStableLightReading() {
  const int NUM_SAMPLES = 10;
  int samples[NUM_SAMPLES];

  // Collect samples
  for (int i = 0; i < NUM_SAMPLES; i++) {
    samples[i] = analogRead(lightSensorPin);
    delay(10);
  }

  // Sort samples
  for (int i = 0; i < NUM_SAMPLES - 1; i++) {
    for (int j = i + 1; j < NUM_SAMPLES; j++) {
      if (samples[i] > samples[j]) {
        int temp = samples[i];
        samples[i] = samples[j];
        samples[j] = temp;
      }
    }
  }

  // Remove extreme values and calculate the average
  long sum = 0;
  for (int i = 2; i < NUM_SAMPLES - 2; i++) {
    sum += samples[i];
  }
  return sum / (NUM_SAMPLES - 4);
}

void setup() {
  Serial.begin(9600);
  Serial.println("Starting Combined Sensor Test!");
}

void loop() {
  // 1. Soil Moisture Sensor
  int soilValue = getAverageSensorValue(soilMoisturePin, 10);
  int soilMoisturePercent = map(soilValue, dryValue, wetValue, 0, 100);
  soilMoisturePercent = constrain(soilMoisturePercent, 0, 100);

  // 2. Light Sensor
  int lightValue = getStableLightReading();
  int lightPercent = map(lightValue, 0, 1023, 0, 100);

  // 3. DHT11 Sensor
  byte temperature = 0;
  byte humidity = 0;
  int dhtErr = dht11.read(dhtPin, &temperature, &humidity, NULL);

  // Output all sensor data in one line
  Serial.print("Soil Moisture:");
  Serial.print(soilMoisturePercent);
  Serial.print(", Light Intensity:");
  Serial.print(lightPercent);
  Serial.print(", ");

  if (dhtErr == SimpleDHTErrSuccess) {
    Serial.print("Humidity:");
    Serial.print(humidity);
    Serial.print(", Temperature:");
    Serial.println(temperature);
  } else {
    Serial.println("Humidity:Error, Temperature:Error");
  }

  delay(2000); // Wait for 2 seconds before the next reading
}
```
---
### 실행 예시

```
Soil Moisture: 45, Light Intensity: 78, Humidity: 60, Temperature: 24
Soil Moisture: 43, Light Intensity: 80, Humidity: 59, Temperature: 23
...
