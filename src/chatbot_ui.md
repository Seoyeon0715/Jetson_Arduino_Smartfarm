---

# **센서 데이터 조회 및 Gradio UI 구현 프로젝트**

이 코드는 **아두이노 센서 데이터**를 실시간 또는 파일에서 처리하고, **OpenAI API**와 **Gradio UI**를 사용하여 사용자 인터페이스를 제공합니다.

---

## **1. 프로젝트 설정**

### **필수 라이브러리 설치**

다음 라이브러리를 설치해야 합니다:

```bash
pip install openai
pip install gradio
pip install pyserial
```

### **환경 변수 설정**

`OpenAI API` 키를 설정해야 합니다.  

**코드**:
```python
import os
os.environ['OPENAI_API_KEY'] = 'please enter your key'
```

---

## **2. 전체 코드**

### **기본 설정 및 라이브러리 Import**

```python
# 필수 라이브러리 설치
!pip install openai
!pip install gradio
!pip install pyserial

# Import
import gradio as gr
import pandas as pd
import serial
import time
import json
from openai import OpenAI
import os

os.environ['OPENAI_API_KEY'] = 'please enter your key'

OpenAI.api_key = os.getenv("OPENAI_API_KEY")
import time
import serial
```

---

### **토양 수분 센서 함수**

```python
def moisture_sensor_info(mode='real-time', file_path=None, port='/dev/ttyUSB0', baudrate=9600):
    """
    토양 수분 센서 데이터를 처리하고 반환하는 함수.

    Parameters:
    - mode (str): 'real-time' 또는 'file'. 실시간 또는 파일 기반 데이터 처리.
    - file_path (str): 'file' 모드에서 사용할 CSV 파일 경로.
    - port (str): 'real-time' 모드에서 사용할 시리얼 포트.
    - baudrate (int): 시리얼 통신 속도.

    Returns:
    - dict: Soil Moisture 값 또는 에러 메시지를 포함한 딕셔너리.
    """
    if mode == 'real-time':
        try:
            arduino = serial.Serial(port, baudrate, timeout=1)
            time.sleep(2)
            arduino.reset_input_buffer()
            attempts = 5
            while attempts > 0:
                if arduino.in_waiting > 0:
                    data = arduino.readline().decode('utf-8', errors='ignore').strip()
                    if "Soil Moisture:" in data:
                        try:
                            soil_moisture = int(data.split(", ")[0].split(":")[1].strip())
                            arduino.close()
                            return {
                                'timestamp': time.strftime("%Y-%m-%d %H:%M:%S"),
                                'Soil Moisture (%)': soil_moisture
                            }
                        except (IndexError, ValueError):
                            pass
                time.sleep(1)
                attempts -= 1
            arduino.close()
            return {'error': 'No valid data received from sensor after multiple attempts.'}
        except Exception as e:
            return {'error': str(e)}
    elif mode == 'file':
        try:
            if not file_path:
                return {'error': 'File path is required for "file" mode.'}
            df = pd.read_csv(file_path)
            if df.empty:
                return {'error': 'The CSV file is empty.'}
            latest_entry = df.iloc[-1]
            return {
                'timestamp': latest_entry['Timestamp'],
                'Soil Moisture (%)': latest_entry['Soil Moisture (%)']
            }
        except Exception as e:
            return {'error': str(e)}
    else:
        return {'error': 'Invalid mode. Use "real-time" or "file".'}
```

---

### **조도 센서 함수**

