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
echo export ROS_DOMAIN_ID=30 >> ~/.bashrc (echo export ROS_DOMAIN_ID=30를 (.bashrc파일의 가장 아래에 추가)
source ~/.bashrc
```

### 3-2. TurtleBot3 SBC Setup

이 작업에서 사용하는 SBC는 라즈베리파이4B이다. 로컬 컴퓨터에서 하는 작업은 [L] 로 표기하고 SBC(라즈베리파이)에서 하는 작업은 [S]로 표기하겠다.

1. Ubuntu 20.04 server 64-bit 를 설치해서 구동 한다.
#### [L]
공식 ubuntu 사이트(https://releases.ubuntu.com/focal/)에서 Ubuntu 20.04.6 LTS (Focal Fossa) - Server install image를 다운받는다. Desktop image에서 다운받지 않았기 때문에 그래픽사용자 인터페이스가 설치되지 않는다. 저장공간을 확보하기 위해 이러한 방식을 사용하였다.

micro sd카드 리더기를 이용하여 라즈베리파이의 micro sd카드를 로컬 컴퓨터에 연결한다.

그 후 로컬 컴퓨터에 설치한 Ubuntu 이미지를 라즈비안을 통해 micro sd카드에 설치한다.

2. 라즈베리파이의 마이크로 sd카드에 ubuntu 20.04를 설치하였으면 라즈베리피이에 마이크로 sd카드를 삽입한다. 그 후 라즈베리파이에 HDMI와 키보드, 마우스 등 입력장치를 연결하기 전 전원을 인가한다.(입력장치를 전원을 먼저 인가하게 되면 입력장치가 인식이 되지 않는다.) 라즈베리파이의 초기 아이디는 `pi`이고 초기 암호는 `raspberry`이다.
집에서 혼자 작업한다면, 초기설정을 사용해도 무리가 없다. 하지만 외부로 인터넷을 연결하여 사용한다면 보안상의 이유로 비밀번호를 변경해주어야 한다.
+ #### [S]
>nano 편집기를 이용하여 인터넷을 설정할 수 있는 .yaml 파일을 수정한다.
>>```
>>sudo nano /etc/netplan/50-cloud-init.yaml
>>```
>파일을 열고 `WIFI_SSID:`를 `내 와이파이 이름:`으로 변경한다.
>비밀번호는 아래와 같이 작성한다.
>>```
>>password:{와이파이 비밀번호}
>>```
>작성을 완료한 후 `ctrl+s(저장)`, `ctrl+c(종료)`
>아래 코드를 입력하여 업데이트 설정을 변경한다.
>>```
>>sudo nano /etc/apt/apt.conf.d/20auto-upgrades
>>```
>nano 편집기가 열리면 아래의 코드를 추가한다.
>>```
>>APT::Periodic::Update-Package-Lists "0";
>>APT::Periodic::Unattended-Upgrade "0";
>>```
>작성을 완료한 후 `ctrl+s(저장)`, `ctrl+c(종료)`
>라즈베리파이를 재부팅합니다.
>>```
>>reboot
>>```
>net-tools를 설치하고 라즈베리파이의 현재 IP를 확인하기 위해 아래 명령어를 입력한다.
>>```
>>sudo apt update
>>sudo apt install net-tools
>>ifconfig
>>```
wlan0: 의 내용 중에서 inet옆의 IP주소를 확인한다.
+ #### [L]
라즈베리파이를 재부팅한 후 로컬 컴퓨터에서 라즈베리파이 원격접속을 진행한다.
>`ssh {라즈베리파이 사용자}@{라즈베리파이의 IP주소}`
> 이후 로컬컴퓨터와 같은 버전의 ROS2-foxy를 설치한다.
> ```
sudo apt install python3-argcomplete python3-colcon-common-extensions libboost-system-dev build-essential
sudo apt install ros-foxy-hls-lfcd-lds-driver
sudo apt install ros-foxy-turtlebot3-msgs
sudo apt install ros-foxy-dynamixel-sdk
sudo apt install libudev-dev
