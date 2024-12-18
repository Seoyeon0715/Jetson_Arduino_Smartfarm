---

# **Arduino 센서 데이터 Discord 알림 시스템**

이 프로젝트는 **Arduino 센서 데이터**를 읽고 특정 조건에 따라 **Discord 웹훅**으로 알림을 보내는 Python 스크립트입니다. **토양 수분 값**이 임계값 이하일 때 경고 메시지를 자동으로 전송합니다.

---

## **1. 프로젝트 개요**

### **기능**
1. Arduino와 시리얼 통신을 통해 센서 데이터를 읽습니다.  
2. **토양 수분 값**이 부족하면 Discord 웹훅에 알림 메시지를 전송합니다.  
3. 모든 센서 데이터를 10초마다 Discord에 전송합니다.

---

## **2. 전체 코드**

```python
import serial  # 시리얼 통신용
import requests  # HTTP 요청용
import time  # 시간 지연용

# 설정
WEBHOOK_URL = "please enter your webhook url"  # Discord 웹훅 URL 입력
SERIAL_PORT = "/dev/ttyUSB0"  # Arduino 시리얼 포트 확인 (ls /dev/tty* 로 확인 가능)
BAUD_RATE = 9600  # Arduino와 동일한 Baud Rate 설정

# Discord에 메시지 보내기
def send_to_discord(message):
    payload = {"content": message}  # Discord 메시지 포맷
    try:
        response = requests.post(WEBHOOK_URL, json=payload)
        if response.status_code == 204:
            print("Message successfully sent to Discord!")
        else:
            print(f"Failed to send: {response.status_code} - {response.text}")
    except Exception as e:
        print(f"An error occurred: {e}")

# Arduino 데이터 읽기
def read_sensor_data():
    try:
        # 시리얼 통신 시작
        with serial.Serial(SERIAL_PORT, BAUD_RATE, timeout=2) as ser:
            print("Connecting to Arduino...")
            time.sleep(2)  # 안정적인 연결을 위해 대기
            while True:
                if ser.in_waiting > 0:
                    # 데이터 읽기
                    data = ser.readline().decode('utf-8').strip()  # Arduino에서 데이터 읽기
                    print(f"센서 데이터 수신: {data}")

                    # 데이터 파싱 (예: "Soil Moisture:75, Light Intensity:28, Humidity:15, Temperature:24")
                    try:
                        values = {kv.split(":")[0].strip(): int(kv.split(":")[1].strip())
                                  for kv in data.split(",")}

                        # 토양 수분 데이터 조건 확인
                        soil_moisture = values.get("Soil Moisture", 0)  # 센서 데이터에서 영문 키를 사용
                        if soil_moisture < 30:
                            alert_message = f"⚠️ Soil moisture is low! Please water the plants! Current moisture level: {soil_moisture}"
                            print(alert_message)
                            send_to_discord(alert_message)

                        # 센서 전체 데이터 Discord에 전송
                        send_to_discord(f"Sensor data alert: {data}")

                    except Exception as parse_error:
                        print(f"Data parsing error: {parse_error}")

                    time.sleep(10)  # 10초마다 데이터 전송

    except Exception as e:
        print(f"Serial communication error: {e}")

# 메인 함수 실행
if __name__ == "__main__":
    read_sensor_data()
```

---

## **3. 설정 및 사용법**

### **환경 설정**
1. **Python 라이브러리 설치**  
   필수 라이브러리를 설치합니다:
   ```bash
   pip install pyserial requests
   ```

2. **Discord 웹훅 URL 설정**  
   Discord 채널에서 웹훅 URL을 생성하고 코드 내 `WEBHOOK_URL`에 입력합니다.

3. **Arduino 시리얼 포트 확인**  
   시스템 환경에 맞는 포트를 확인하고 `SERIAL_PORT`에 설정합니다:  
   - **Linux/Mac**: `/dev/ttyUSB0` 또는 `/dev/ttyACM0`  
   - **Windows**: `COM3`, `COM4` 등  

4. **Arduino Baud Rate**  
   Arduino 코드와 동일한 `BAUD_RATE`를 설정합니다.

---

### **실행**
```bash
python your_script_name.py
```

---

## **4. 주요 기능 설명**

1. **시리얼 통신**  
   Arduino 센서 데이터를 읽고 UTF-8로 디코딩합니다.

2. **데이터 파싱**  
   센서 데이터가 다음 형식일 때 파싱합니다:  
   ```
   "토양수분: 250, 조도: 500, 온도: 25"
   ```

3. **Discord 알림**  
   - **토양 수분 부족 조건**: `토양수분 < 30`이면 경고 메시지를 Discord로 전송합니다.  
   - **주기적 데이터 전송**: 모든 센서 데이터를 10초마다 Discord에 알립니다.

---

## **5. 예시 출력**

### **터미널 출력**
```
Arduino와 연결 중...
센서 데이터 수신: 토양수분: 25, 조도: 500, 온도: 26
⚠️ 토양 수분이 부족합니다! 물을 주세요! 현재 수분 값: 25
Discord로 메시지 전송 성공!
센서 데이터 알림: 토양수분: 25, 조도: 500, 온도: 26
```

### **Discord 메시지 예시**
```
⚠️ 토양 수분이 부족합니다! 물을 주세요! 현재 수분 값: 25
센서 데이터 알림: 토양수분: 25, 조도: 500, 온도: 26
```

---

## **6. 주의사항**
1. **Discord 웹훅 보안**: 웹훅 URL이 외부에 노출되지 않도록 주의하세요.  
2. **포트 설정**: `SERIAL_PORT`는 시스템에 따라 다를 수 있으므로 정확히 확인해야 합니다.  
3. **센서 값 조정**: 조건값(예: `soilMoisture < 30`)은 센서와 환경에 맞게 조정하세요.

---

