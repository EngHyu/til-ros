# Zumi
주미는 인공지능 학습이 가능한 자율주행 자동차 키트로서 얼굴이나 사물을 인식시켜 인공지능의 원리를 배울 수 있습니다. 라즈베리 파이 제로와 Pi카메라, OLED 화면을 탑재하여 프로그래밍 가능하고 파이썬, 아두이노와 웹 기반 블럭 코딩 프로그램을 지원합니다.

[![thumbnail](https://scontent-gmp1-1.cdninstagram.com/v/t51.2885-15/e35/s1080x1080/118296821_121755749381359_4063394404171634639_n.jpg?_nc_ht=scontent-gmp1-1.cdninstagram.com&_nc_cat=107&_nc_ohc=rTqvxHqhIbMAX8wGedM&oh=2f128514f750f859dad4afc94de26d99&oe=5F6B5015)](https://youtu.be/h8sRoflcQMA)

# zumidashboard.ai 접속 불가
주미의 전원을 키면, zumi8667과 같이, zumi + 숫자 네 자리로 시작하는 와이파이가 탐색됩니다. 

해당 와이파이에 접속하면 주미를 제어하는 웹 페이지에 접근할 수 있는데요. zumidashboard.ai로 접속하라는 안내 문구가 표시됩니다.

![에러](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcpoSaP%2FbtqHguFBy0z%2FJ1QVF6nEAki24DzOGEK6c1%2Fimg.png)

zumi8667 같은 와이파이를 잡지 않고 해당 url에 접속하면 다음과 같은 그림이 뜨는데요.

해당 와이파이를 잡아도 같은 페이지로 안내되는 경우가 있다고 합니다. 이는 컴퓨터의 DNS 설정에 따라 다르다고 하네요...

이런 상황이 발생하는 분들의 경우 [zumi8667.local](zumi8667.local)이나 [192.168.10.1/](192.168.10.1/)로 접속하시면 같은 화면을 보실 수 있습니다!!

![대시보드](https://gblobscdn.gitbook.com/assets%2F-LrnoVjy4w-Iq5vEs2ck%2F-LuRt7uCmPPVj1Ne05Ff%2F-LuRt8dLwlyOTYTrser7%2F007.jpg?alt=media)

## ssh 연결
이를 응용하여 ssh 연결을 시도해볼 수 있습니다. 터미널에서 `ssh pi@192.168.10.1`을 통해 주미에 접속할 수 있습니다. 기본 패스워드는 pi입니다.

# wifi 연결
주미 대시보드에 접속하면, 와이파이를 설정하라는 안내가 나옵니다. 평소 사용하던 와이파이를 연결하면, 주미 와이파이에 연결된 상태에서도 구글이나 기타 검색을 이용할 수 있습니다.

와이파이는 2.4G여야 하며, 공공 와이파이, 숨김 와이파이, 5G 와이파이는 지원하지 않는다고 하네요.

![에러](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FmFmcM%2FbtqG7o0VPGg%2F9anPNZPj3Iaz6tXMtfCBF0%2Fimg.png)

문제는, 2.4G 와이파이 또한 연결되지 않는 경우가 있다는 겁니다. 분명히 와이파이 비밀번호를 맞게 입력했지만, 연결에 실패했는데요. 이는 라즈베리파이 제로와 공유기가 충돌하여 발생하는 문제였습니다.

ipTIME에서 미디어텍 칩셋을 사용한 공유기에서 이러한 문제가 발생하였고, 2017년도 10월 이후 배포된 펌웨어는 해당 문제를 해결하였다고 합니다.

원래 사용하던 와이파이를 잡은 뒤, [192.168.0.1](192.168.0.1)에 접속하여 온라인 자동 업그레이드를 진행하고 나니 깔끔하게 동작합니다.

![업그레이드 1](https://t1.daumcdn.net/cfile/tistory/99958E335A0D96FA21)

![업그레이드 2](https://t1.daumcdn.net/cfile/tistory/99D19D335A0D99630D)

# jupyter kernel error: '_xsrf' argument missing from post
와이파이를 잡지 못하고 오프라인 모드로 접속했을 때, jupyter를 실행하면 다음과 같이 커널 에러가 발생했습니다.

![에러1](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcV4zls%2FbtqG6064xuA%2F5KIt3NkkKw1CcJIKmYVsgk%2Fimg.png)

![에러2](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbKxRvD%2FbtqG6HM5FGd%2FSGrpQK2W2KpwkOJqFIk7y0%2Fimg.png)

해당 문제는 공유기 펌웨어 업그레이드를 진행한 이후 사라졌습니다... 도대체 뭐가 문제였는지 모르겠네요!!

# 출처
- [주미 FAQ](http://robolink.ipdisk.co.kr/publist/HDD1/download/file/Zumi_FAQ_v1.pdf)
- [라즈베리파이 제로W WiFi 연결 안되는 현상 (IPTIME 미디어텍 칩셋)](https://kjun.kr/495)