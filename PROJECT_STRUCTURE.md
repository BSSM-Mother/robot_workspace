# Robot Workspace 프로젝트 구조

## 📋 전체 구조

```
src/
├── camera_ros/          # 카메라 센서 드라이버
├── robot_base/          # 모터/휠 제어 (하드웨어 인터페이스)
├── robot_control/       # 고수준 제어 로직 (PID 제어, 장애물 회피, 추적)
├── robot_perception/    # 비전 기반 인식 (사람 감지)
├── robot_description/   # 로봇 모델 정의 (URDF, 메시)
├── robot_launch/        # 노드 실행 관리 (런치 파일)
├── robot_mqtt/          # 외부 통신 (MQTT 브릿지)
└── sllidar_ros2/        # LIDAR 센서 드라이버
```

---

## 📦 각 패키지 상세 설명

### 1. **camera_ros** - 카메라 드라이버
- **역할**: V4L2/libcamera 지원 카메라에서 이미지 수집
- **지원**: Raspberry Pi Camera, USB 카메라 등
- **출력**: `sensor_msgs/Image` 토픽으로 영상 스트림 발행
- **의존성**: libcamera, cv_bridge, camera_info_manager
- **README**: [camera_ros/README.md](camera_ros/README.md)

### 2. **robot_base** - 모터 제어 (하드웨어 인터페이스)
- **역할**: 로봇 바퀴 모터 제어 (시리얼 통신)
- **구성요소**:
  - `wheel_controller.cpp`: 시리얼 포트로 모터 명령 전송
  - 보레이트: 115200 bps
- **입력**: `geometry_msgs/Twist` (`cmd_vel_raw` 토픽)
- **출력**: 시리얼 패킷 → 모터 드라이버
- **README**: [robot_base/README.md](robot_base/README.md)

### 3. **robot_control** - 고수준 제어 로직
- **역할**: 로봇의 자율 제어 및 행동 로직
- **포함 노드들**:
  - **tracking_controller**: PID 제어로 사람 따라가기
  - **obstacle_avoider**: 라이다 데이터로 장애물 회피
  - **person_mover**: 사람 움직임에 반응하여 로봇 이동
  - **ball_randomizer**: 공 위치 랜덤화 (테스트용)
- **입력**: 카메라/라이다 센서 데이터, 사람 위치
- **출력**: `cmd_vel_raw` 토픽의 속도 명령 (robot_base로 전달)
- **README**: [robot_control/README.md](robot_control/README.md)

### 4. **robot_perception** - 비전 인식 (Python)
- **역할**: 카메라 이미지에서 사람/객체 감지
- **포함 모듈**:
  - `person_detector.py`: YOLOv8/YOLOv11 기반 사람 감지
  - ONNX 모델 또는 PyTorch 모델 지원
- **입력**: `sensor_msgs/Image` (camera_ros에서)
- **출력**: `geometry_msgs/Point` (사람 위치 좌표)
- **README**: [robot_perception/README.md](robot_perception/README.md)

### 5. **robot_description** - 로봇 모델 정의
- **역할**: 로봇의 물리적 구조 정의
- **포함**:
  - `urdf/`: 로봇 URDF 파일 (관절, 센서 구성)
  - `meshes/`: 로봇 3D 메시
  - `worlds/`: Gazebo 시뮬레이션 월드 파일
  - `rviz/`: RViz 시각화 설정
- **사용처**: RViz 시각화, 시뮬레이션, TF 변환
- **README**: [robot_description/README.md](robot_description/README.md)

### 6. **robot_launch** - 런치 파일 (시스템 통합)
- **역할**: 전체 로봇 시스템 노드 통합 실행 관리
- **포함**:
  - 런치 파일: 모든 노드, 파라미터 설정
  - 설정 파일: 각 노드별 파라미터
- **기능**:
  - 실제 로봇 모드 / 시뮬레이션 모드 선택 가능
  - Gazebo 시뮬레이터 연동
- **실행**: `ros2 launch robot_launch robot.launch.py`
- **README**: [robot_launch/README.md](robot_launch/README.md)

### 7. **robot_mqtt** - MQTT 브릿지 (외부 통신)
- **역할**: MQTT를 통해 외부 시스템과 로봇 통신
- **기능**:
  - ROS2 토픽 ↔ MQTT 메시지 양방향 변환
  - 원격에서 로봇 명령 제어 가능
- **입출력**: MQTT 브로커와 ROS2 토픽 연동
- **README**: [robot_mqtt/README.md](robot_mqtt/README.md)

### 8. **sllidar_ros2** - LIDAR 드라이버
- **역할**: 라이다 센서 데이터 수집
- **출력**: `sensor_msgs/LaserScan` 토픽으로 거리 데이터 발행
- **용도**: 장애물 감지, 방지벽 탐지
- **README**: 외부 패키지 (사용 설명은 robot_launch 참고)

---

## 🔄 데이터 흐름도

```
카메라 센서
   ↓
[camera_ros] → sensor_msgs/Image
   ↓
[robot_perception] → geometry_msgs/Point (사람 위치)
   ↓
┌─────────────────────────┐
│  robot_control 노드들    │
│ - tracking_controller   │
│ - obstacle_avoider      │ ← LIDAR 데이터 입력
│ - person_mover          │
└─────────────────────────┘
   ↓
[cmd_vel_raw] → geometry_msgs/Twist
   ↓
[robot_base] → 시리얼 통신
   ↓
   모터 드라이버
   ↓
   로봇 이동
```

---

## 🚀 빌드 및 실행

### 빌드
```bash
# 전체 빌드
colcon build

# 특정 패키지만 빌드
colcon build --packages-select robot_control
```

### 실행
```bash
# 런치 파일로 전체 시스템 실행
source install/setup.bash
ros2 launch robot_launch robot.launch.py
```

---

## 📝 주요 토픽 및 메시지 타입

| 토픽 | 타입 | 방향 | 설명 |
|------|------|------|------|
| `/camera/image_raw` | `sensor_msgs/Image` | 발행 | 카메라 원본 영상 |
| `/person_position` | `geometry_msgs/Point` | 발행 | 감지된 사람 위치 |
| `/cmd_vel_raw` | `geometry_msgs/Twist` | 수신 | 로봇 속도 명령 |
| `/scan` | `sensor_msgs/LaserScan` | 발행 | LIDAR 거리 데이터 |
| `/robot/follow_mode` | `std_msgs/Bool` | 수신 | 추적 모드 ON/OFF |

---

## 🛠️ 개발 가이드

- 각 패키지 디렉토리의 README.md 참고
- 린트: `ros2 run ament_lint_auto ament_lint_auto .`
- 포맷: `ament_uncrustify --reformat src/`
