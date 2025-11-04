---
title: IP camera 실시간 영상 yolo 사용 객체 탐지(NVDEC)
description: rtsp 주소 형식의 실시간 영상을 yolo 모델로 객체 탐지하고 웹페이지에서 볼 수 있게 만든 프로젝트 입니다. 추가적으로 NVDEC 기능을 사용해서 CPU 부담을 줄이는 작업도 했습니다.
date: 2025-11-04 16:23:00 +0900
categories: [NVIDIA]
tags: [nvidia, jetson, nvdec, yolo, ipcamera]     # TAG names should always be lowercase
---

# 카메라 정보

- XNO-8020R
• 1/1.8형 6메가픽셀 CMOS 채용
• 최대 30fps@5메가픽셀 (2560x1920)
• 화각(H): 97.5° / 3.7mm 고정 초점렌즈
• H.265, H.264, MJPEG 코덱 지원
• WiseStreamII 지원 (대역폭 최대 99% 절감)
• 최저조도 0Lux (IR LED On), 야간 가시거리 30m
• Day & Night (ICR), WDR (120dB) 역광보정
• 움직임, 초점 흐림, 탬퍼링 감지, 안개 보정
• IP67, IP66,방진/방수 규격, IK10 충격 대응 규격 획득
• 메모리 카드 2개 슬롯 (최대 512GB 지원)

<img width="422" height="107" alt="image" src="https://github.com/user-attachments/assets/b72e9c71-8f73-432a-a70b-3d98410a0fcc" />


# Gradio로 실시간 영상 확인 가능한 웹 만들기
- GPT의 도움을 받아서 작성한 코드입니다.