```python
def light_sensor_info(mode='real-time', file_path=None, port='/dev/ttyUSB0', baudrate=9600):
    """
    조도 센서 데이터를 처리하고 반환하는 함수.
    """
    if mode == 'real-time':
        try:
            arduino = serial.Serial(port, baudrate, timeout=1)
            time.sleep(2)
            arduino.reset_input_buffer()
            attempts = 5
            while attempts > 0:
                if arduino.in_waiting > 0:
                    data = arduino.readline().decode('utf-8', errors='ignore').strip()
                    if "Light Intensity:" in data:
                        try:
                            light_intensity = int(data.split(", ")[1].split(":")[1].strip())
                            arduino.close()
                            return {
                                'timestamp': time.strftime("%Y-%m-%d %H:%M:%S"),
                                'Light Intensity (%)': light_intensity
                            }
                        except (IndexError, ValueError):
                            pass
                time.sleep(1)
                attempts -= 1
            arduino.close()
            return {'error': 'No valid data received from sensor after multiple attempts.'}
        except Exception as e:
            return {'error': str(e)}
    elif mode == 'file':
        try:
            if not file_path:
                return {'error': 'File path is required for "file" mode.'}
            df = pd.read_csv(file_path)
            if df.empty:
                return {'error': 'The CSV file is empty.'}
            latest_entry = df.iloc[-1]
            return {
                'timestamp': latest_entry['Timestamp'],
                'Light Intensity (%)': latest_entry['Light Intensity (%)']
            }
        except Exception as e:
            return {'error': str(e)}
    else:
        return {'error': 'Invalid mode. Use "real-time" or "file".'}
```

---

### **DHT11 센서 함수**

```python
def dht_sensor_info(mode='real-time', file_path=None, port='/dev/ttyUSB0', baudrate=9600):
    """
    온습도 센서 데이터를 처리하고 반환하는 함수.
    """
    if mode == 'real-time':
        try:
            arduino = serial.Serial(port, baudrate, timeout=1)
            time.sleep(2)
            arduino.reset_input_buffer()
            attempts = 5
            while attempts > 0:
                if arduino.in_waiting > 0:
                    data = arduino.readline().decode('utf-8', errors='ignore').strip()
                    if "Humidity:" in data and "Temperature:" in data:
                        try:
                            humidity = int(data.split(", ")[2].split(":")[1].strip())
                            temperature = int(data.split(", ")[3].split(":")[1].strip())
                            arduino.close()
                            return {
                                'timestamp': time.strftime("%Y-%m-%d %H:%M:%S"),
                                'Humidity (%)': humidity,
                                'Temperature (C)': temperature
                            }
                        except (IndexError, ValueError):
                            pass
                time.sleep(1)
                attempts -= 1
            arduino.close()
            return {'error': 'No valid data received from sensor after multiple attempts.'}
        except Exception as e:
            return {'error': str(e)}
    elif mode == 'file':
        try:
            if not file_path:
                return {'error': 'File path is required for "file" mode.'}
            df = pd.read_csv(file_path)
            if df.empty:
                return {'error': 'The CSV file is empty.'}
            latest_entry = df.iloc[-1]
            return {
                'timestamp': latest_entry['Timestamp'],
                'Humidity (%)': latest_entry['Humidity (%)'],
                'Temperature (C)': latest_entry['Temperature (C)']
            }
        except Exception as e:
            return {'error': str(e)}
    else:
        return {'error': 'Invalid mode. Use "real-time" or "file".'}
```
---

## **센서 함수 정의 (`sensor_functions`)**

`sensor_functions`는 OpenAI API가 호출할 수 있도록 정의된 JSON 구조입니다. 각 함수는 특정 센서의 데이터를 실시간으로 가져오거나 저장된 파일에서 조회할 수 있도록 구성되어 있습니다.

---

### **코드**

