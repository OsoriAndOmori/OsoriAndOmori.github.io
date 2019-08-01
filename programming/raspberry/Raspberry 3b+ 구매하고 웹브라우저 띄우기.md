아직도 헤딩을 하고 있긴 하지만 재밌는 부분이 많아 라즈베리파이 적용 공유 드립니다.

편하게 쓰는걸 좋아해서 블로그 쓰는 기분으로 진짜 편하게 쓰겠습니당..
딱딱하게 쓰면 쓰면서도 재미가 없어가지고..
```
<목차>
0. 고전 게임기계를 라즈베리파이로 만들 생각하고 있었음
1. 라즈베리파이를 삼.
2. 모든걸 깨끗하게 포맷한다.
3. 마음에드는 OS를 SD 카드에 구운다.  
4. 자잘한 패치한다.(한글, 스크린세이버, 원격 공유폴더 등등)
5. 와이파이 잡는다. 
```
----------------------------------------------------------
## 0. 고전 게임기계를 라즈베리파이로 만들 생각하고 있었음
<details><summary>클릭!</summary>
구글링을 하다보면 게임 기계를 자기손으로 만든걸 인증하는 용자들이 있습니다.
<img width="809" alt="34c3780e-d30a-11e8-9af7-8cdd09eb32ed" src="https://user-images.githubusercontent.com/22016317/48462130-45a47680-e81a-11e8-8904-9bf2932bf306.png">

