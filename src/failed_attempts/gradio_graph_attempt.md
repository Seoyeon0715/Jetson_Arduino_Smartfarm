

# Gradio 기반 센서 데이터 시각화

`src/failed_attempts/gradio_graph_attempt.md`에서는 기존 **센서 데이터 조회 함수**에 **그래프 생성 기능**이 추가되었습니다.  
**측정 센서별 함수**  `moisture_sensor_info`, `light_sensor_info`, `dht_sensor_info`에 대한 설명은 기존의 **chatbot_ui.md**를 참고하세요.
 

---

## **1. 설치 및 실행**

**필요한 라이브러리 설치**  
```bash
pip install gradio pandas matplotlib openai
```

**스크립트 실행**  
```bash
python script_name.py
```

---

## **2. 새롭게 추가된 함수**

### **graph_from_file**

CSV 파일에서 데이터를 읽고 **샘플링 후 그래프**를 생성하는 함수입니다.

**코드**:
```python
import pandas as pd
import matplotlib.pyplot as plt

def graph_from_file(file_path, x_column, y_columns, sample_step=27):
    try:
        # CSV 파일 읽기
        data = pd.read_csv(file_path)
        
        # 데이터 샘플링: 첫 번째 행 이후 sample_step만큼 건너뛰기
        sampled_data = data.iloc[::sample_step, :].reset_index(drop=True)

        # 숫자에서 %와 C 기호 제거 후 변환
        for column in y_columns:
            if '%' in sampled_data[column].iloc[0]:
                sampled_data[column] = sampled_data[column].str.rstrip('%').astype(int)
            elif 'C' in sampled_data[column].iloc[0]:
                sampled_data[column] = sampled_data[column].str.rstrip('C').astype(int)

        # 그래프 생성
        plt.figure(figsize=(10, 6))
        for column in y_columns:
            plt.plot(sampled_data[x_column], sampled_data[column], label=column)
        
        plt.xlabel(x_column)
        plt.ylabel("Values")
        plt.title("Sampled Sensor Data Visualization")
        plt.xticks(rotation=45)
        plt.legend()
        plt.tight_layout()

        # 그래프 저장
        graph_path = "sensor_data_graph.png"
        plt.savefig(graph_path)
        plt.close()

        return {"graph_path": graph_path, "message": "Graph created successfully with sampled data."}
    except Exception as e:
        return {"error": str(e)}
```

---

## **3. 업데이트된 sensor_functions**

**`graph_from_file`** 함수가 추가된 `sensor_functions`는 다음과 같습니다.  

```python
sensor_functions = [
    # 기존의 moisture_sensor_info, light_sensor_info, dht_sensor_info는 chatbot_ui.md 참고
    {
        "type": "function",
        "function": {
            "name": "graph_from_file",
            "description": "Generates a graph using data from a CSV file with specified x and y columns.",
            "parameters": {
                "type": "object",
                "properties": {
                    "file_path": {
                        "type": "string",
                        "description": "The path to the CSV file containing the sensor data."
                    },
                    "x_column": {
                        "type": "string",
                        "description": "The column name to use as the x-axis (e.g., 'Timestamp')."
                    },
                    "y_columns": {
                        "type": "array",
                        "items": {"type": "string"},
                        "description": "A list of column names to use as the y-axis (e.g., 'Soil Moisture (%)')."
                    },
                    "sample_step": {
                        "type": "integer",
                        "description": "The step size for sampling data points to reduce graph density. Default is 27.",
                        "default": 27
                    }
                },
                "required": ["file_path", "x_column", "y_columns"]
            }
        }
    }
]
```

---

## **4. Gradio UI**

`graph_from_file` 함수를 포함한 사용자 입력 및 OpenAI 연동 Gradio UI는 다음과 같습니다.

```python
import gradio as gr
import json
from openai import OpenAI

# 사용자 메시지 처리 함수
def ask_openai(llm_model, messages, user_message, functions):
    client = OpenAI()
    proc_messages = messages.copy()

    if user_message:
        proc_messages.append({"role": "user", "content": user_message})

    response = client.chat.completions.create(
        model=llm_model, messages=proc_messages, tools=functions, tool_choice="auto"
    )
    response_message = response.choices[0].message
    tool_calls = response_message.tool_calls

    if tool_calls:
        available_functions = {
            "graph_from_file": graph_from_file  # 추가된 함수
        }

        proc_messages.append(response_message)

        for tool_call in tool_calls:
            function_name = tool_call.function.name
            function_to_call = available_functions.get(function_name)
            if function_to_call:
                try:
                    function_args = json.loads(tool_call.function.arguments)
                    function_response = function_to_call(**function_args)
                    proc_messages.append({
                        "tool_call_id": tool_call.id,
                        "role": "tool",
                        "name": function_name,
                        "content": json.dumps(function_response),
                    })
                except Exception as e:
                    proc_messages.append({
                        "tool_call_id": tool_call.id,
                        "role": "tool",
                        "name": function_name,
                        "content": json.dumps({"error": str(e)}),
                    })

        second_response = client.chat.completions.create(
            model=llm_model, messages=proc_messages
        )
        assistant_message = second_response.choices[0].message.content
    else:
        assistant_message = response_message.content

    proc_messages.append({"role": "assistant", "content": assistant_message})
    return proc_messages, assistant_message

# Gradio 인터페이스
messages = []

def process(user_message, chat_history):
    proc_messages, ai_message = ask_openai("gpt-4o", messages, user_message, functions=sensor_functions)
    chat_history.append((user_message, ai_message))
    return "", chat_history

with gr.Blocks() as demo:
    gr.Markdown("# 센서 데이터 조회 및 그래프 생성")
    chatbot = gr.Chatbot(label="센서 데이터 조회 및 시각화")
    user_textbox = gr.Textbox(label="입력")
    user_textbox.submit(process, [user_textbox, chatbot], [user_textbox, chatbot])

demo.launch(share=True, debug=True)
```

---

## **5. 실행 예시**

### **입력 예시**
```
"CSV 파일에서 그래프를 생성해줘. x축은 Timestamp, y축은 Soil Moisture (%)와 Temperature (C)로 해줘."
```

### **출력 결과**
- 생성된 그래프 이미지: `sensor_data_graph.png`  
- x축: `Timestamp`  
- y축: `Soil Moisture (%)`, `Temperature (C)`

---

