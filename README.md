# Ubuntu20.04 / ROS2-foxy / TurtleBot3 <details open="open">
#### (따라하기 쉽게 작성하는 것을 목표로 하였습니다. (https://github.com/krshim/ros2-study)를 참고하세요.)
<summary>목차</summary>
  <ol>
    <li><a href="#터틀봇3-세팅">터틀봇3 세팅</a></li>
  </ol>
</details>

<a id=터틀봇3-세팅></a>

## 1. TurtleBot3 세팅
### 1-1. PC Setup
ROS 의 설치는 이미 되어 있으므로 turtlebot3 에 관련된 설치 파일만을 진행 한다.

가제보는 ROS2에서 제공하는 시뮬레이션 모듈이다. 매우 유용한 기능이 많으므로 사용법을 숙지해 두는게 좋고 로봇 시뮬레이션을 사용하기 위해서는 URDF 작성법도 같이 익혀 두는게 좋다.
```
sudo apt-get install ros-foxy-gazebo-* # 가제보 설치
```

카토그래퍼는 ROS2에서 지원하는 SLAM 의 하나로 터틀봇3 에서는 라이다 센서 를 기반으로 지도를 제작하게 된다.
```
sudo apt install ros-foxy-cartographer
sudo apt install ros-foxy-cartographer-ros
```
제작한 지도를 바탕으로 터틀봇을 움직이기 위해 설치하는 모듈이다. BT(behavior Tree)를 기반으로 잘 짜여진 네비게이션 모듈이다.
```
sudo apt install ros-foxy-navigation2
sudo apt install ros-foxy-nav2-bringup
```
터틀봇을 구동하기 위해 필요한 패키지들을 설치한다. ROS2-foxy 버전은 관련 패키지가 ROS2의 공식 Debian 저장소에 있기 때문에 apt를 사용하여 다운받을 수 있다. 하지만 apt를 사용하여 빌드된 패키지를 설치하게 되면 미리 제작된 바이너리를 사용하기 때문에 커스터마이징이 제한적이다. 따라서 사용자가 직접 코드를 수정하기 위해서 로보티즈 공식 깃허브에서 패키지를 복제하여 사용하는 것을 추천한다.

apt를 사용하여 설치(비추천)
```
sudo apt install ros-foxy-dynamixel-sdk
sudo apt install ros-foxy-turtlebot3-msgs
sudo apt install ros-foxy-turtlebot3
```
git clone을 사용하여 설치(추천) / 로보티즈 공식 깃허브에서 그냥 `git colne`을 하게 되면 `colcon build`할 때 ROS1에서 사용되는 catkin 빌드 툴과 충돌을 일으킬 수 있으므로 `-b` 명령어를 이용하여 원하는 브랜치(여기서는 foxy-devel)만 다운받도록 한다.
```
mkdir -p ~/turtlebot3_ws/src #turtlebot3 폴더와 그 안에 src폴더(작업공간) 생성
cd ~/turtlebot3_ws/src/
git colne -b foxy-devel https://github.com/ROBOTIS-GIT/DynamixelSDK.git
git clone -b foxy-devel https://github.com/ROBOTIS-GIT/turtlebot3_msgs.git
git clone -b foxy-devel https://github.com/ROBOTIS-GIT/turtlebot3.git
cd .. # cd ~/turtlebot3_ws
colcon build --symlink-install # 파일이 복사되지 않고 소스 파일을 가리키는 심볼릭 링크가 생성되기 때문에 사용되는 디스크 공간의 양이 줄어들어 빌드를 더 빠르게 할 수 있다.
```

터틀봇3 도메인 아이디를 설정한다.( .bashrc 파일을 에디터로 열어서 중복되지 않게 설정을 확인한다.)
```
echo 'export ROS_DOMAIN_ID=30 #TURTLEBOT3' >> ~/.bashrc$ source ~/.bashrc
```

### 3-2. TurtleBot3 SBC Setup

터틀봇3 의 라즈베리파이4에는 Ubuntu 20.04 server 64-bit 를 설치해서 구동을 한다. (https://emanual.robotis.com/docs/en/platform/turtlebot3/overview/)

