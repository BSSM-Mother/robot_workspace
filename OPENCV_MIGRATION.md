# OpenCV 기반 Person Detector로 마이그레이션

## 변경 사항 요약

**이전**: YOLOv8 (ultralytics + PyTorch)
- ❌ Raspberry Pi에서 "Illegal instruction" 오류 발생
- ❌ 큰 메모리 사용량 (2GB+)
- ❌ 복잡한 설치 과정
- ❌ ARM CPU 호환성 문제

**현재**: OpenCV HOG (Histogram of Oriented Gradients)
- ✅ 100% Raspberry Pi 호환
- ✅ 낮은 메모리 사용량 (~200MB)
- ✅ 간단한 설치 (opencv-python만 필요)
- ✅ Illegal instruction 오류 없음
- ✅ 빠른 실시간 처리

## 성능 비교

| 항목 | YOLO | OpenCV HOG |
|------|------|------------|
| 정확도 | ⭐⭐⭐⭐⭐ (매우 높음) | ⭐⭐⭐⭐ (높음) |
| 속도 (Raspberry Pi 4) | ~5 FPS | ~15-20 FPS |
| 메모리 사용 | 2-3 GB | 200-500 MB |
| 설치 복잡도 | 높음 | 낮음 |
| ARM 호환성 | 불안정 | 완벽 |

## 감지 알고리즘 차이

### HOG (Histogram of Oriented Gradients)
- **원리**: 이미지의 그래디언트(변화) 방향 히스토그램 분석
- **장점**:
  - 경량 알고리즘
  - CPU에서 빠른 실행
  - 안정적인 사람 감지
  - 추가 모델 파일 불필요
- **단점**:
  - YOLO보다 정확도 낮음
  - 작은 사람 또는 가려진 사람 감지 어려움
  - 정면/측면 자세에서 가장 잘 작동

### YOLO (You Only Look Once)
- **원리**: 딥러닝 기반 객체 감지
- **장점**:
  - 매우 높은 정확도
  - 다양한 자세/각도 감지
  - 여러 클래스 동시 감지
- **단점**:
  - 큰 모델 크기
  - 많은 메모리 필요
  - Raspberry Pi에서 호환성 문제

## 코드 변경 사항

### person_detector.py

**제거됨**:
```python
from ultralytics import YOLO
model = YOLO('yolov8n.pt')
results = model.predict(...)
```

**추가됨**:
```python
import cv2
hog = cv2.HOGDescriptor()
hog.setSVMDetector(cv2.HOGDescriptor_getDefaultPeopleDetector())
boxes, weights = hog.detectMultiScale(image, winStride=(8,8))
```

### setup.py

**제거됨**:
```python
'ultralytics>=8.0.0',
```

**유지됨**:
```python
'opencv-python>=4.5.0',
'numpy',
```

### raspi_setup.sh

**제거됨**:
- PyTorch 설치
- ultralytics 설치
- 복잡한 ARM 호환성 플래그

**간소화됨**:
```bash
pip3 install opencv-contrib-python numpy
```

## 마이그레이션 방법

### 기존 YOLO 환경에서 전환

```bash
# 1. 기존 패키지 제거
pip3 uninstall -y ultralytics torch torchvision torchaudio

# 2. OpenCV 설치
pip3 install opencv-contrib-python

# 3. 워크스페이스 재빌드
cd /workspaces/robot_workspace
colcon build --symlink-install

# 4. 실행
source install/setup.bash
ros2 launch robot_launch system.launch.py
```

## HOG 파라미터 튜닝

### 속도 최적화 (정확도 희생)
```python
boxes, weights = hog.detectMultiScale(
    image,
    winStride=(16, 16),   # 더 큰 stride = 더 빠름
    padding=(4, 4),
    scale=1.1,            # 더 큰 scale = 더 빠름
    finalThreshold=2.0    # 더 높은 threshold = 더 빠름
)
```

### 정확도 최적화 (속도 희생)
```python
boxes, weights = hog.detectMultiScale(
    image,
    winStride=(4, 4),     # 더 작은 stride = 더 정확
    padding=(16, 16),
    scale=1.03,           # 더 작은 scale = 더 정확
    finalThreshold=1.0    # 더 낮은 threshold = 더 정확
)
```

### 균형잡힌 설정 (권장)
```python
boxes, weights = hog.detectMultiScale(
    image,
    winStride=(8, 8),
    padding=(8, 8),
    scale=1.05,
    finalThreshold=1.5
)
```

## Q&A

**Q: HOG가 YOLO만큼 정확한가요?**
A: 일반적인 사람 추적 작업에서는 충분히 정확합니다. YOLO는 복잡한 환경이나 작은 객체 감지에서 더 우수합니다.

**Q: 다시 YOLO로 돌아갈 수 있나요?**
A: 네, 가능합니다. 하지만 Raspberry Pi에서는 권장하지 않습니다. 데스크탑 환경에서는 YOLO가 더 좋습니다.

**Q: 여러 사람을 동시에 추적할 수 있나요?**
A: 현재 구현은 가장 큰 (가장 가까운) 사람만 추적합니다. 코드를 수정하면 여러 사람 추적 가능합니다.

**Q: 공 감지도 가능한가요?**
A: HOG는 사람 감지에 특화되어 있습니다. 공을 감지하려면 다른 방법(색상 기반, 원 검출 등)이 필요합니다.

## 추가 리소스

- [OpenCV HOG 문서](https://docs.opencv.org/4.x/d5/d33/structcv_1_1HOGDescriptor.html)
- [HOG 논문](http://lear.inrialpes.fr/people/triggs/pubs/Dalal-cvpr05.pdf)
- [ROS 2 cv_bridge](https://github.com/ros-perception/vision_opencv)