[라즈베리파이를 이용한 레트로 게임기 만들기](http://rect2ellipse.com/?p=539)
꽤 엔틱해보여서 이런거 하나 잘 집에하나 모셔놓으면 손님이나 애들이 왔을 때도 놀기 좋겠구나 생각하고 있었죠.
</details>

## 1. 라즈베리파이(컴퓨터)를 산다.
<details><summary>클릭!</summary>
일단 될지 안될지도 모르는데 아무리 제가 연구하고 싶어도 사비로 사긴 아까워가지고
사내 모니터링 툴 만든다는 멋진 명분으로 라즈베리파이 스타트 키트 2개를 샀습니다. 

[모델은 라즈베리파이3 b+](http://www.icbanq.com/P008271082)
- 무선 랜 wifi 지원,
- hdmi, mini display, usb 4개, 코어 램 등등은 생략~

<img width="556" alt="8d629416-d30c-11e8-8798-e382e6434640" src="https://user-images.githubusercontent.com/22016317/48462129-45a47680-e81a-11e8-835b-6b53d87fd906.png">

</details>


## 2. 모든걸 깨끗하게 포맷한다.
<details><summary>클릭!</summary>

![a8c2fc94-d30e-11e8-9bfe-c2f6b00a94a4 1](https://user-images.githubusercontent.com/22016317/48462180-8b613f00-e81a-11e8-9024-ea87bb7b72b8.png)


배송은 한 5일 걸렸던거 같습니다. 아주 기대하고 있었는데, 회사에서 사주는거라 역시 느립니다.
무튼 도착해서 까보니 역시 스타트 키트 답게 뭐가 많습니다. 이것저것 끼고 코드도 꼽아보니 불도 들어오는군요.
불량은 아닌 것 같습니다.

그리고 한번 생각해봅니다.
<라즈베리파이 = 씨피유, 메모리, 메인보드>, < SD카드 = 하드디스크 > < 전원장치 = 전원장치 >
</details>

## 3. 마음에드는 OS 를 SD(하드디스크) 카드에 구운다.
<details><summary>클릭!</summary>
그럼 상식적으로 SD 카드를 포맷해서 날것으로 만든다음 부팅디스크를 만들면 될 것 같습니다.

[Raspberrypi Official Home](https://www.raspberrypi.org/downloads/)  에 가보면 Official OS 2개  Noob, Raspbian 을 포함해 우분투, 윈도우 까지 대충 10개를 오픈소스 OS로 공개하고 있는데요.
우분투로 대학교 때 개발을 해서 우분투 깔고 싶었지만, 초심자 답게 "공식 OS 라즈비안" 깔기로합니다. 
근데 끝나고 나서 든 생각은 다음에 다시 설치하면 우분투 깔 겁니다. 유투브로 남이 하는거 보니 익숙해서 편해보이네요.

포맷은 [SD Card Formatter](https://www.sdcard.org/downloads/formatter_4/)를 이용합니다.
제곧내를 이럴 때 쓰는건지 이름보니 포맷 잘할 것 같습니다. 
제 imac 에 SD 카드를 꼽으니 알아서 스펙이 다 나오고 좋네요. 전 이런 UI단순하고 원클릭으로 해결하는걸 좋아합니다. 기다리기 귀찮으니 빨라보이는 Quick format을 합니다.

포맷은 생각보다 오래 걸립니다. 한 5분 ~ 10분 기다렸던거 같습니다. 중간에 멈춘거 같았는데 곧 잘 되더라구요.
형편없는 Finish Alert 를 구경하고 이제 라즈비안을 구울 차례입니다.

굽는 툴은 [Etcher](https://etcher.io/) 로 합니다. 다운받으러 갔더니 [spring.io](https://spring.io/) 만큼 대문이 자극적입니다.


<img width="600" alt="66ccd6e2-d30f-11e8-9ea8-623abbd2fe6f" src="https://user-images.githubusercontent.com/22016317/48462131-45a47680-e81a-11e8-82ab-9a6d093ec9f7.png">

역시 사용방법은 포맷보다 더 단순합니다. 키니깐 SD 카드도 자동으로 다 선택되어있네요. 그냥 다운받은 Raspbian 이미지 클릭하고 굽기 누릅니다. 이것도 꽤 걸립니다.

</details>

## 4. 다 연결하고 자잘한 패치한다.(한글, 스크린세이버, 원격 공유폴더 등등)
<details><summary>클릭!</summary>
케이블도 연결하고 이것저것 Tv 에 연결해서 부팅해봅니다.
아주 단순한 OS configuration 이후 배경화면이 보이는데 헐.

<img width="829" alt="80a7191c-d317-11e8-8f82-ed089b2de22b" src="https://user-images.githubusercontent.com/22016317/48462132-463d0d00-e81a-11e8-95b5-7ced6f461979.png">
한글이 깨집니다. 모든 글자가 네모로보여요.
작업표시줄에 있는 터미널을 클릭해서 한글패치를 합니다. 삽질은 했지만 답은 간단합니다.

루트권한을 따로 조작하지 않았더니 sudo 그냥 치면 다 되네요. 편합니다.

```
sudo apt-get install fonts-unfonts-core
```

모니터가 잠자면 안되니깐 화면보호기를 찾는데 놀랍게도 라즈비안은 스크린세이버가 없습니다. 역시 설치..
커맨드로 설정파일 수정하는 법도 있다는데, 역시 제가 하면 안됩니다. 그러니 그냥 어플 설치

```
sudo apt-get install xscreensaver
```

언제 또 써먹을지 모르공유 폴더를 설치해봅니다.

```
sudo apt-get install netatalk
```

맥에서 바로 넣을 수가 있네요.ㅋㅋ 굿


<img width="355" alt="904482fe-d314-11e8-8d62-b68172ad1038" src="https://user-images.githubusercontent.com/22016317/48462133-463d0d00-e81a-11e8-976f-ff20095ba728.png">

</details>

## 5. 와이파이 잡는다. 
<details><summary>클릭!</summary>

보안 WPA / 802.1X 형식의 WIFI 는 라즈비안에서 기본으로 받아들이질 못하는데요.
직접 Command 로 들어가 configure 수정을 하면 부팅하면서 억지로 가능하긴합니다. 

WiFi 관련 네트워크 지식이 부족해 값들을 잘못 쓴건지 동작을 하진않네요. ㅠ 일단 실패
wlan도 MAC Address 신청하고 사용해야하니 참고하시면 될 것 같습니다.

```
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
```

```json
network={
    ssid="와이파이이름"
    key_mgmt=WPA-EAP
    ear=PEAP
    identify="아디"
    password="비밀번호"
    priority=1 #컴키면 1등으로 잡아줌
}

```

</details>

-----------------------

결국 한건 새거 사와서 OS 깔고 패치하고 끝이지만, 시도를 해본 자체로 의미 있었던 것 같습니다. 
일단 게임기부터 만들고 개인적으로 더 회사 서비스에 맞춰서 디벨롭해볼게 없나 고민해볼까합니다.

- 참고
  [라즈베리공홈](https://www.raspberrypi.org/)
  기타 구글 블로그 몇 개