```
import os
import csv
import cv2
import datetime
import gradio as gr
from ultralytics import YOLO

# 모델 로드
MODEL_PATH = "yolo11n.pt"
model = YOLO(MODEL_PATH)

# IP camera rtsp 주소
STREAM_URL = "rtsp://username:password@ipadress:port/path"

# ------------------------------
# 글로벌 변수 (최근 프레임 & 탐지 결과 저장)
# ------------------------------
last_frame = None
last_detections = []

# ROI 영역 (x1, y1, x2, y2)
roi_coords = (200, 100, 500, 400)

LOW_RES = (300, 300)

# ------------------------------
# YOLO 탐지 함수
# ------------------------------
def detect_and_draw(frame):
    global last_frame, last_detections, roi_coords

    x1, y1, x2, y2 = roi_coords
    roi = frame[y1:y2, x1:x2]

    if roi.size == 0:
        return frame

    # ROI 추론
    roi_resized = cv2.resize(roi, LOW_RES)
    results = model(roi_resized, verbose=False)

    scale_x = (x2 - x1) / LOW_RES[0]
    scale_y = (y2 - y1) / LOW_RES[1]
    offset_x, offset_y = x1, y1

    detections = []
    for detection in results[0].boxes.data:
        bx1, by1, bx2, by2, conf, cls = detection
        bx1 = int(bx1 * scale_x + offset_x)
        by1 = int(by1 * scale_y + offset_y)
        bx2 = int(bx2 * scale_x + offset_x)
        by2 = int(by2 * scale_y + offset_y)
        label = results[0].names[int(cls)]
        detections.append([label, float(conf), bx1, by1, bx2, by2])

        # 트럭이면 빨간색, 아니면 초록색
        if label == "truck":
            color = (0, 0, 255)  # Red
            # 콘솔 출력
            print(f"Detected {label} at (x1={bx1}, y1={by1}, x2={bx2}, y2={by2})")
        else:
            color = (0, 255, 0)  # Green

        # 박스 + 텍스트 시각화
        cv2.rectangle(frame, (bx1, by1), (bx2, by2), color, 2)
        cv2.putText(frame, f"{label} {conf:.2f}", (bx1, by1 - 10),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.9, color, 2)

    # ROI 영역 시각화 (파란색 테두리)
    cv2.rectangle(frame, (x1, y1), (x2, y2), (255, 0, 0), 2)


    last_frame = frame.copy()
    last_detections = detections
    return frame
# ------------------------------
# 스트리밍 제너레이터
# ------------------------------
def process_stream():
    cap = cv2.VideoCapture(STREAM_URL)
    if not cap.isOpened():
        print("Error: Could not open stream.")
        yield None
        return

    frame_count = 0
    while cap.isOpened():
        ret, frame = cap.read()
        if not ret:
            break
        frame_count += 2
        # 탐지 프레임 주기를 설정해서 영상의 지연을 줄일 수 있습니다.
        if frame_count % 30 == 0:
            result = detect_and_draw(frame)
            yield cv2.cvtColor(result, cv2.COLOR_BGR2RGB)
    cap.release()

# ------------------------------
# CSV Flagging Callback
# ------------------------------
class CSVFlaggingCallback(gr.FlaggingCallback):
    def setup(self, components, flagging_dir):
        self.flag_dir = flagging_dir
        self.flag_file = os.path.join(flagging_dir, "dataset.csv")
        os.makedirs(flagging_dir, exist_ok=True)
        if not os.path.exists(self.flag_file):
            with open(self.flag_file, "w", newline="") as f:
                writer = csv.writer(f)
                writer.writerow(["timestamp", "image_path", "label", "confidence", "x1", "y1", "x2", "y2"])

    def flag(self, data, flag_option=None, flag_index=None, username=None):
        global last_frame, last_detections
        if last_frame is None or not last_detections:
            return "No detections to save."

        # 이미지 저장
        ts = datetime.datetime.now().strftime("%Y%m%d_%H%M%S")
        img_path = os.path.join(self.flag_dir, f"frame_{ts}.jpg")
        cv2.imwrite(img_path, last_frame)

        # CSV 저장
        with open(self.flag_file, "a", newline="") as f:
            writer = csv.writer(f)
            for det in last_detections:
                label, conf, x1, y1, x2, y2 = det
                writer.writerow([ts, img_path, label, conf, x1, y1, x2, y2])

        return f"✅ Saved {len(last_detections)} detections to dataset.csv"

# ------------------------------
# Gradio Blocks UI
# ------------------------------
with gr.Blocks(
    css="""
        #stream_window {width: 95vw; height: 95vh; object-fit: fill; border-width: 0;}
        footer{display:none !important}
    """,
    theme=gr.themes.Soft()
) as demo:
    img = gr.Image(
        label=None,
        sources=[],
        show_label=False,
        elem_id="stream_window",
        show_share_button=False,
    )
    with gr.Row():
        btn = gr.Button("Flag", elem_id="flag_button", variant="primary", size="sm")

    demo.load(fn=process_stream, inputs=None, outputs=img)

    callback = CSVFlaggingCallback()
    callback.setup([img], "flagged_data_points")

    # 버튼 클릭 → dataset.csv 저장
    btn.click(lambda *args: callback.flag(list(args)), [img], None, preprocess=False)

# ------------------------------
# 실행
# ------------------------------
if __name__ == "__main__":
    demo.launch(share=True)
```

gradio로 빠르게 테스트 했습니다.

하지만 IP camera의 영상을 다수의 사람이 웹에 접속해서 봐야하기 때문에, 
(gradio는 1명까지만 private하게 접속할 수 있고 2명 이상부터는 공개해야합니다.)

gradio를 사용하지 않고 Fast api를 써서 코드를 다시 작성했습니다.

# NVDEC 사용방법
- jtop으로 현황 파악을 했습니다.

<img width="484" height="350" alt="image" src="https://github.com/user-attachments/assets/6a8d0238-9b09-4318-8b8e-00236fd1caf4" />


비디오 디코딩을 수행하여 CPU의 컴퓨팅 집약적인 작업을 오프로드하는 엔비디아 그래픽 카드 기능인 NVDEC를 사용하지 않고 있는게 보였고, 

이 기능을 사용하기 위해서 install_opencv4.10.0_Jetpack6.1.sh를 설치했습니다.

[orin nano에서 NVDEC 사용법을 묻는 reddit 글](https://www.reddit.com/r/JetsonNano/comments/1njimpz/opencv_with_cuda_and_gstreamer/)을 보고 해당 글에 있는 답변을 따라서 [install_opencv4.10.0_Jetpack6.1.sh](https://github.com/AastaNV/JEP/blob/master/script/install_opencv4.10.0_Jetpack6.1.sh)를 수행했습니다.
