---

## 아두이노 센서의 데이터 수집하기 (`collect_sensor_data.md`)

이 스크립트는 **Python**과 **Pandas**를 사용해 아두이노 센서 데이터를 실시간으로 수집하고 **CSV 파일**로 저장하는 코드를 제공합니다.
이 코드는 **Jetson Nano**의 **가상환경**을 통해 **Jupyter Notebook**에서 실행된다. 

---

### **전체 코드**

```python
import serial
import time
import pandas as pd

# 직렬 통신 설정
port = "/dev/ttyUSB0"  # 아두이노의 직렬 포트
baudrate = 9600        # 아두이노와 동일한 보드레이트
arduino = serial.Serial(port, baudrate, timeout=1)

# 데이터 저장용 리스트
data = []

# 현재 날짜 초기화
current_date = time.strftime("%Y-%m-%d")

try:
    print("Collecting data from Arduino... Press Ctrl+C to stop.")
    while True:
        if arduino.in_waiting > 0:  # 데이터가 직렬 포트에 들어왔는지 확인
            line = arduino.readline().decode("utf-8").strip()  # 데이터 읽기 및 디코딩
            
            # 데이터 필터링: 센서 데이터만 처리
            if "Soil Moisture:" in line and "Light Intensity:" in line:
                timestamp = time.strftime("%Y-%m-%d %H:%M:%S")  # 현재 시간을 초 단위로 저장
                
                # 센서 데이터 파싱
                parts = line.split(", ")
                soil_moisture = parts[0].split(":")[1].strip()  # Soil Moisture 값
                light_intensity = parts[1].split(":")[1].strip()  # Light Intensity 값
                humidity = parts[2].split(":")[1].strip()  # Humidity 값
                temperature = parts[3].split(":")[1].strip()  # Temperature 값
                
                print(f"{timestamp}, Soil Moisture: {soil_moisture}, Light Intensity: {light_intensity}, Humidity: {humidity}, Temperature: {temperature}")
                
                # 데이터 저장
                data.append({
                    "Timestamp": timestamp,
                    "Soil Moisture (%)": soil_moisture,
                    "Light Intensity (%)": light_intensity,
                    "Humidity (%)": humidity,
                    "Temperature (C)": temperature
                })

                # 데이터프레임 생성
                df = pd.DataFrame(data)

                # 파일명 생성 (현재 날짜 기반)
                file_name = f"sensor_data_{current_date.replace('-', '')}.csv"

                # 데이터프레임을 CSV 파일로 저장
                df.to_csv(file_name, index=False, encoding="utf-8")
                print(f"Data saved to {file_name}")

except KeyboardInterrupt:
    print("Data collection stopped.")

finally:
    # 마지막으로 남은 데이터를 저장
    if data:
        df = pd.DataFrame(data)
        file_name = f"sensor_data_{current_date.replace('-', '')}.csv"
        df.to_csv(file_name, index=False, encoding="utf-8")
        print(f"Final data saved to {file_name}")

    # 포트 닫기
    arduino.close()
```

---

### **프로젝트 개요**

이 스크립트는 다음과 같은 역할을 합니다:

1. **아두이노 직렬 통신 설정**: 데이터를 읽어오기 위해 직렬 포트 연결  
2. **데이터 필터링 및 파싱**: 아두이노로부터 전송된 센서 데이터(토양 수분, 조도, 온습도)를 읽어 정확하게 분류  
3. **실시간 데이터 저장**: 수집된 데이터를 CSV 파일로 실시간 저장  
4. **종료 시 데이터 보존**: 프로그램 종료 시 남은 데이터를 안전하게 저장  

---

### **사용 방법**

1. 아두이노 코드(`mainarduino.md`)를 업로드하고 실행합니다.  
2. Python 스크립트를 실행합니다:
   ```bash
   python collect_sensor_data.py
   ```
3. 프로그램 실행 중 `Ctrl+C`를 눌러 종료할 수 있습니다.  
4. 수집된 데이터는 **`sensor_data_YYYYMMDD.csv`** 파일에 저장됩니다.

---

### **출력 예시 (터미널)**

```
2024-06-01 14:23:45, Soil Moisture: 45, Light Intensity: 78, Humidity: 60, Temperature: 24
Data saved to sensor_data_20240601.csv
2024-06-01 14:23:47, Soil Moisture: 43, Light Intensity: 80, Humidity: 59, Temperature: 23
Data saved to sensor_data_20240601.csv
```

---

### **파일 예시 (CSV)**

| Timestamp           | Soil Moisture (%) | Light Intensity (%) | Humidity (%) | Temperature (C) |
|---------------------|-------------------|---------------------|-------------|-----------------|
| 2024-06-01 14:23:45 | 45                | 78                  | 60          | 24              |
| 2024-06-01 14:23:47 | 43                | 80                  | 59          | 23              |

---

### **주의사항**

1. **직렬 포트 설정**: `/dev/ttyUSB0` (Linux/Mac) 또는 `COM3` (Windows)로 시스템에 맞게 수정해야 합니다.  
2. **보드레이트**: 아두이노 코드와 Python 스크립트의 보드레이트(`9600`)가 일치해야 합니다.  
3. **종료 시 데이터 저장**: `Ctrl+C`로 종료하면 마지막까지 수집된 데이터가 저장됩니다.

---

