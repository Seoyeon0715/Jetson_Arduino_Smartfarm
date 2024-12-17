# 프로젝트 이름

## 📖 프로젝트 개요
간단한 프로젝트의 목적과 내용을 설명합니다. 무엇을 하고자 했는지 명확히 전달하세요.

## 🚀 주요 기능
- **기능 1**: 간단한 설명
- **기능 2**: 간단한 설명
- **기능 3**: 간단한 설명

## 🛠️ 기술 스택
- **프로그래밍 언어**: Python, C++
- **플랫폼/프레임워크**: Jetson Nano, Arduino, TensorFlow 등

## 📂 **프로젝트 구조**
```plaintext
📁 스마트 환경 데이터 모니터링 시스템/
├── README.md             # 프로젝트 설명
├── src/                  # 소스 코드
│   ├── main.py           # 메인 프로그램 실행 코드 
│   ├── sensor_data.py    # 센서 데이터 수집 코드 **= sensor data upload.ipynb**
│   ├── discord_alert.py  # Discord 알림 시스템 코드 **박서연이 보내줄거임**
│   └── chatbot_ui.py     # Gradio UI 기반 챗봇 코드 **= gradioui_update(2) - 그라디오 파일 **
├── test/                 # 테스트 코드
├── docs/                 # 추가 문서화 
├── data/                 # CSV 데이터 저장 폴더 
│   └── sensor_data.csv   # 수집된 센서 데이터 **=sensor data 20241213.csv**
└── requirements.txt      # 의존성 리스트