```python
sensor_functions = [
    {
        "type": "function",
        "function": {
            "name": "moisture_sensor_info",
            "description": "Retrieves soil moisture data either in real-time from a sensor or from a file.",
            "parameters": {
                "type": "object",
                "properties": {
                    "mode": {
                        "type": "string",
                        "description": "'real-time' for live data from the sensor or 'file' for data from a CSV file.",
                        "enum": ["real-time", "file"]
                    },
                    "file_path": {
                        "type": "string",
                        "description": "The path to the CSV file containing soil moisture data (required if mode is 'file').",
                        "nullable": True
                    },
                    "port": {
                        "type": "string",
                        "description": "The serial port to which the sensor is connected (required if mode is 'real-time'). Default is '/dev/ttyUSB0'.",
                        "nullable": True
                    },
                    "baudrate": {
                        "type": "integer",
                        "description": "The baud rate for serial communication with the sensor. Default is 9600.",
                        "nullable": True
                    }
                },
                "required": ["mode"]
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "light_sensor_info",
            "description": "Retrieves light intensity data either in real-time from a sensor or from a file.",
            "parameters": {
                "type": "object",
                "properties": {
                    "mode": {
                        "type": "string",
                        "description": "'real-time' for live data from the sensor or 'file' for data from a CSV file.",
                        "enum": ["real-time", "file"]
                    },
                    "file_path": {
                        "type": "string",
                        "description": "The path to the CSV file containing light intensity data (required if mode is 'file').",
                        "nullable": True
                    },
                    "port": {
                        "type": "string",
                        "description": "The serial port to which the sensor is connected (required if mode is 'real-time'). Default is '/dev/ttyUSB0'.",
                        "nullable": True
                    },
                    "baudrate": {
                        "type": "integer",
                        "description": "The baud rate for serial communication with the sensor. Default is 9600.",
                        "nullable": True
                    }
                },
                "required": ["mode"]
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "dht_sensor_info",
            "description": "Retrieves temperature and humidity data either in real-time from a sensor or from a file.",
            "parameters": {
                "type": "object",
                "properties": {
                    "mode": {
                        "type": "string",
                        "description": "'real-time' for live data from the sensor or 'file' for data from a CSV file.",
                        "enum": ["real-time", "file"]
                    },
                    "file_path": {
                        "type": "string",
                        "description": "The path to the CSV file containing temperature and humidity data (required if mode is 'file').",
                        "nullable": True
                    },
                    "port": {
                        "type": "string",
                        "description": "The serial port to which the sensor is connected (required if mode is 'real-time'). Default is '/dev/ttyUSB0'.",
                        "nullable": True
                    },
                    "baudrate": {
                        "type": "integer",
                        "description": "The baud rate for serial communication with the sensor. Default is 9600.",
                        "nullable": True
                    }
                },
                "required": ["mode"]
            }
        }
    }
]
```

---

### **구조 및 설명**

각 함수는 다음과 같은 속성으로 구성됩니다:

1. **`name`**: 함수의 이름 (예: `moisture_sensor_info`)  
2. **`description`**: 함수의 역할과 목적에 대한 설명  
3. **`parameters`**:  
   - **`mode`**: `real-time` 또는 `file` 모드를 선택  
   - **`file_path`**: CSV 파일 경로 (파일 모드에서 필수)  
   - **`port`**: 센서 연결 시리얼 포트 (`real-time` 모드에서 사용)  
   - **`baudrate`**: 시리얼 통신 속도 (기본값: 9600)  

---

### **Gradio UI 구현**

```python
def ask_openai(llm_model, messages, user_message, functions):
    client = OpenAI()
    proc_messages = messages
    if user_message:
        proc_messages.append({"role": "user", "content": user_message})

    response = client.chat.completions.create(
        model=llm_model, messages=proc_messages, tools=functions, tool_choice="auto"
    )
    response_message = response.choices[0].message
    assistant_message = response_message.content
    return proc_messages, assistant_message


messages = []

def process(user_message, chat_history):
    proc_messages, ai_message = ask_openai("gpt-4o-mini", messages, user_message, functions=sensor_functions)
    chat_history.append((user_message, ai_message))
    return "", chat_history

with gr.Blocks() as demo:
    chatbot = gr.Chatbot(label="센서 데이터 조회")
    user_textbox = gr.Textbox(label="입력")
    user_textbox.submit(process, [user_textbox, chatbot], [user_textbox, chatbot])

demo.launch(share=True, debug=True)
```

---



