# 라즈베리파이 C 서버

## 목표

- 라즈베리파이4 에서 아파치 웹서버 대신에 C언어로 만든 HTTP 서버에 Vue 프로젝트 배포해보기

## 순서

1. 리눅스 명령어 nohup 을 사용해서 백그라운드로 웹서버 가동
2. Vue 프로젝트에서 axios 하기 위한 data.html 파일 내용 업데이트
3. Vue 배포 파일 옮기기
4. 라즈베리파이 부팅 시 자동으로 서버 가동, 데이터 업데이트 설정

## 참고

[C언어 HTTP 서버](https://www.notion.so/C-HTTP-426fa63fd6dc43f9b4866d1491d4dc59)

[로또 웹](https://www.notion.so/05d7e67ad0b540bba27f2f1636e9e89b)

## 웹서버 가동

![스크린샷 2021-10-07 오후 3.06.20.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e5f456f0-d3af-4a3d-ad50-67c77b21ab60/스크린샷_2021-10-07_오후_3.06.20.png)

C언어 HTTP 서버 페이지에서 작성한 코드를 백그라운드로 실행시켜준다.

포트는 80번을 사용했다.

약간의 코드 수정이 있었는데 /getdata 로 접속하면 data.html 을 불러오도록 변경했다.

```jsx
if (!strcmp(safe_uri, "/getdata")) strcpy(safe_uri, "/data.html");
```

## 데이터 업데이트

기존에 로또 웹 프로젝트에서 로또 데이터를 받아올 때는 아파치 웹서버에 배포한 json 정보를 받아왔었다.

C 서버로 전환하면서 아파치 서버는 중지 시켰기 때문에 더 이상 해당 주소로 데이터를 받을 수 없기 때문에 약간의 수정이 필요했다.

로또 데이터를 라즈베리파이에서 크롤링하고 있었기 때문에 해당 json 파일의 내용을 html 파일로 옮겨주는 방법으로 해결했다.

sh 파일을 만들고 crontab 을 이용해 매주 토요일 22시마다 업데이트 되도록 구현했다.

```bash
#! /bin/bash
cat /home/pi/Lotto/lotto.json > /home/pi/C-Server/data.html
```

![스크린샷 2021-10-07 오후 3.15.22.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5a0bc194-0274-4f6c-9b0d-35ad1558e3e9/스크린샷_2021-10-07_오후_3.15.22.png)

## Vue 프로젝트 옮기기

로또 웹에서 npm run build 로 만든 배포 파일을 라즈베리파이로 옮겨줬다.

## 결과

[](https://hyeokjoon.com)

정상적으로 배포가 된 모습이다.

## 라즈베리파이 부팅시 설정

라즈베리파이가 비정상적으로 종료 됐을 때를 대비해 다시 부팅을 하면 자동으로 웹서버를 가동하고 데이터를 업데이트 할 수 있게 설정했다.

```bash
#! /bin/bash
nohup sudo /home/pi/C-Server/http 80 &
nohup python /home/pi/Lotto/LottoData.py &
```

부팅 시 실행할 sh 파일을 작성해준다.

```bash
sudo chmod +x run_on_boot.sh
```

실행 권한을 설정해준다.

```bash
sudo vi /etc/rc.local
```

부팅 후 실행을 위해 /etc/rc.local 파일을 수정해준다.

```bash
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.

# Print the IP address
_IP=$(hostname -I) || true
if [ "$_IP" ]; then
  printf "My IP address is %s\n" "$_IP"
fi

# 여기에 실행하고자 하는 파일의 실행 명령어를 입력하고 뒤에 &를 붙인다.
/home/pi/C-Server/run_on_boot.sh &

exit 0
```

이렇게 작성을 하고 저장을 해준다.

```bash
sudo reboot
```

재부팅 후 실행이 되는지 확인한다.
