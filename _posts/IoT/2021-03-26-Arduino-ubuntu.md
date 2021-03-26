---
layout: post
title:  "[IoT]Ubuntu에서 arduino 2.0.0 beta4 사용"
date:   2021-03-26 14:19:35 +0900
categories: [IoT]
---

# 다운로드, 컴파일

[아두이노 사이트](https://www.arduino.cc/en/software)에서 2.0 베타를 다운받았습니다. 시간 지나면 바뀌겠지만?

- Linux 64bits 용으로 다운받습니다.
- 다운받은 압축폴더 해제하면 별도의 설치 안 하고도 실행이 되는 듯한?
- permission denied에러가 뜹니다. [이 블로그](https://sakulstra.github.io/2015/04/22/arduino_permission_denied.html)에서 설명하는 것처럼 `sudo chmod a+rw /dev/ttyACM0` 했더니 그 에러는 없어졌습니다.
- 개인 라이브러리?를 사용하려고 보니 라이브러리를 찾을 수 없어 컴파일이 안 된다는 메세지가 떴습니다. 우선 라이브러리 폴더를 압축하고, 라이브러리 메뉴를 상단 메뉴바 Sketch -> Include Library-> Add .ZIP Library 로 들어가서 압축한 파일을 선택하니까 컴파일이 되었습니다. 
- 라이브러리 설치되는 경로가 `/root/Arduino/libraries`인 것 같습니다. 