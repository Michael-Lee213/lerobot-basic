# lerobot-basic

코드 출처 : https://github.com/huggingface/lerobot/tags <br>
문서 출처 : https://roboseasy.ai <br>

1. 설치 및 환경 설정
- git clone https://github.com/huggingface/lerobot.git (v0.4.3)

2. 가상환경 활성화
- source .venv/bin/activate

3. ffmpeg 및 LeRobot 설치
- sudo apt update
- sudo apt install ffmpeg
- git clone https://github.com/huggingface/lerobot.git

5. 로봇 하드웨어 드라이버
# Feetech 모터 (SO-ARM100, SO-ARM101)
- uv pip install -e ".[feetech]" (실제 로봇 하드웨어를 사용으로 Feetech 모터 드라이버만 설치 필요)

6.모델 학습 과정을 추적 도구 설치 
# W&B 설치
- uv pip install wandb

7. USB 포트 번호 고정
   먼저 각 장치의 시리얼 넘버(Serial Number)를 확인
   - sudo chmod 666 /dev/ttyACM0
   - sudo chmod 666 /dev/ttyACM1
     
  - 각 장치의 고유 시리얼 넘버를 확인 (ATTRS{serial}=="5AB0183022")
  - udev 규칙 파일 생성하여 SUBSYSTEM=="tty", ATTRS{serial}=="5AB0183022", SYMLINK+="so101_leader" / SUBSYSTEM=="tty", ATTRS{serial}=="5AB0182087", SYMLINK+="so101_follower" 포트 번호 고정
  - 이후 규칙 적용

8. SO-ARM Calibration
   - Leader Arm Calibration / Follower Arm Calibration
     - 아래 이미지와 같이 Calibration 결과 Calibration이 완료되면 다음과 같은 관절 위치 정보가 표시
     <img width="383" height="220" alt="image" src="https://github.com/user-attachments/assets/8198dfef-9229-44ee-9935-062087881ac1" />

9. SO-ARM Teleoperation
    - 모방 학습 (카메라 index 찾기 > 카메라 추가 설정 진행)

lerobot-teleoperate \
  --teleop.type=so101_leader \
  --teleop.port=/dev/so101_leader \
  --teleop.id=leader \
  --robot.type=so101_follower \
  --robot.port=/dev/so101_follower \
  --robot.id=follower \
  --robot.cameras='{
      top: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 25},
  }' \
  --display_data=true
  
  - wrist 리스트 삭제 필요,  index_or_path: 2 > 0 0으로 설정, 연결되어 있는 카메라 중 웹캠 제외 필요

[실행 이미지]
<img width="1815" height="739" alt="image" src="https://github.com/user-attachments/assets/584375f3-3165-499b-8be1-fba049c9cf94" />



   
