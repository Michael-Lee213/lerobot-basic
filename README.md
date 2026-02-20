# lerobot-basic

코드 출처 : https://github.com/huggingface/lerobot/tags <br>
문서 및 이미지 출처 : https://roboseasy.ai <br>

<img width="765" height="433" alt="image" src="https://github.com/user-attachments/assets/a0f02924-a8c8-4069-878a-0ca6e653391f" />


1. 설치 및 환경 설정
- git clone https://github.com/huggingface/lerobot.git (v0.4.3)

2. 가상환경 활성화
- source .venv/bin/activate

3. ffmpeg 및 LeRobot 설치
- sudo apt update
- sudo apt install ffmpeg
- git clone https://github.com/huggingface/lerobot.git

4. 로봇 하드웨어 드라이버
- Feetech 모터 (SO-ARM100, SO-ARM101)
- uv pip install -e ".[feetech]" (실제 로봇 하드웨어를 사용으로 Feetech 모터 드라이버만 설치 필요)

5.모델 학습 과정을 추적 도구 설치 
- W&B 설치
- uv pip install wandb

6. USB 포트 번호 고정
   먼저 각 장치의 시리얼 넘버(Serial Number)를 확인
   - sudo chmod 666 /dev/ttyACM0
   - sudo chmod 666 /dev/ttyACM1
     
  - 각 장치의 고유 시리얼 넘버를 확인 (ATTRS{serial}=="5AB0183022")
  - udev 규칙 파일 생성하여 SUBSYSTEM=="tty", ATTRS{serial}=="5AB0183022", SYMLINK+="so101_leader" / SUBSYSTEM=="tty", ATTRS{serial}=="5AB0182087", SYMLINK+="so101_follower" 포트 번호 고정
  - 이후 규칙 적용

7. SO-ARM Calibration
   - Leader Arm Calibration / Follower Arm Calibration
     - 아래 이미지와 같이 Calibration 결과 Calibration이 완료되면 다음과 같은 관절 위치 정보가 표시 <br><br>
     <img width="383" height="220" alt="image" src="https://github.com/user-attachments/assets/8198dfef-9229-44ee-9935-062087881ac1" />

8. SO-ARM Teleoperation
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
<br>
<img width="1815" height="739" alt="image" src="https://github.com/user-attachments/assets/584375f3-3165-499b-8be1-fba049c9cf94" />

9. 모방학습

1) HuggingFace CLI 토큰으로 로그인
hf auth login --add-to-git-credential --token YOUR_TOKEN_HERE
예시: hf auth login --add-to-git-credential --token YOUR_HUGGINGFACE_TOKEN_HERE

2) 로그인 확인 및 환경 변수 설정
HF_USER=$(hf auth whoami | head -n 1)
echo $HF_USER
> 로그인이 성공했다면 본인의 HuggingFace 사용자명이 출력

3) 시각화 도구 설치
pip install rerun-sdk

4) 데이터 수집 > 유저 네임 변경 필요 / 테스크 네임 폴더 내임 그대로 유지 > 해당 디렉토리로 자동 저장
export TASK_NAME="pick_and_place"
export HF_USER="your_username"

5) 데이터 수집 후 허깅페이스에 자동 업로드

lerobot-record \
    --teleop.type=so101_leader \
    --teleop.port=/dev/so101_leader \
    --teleop.id=leader \
    --robot.type=so101_follower \
    --robot.port=/dev/so101_follower \
    --robot.id=follower \
    --robot.cameras='{
        top: {type: opencv, index_or_path: /dev/cam_top, width: 640, height: 480, fps: 25},
        wrist: {type: opencv, index_or_path: /dev/cam_wrist, width: 640, height: 480, fps: 25},
    }' \
    --dataset.single_task=${TASK_NAME} \
    --dataset.repo_id=${HF_USER}/${TASK_NAME} \
    --dataset.num_episodes=45 \ >> 에피소드 한 프레임과 같은 의미
    --dataset.episode_time_s=15 \ 한 프레임 당 걸리는 소요 시간
    --dataset.reset_time_s=3 \ 모방 학습 액션 후 리셋 시간
    --display_data=true

> 학습 완료 후 ~/.cache/huggingface/datasets/${HF_USER}/${TASK_NAME}/ 자동 저장

6) 특정 에피소드 리플레이

lerobot-replay \
    --robot.type=so101_follower \
    --robot.port=/dev/so101_follower \
    --robot.id=follower \
    --dataset.repo_id=${HF_USER}/${TASK_NAME} \
    --dataset.episode=0
    
7) 데이터 기반 기본 학습 설정 (환경 설정)

    train
lerobot-train \
  --dataset.repo_id=${HF_USER}/${TASK_NAME} \
  --policy.repo_id=${HF_USER}/${TASK_NAME} \
  --policy.type=act \
  --policy.device=cuda \
  --job_name=act_so101 \
  --output_dir=outputs/train/act_so101/${TASK_NAME} \
  --wandb.enable=false \
  --batch_size=8 \
  --steps=100_000 \
  --save_checkpoint=true \
  --save_freq=10_000 \
  --dataset.video_backend=pyav

( 버전 차이로 인해, 오류 발생 시 예시 기존 예시 코드가 아닌 위 코드로 환경 설정 필요 ) 

8) 학습 재개
lerobot-train \
  --config_path=train_config.json \
  --resume=true

9) 평가 및 실행 시간 설정을 길게 해서 끊기지 않고 반복 작업 수행
lerobot-record \
    --policy.path=${HF_USER}/${TASK_NAME} \
    --robot.type=so101_follower \
    --robot.port=/dev/so101_follower \
    --robot.id=follower \
    --robot.cameras='{
      top: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 25},
    }' \
    --dataset.repo_id=${HF_USER}/eval_${TASK_NAME} \
    --dataset.single_task=eval_${TASK_NAME} \
    --dataset.episode_time_s=100_000 \
    --dataset.reset_time_s=1 \
    --display_data=true
   
10 ) 디렉토리 확인
   cd outputs/train/
   cd train/act_so101/pick_and_place/checkpoints/last
   cd pretrained_model/nano train_config.json
   ( CMD 화면 내 파라메타 수정 창 확인 )

   >> /hs/lerobot$ lerobot-train   --config_path=outputs/train/act_so101/${TASK_NAME}/checkpoints/last/pretrained_model/train_config.json --resume=true >> 파라메타 수정 후 재학습
    ( 현 예시는 학습 중 강제 종료 후 학습 파라메타 210정도로 수정하고 재 학습을 진행한 케이스 )

11)  평가 및 실행 시간 설정, 반복 학습 진행 ( 최종 학습 모델 확인 ) 

lerobot-record \
    --policy.path=${HF_USER}/${TASK_NAME} \
    --robot.type=so101_follower \
    --robot.port=/dev/so101_follower \
    --robot.id=follower \
    --robot.cameras='{
      top: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 25},
    }' \
    --dataset.repo_id=${HF_USER}/eval_${TASK_NAME} \
    --dataset.single_task=eval_${TASK_NAME} \
    --dataset.episode_time_s=100_000 \
    --dataset.reset_time_

[허깅 페이스 내 데이터 수집 예시] <br>

<img width="419" height="334" alt="image" src="https://github.com/user-attachments/assets/a0b474c2-ecde-4a50-b183-1d7ea0d35f8b" />



   
