# 주미 자율주행
https://www.youtube.com/watch?v=BBwEF6WBUQs

주미에서 자율주행에 도전하기에 앞서, 라즈베리파이를 이용하여 자율주행에 성공한 사례를 먼저 찾을 수 있었습니다.

https://zhengludwig.wordpress.com/projects/self-driving-rc-car/

# 목적
원글의 목적은 총 세 가지 과제를 수행하는 것이었습니다. RC 카를 개조하여

1. 트랙을 자율주행하고,
2. stop 표지판이나 빨간불을 보면 멈추며,
3. 정면 충돌을 피하기

이를 위해 필요한 기술은 OpenCV + DNN + Haar-Cascade입니다. 다행히 과정을 자세히 서술해놓아, 파이썬을 이용하여 따라서 구현해볼만 하다는 생각이 들었습니다.

# 시스템 디자인
자율주행 자동차의 시스템은 세 가지 서브 시스템으로 이루어집니다. 카메라와 초음파 센서로 이루어진 입력 유닛과, 프로세싱 유닛으로 컴퓨터를 사용하였고 모터를 조작하는 구동 유닛이 있습니다. 주미는 초음파 센서는 없으니 적외선 센서로 이를 대체하여야 하며, 보드게임을 즐길 때마다 컴퓨터에 연결하는 것은 번거로우니 최대한 라즈베리파이만으로 동작할 수 있도록 효율적으로 만들어야 하겠습니다.

![그냥 아무 사진..](https://zhengludwig.files.wordpress.com/2015/07/img_9008.jpg?w=276&h=184)

# 입력 유닛
라즈베리파이 B+와 pi 카메라, HC-SR04를 사용하였네요. 두 개의 클라이언트 프로그램이 라즈베리파이에서 컴퓨터로 비디오와 초음파 데이터를 송출한다고 합니다. 지연 시간을 줄이기 위해 비디오는 QVGA 해상도(320x240)를 채택하였습니다.

실제 구동에서는 컴퓨터를 사용하지 않는다고 하더라도, 인공지능을 학습시킬 때에는 보다 연산 능력이 좋은 컴퓨터를 사용하면 빠른 속도로 학습을 끝낼 수 있습니다. 이후 학습 모델을 라즈베리파이로 옮겨와 예측에 사용합니다. 그렇다면 역시 비디오와 적외선 센서 값을 전달하는 프로그램을 만들어야 하겠네요. 아마 첫 번째 과정이 될 것입니다. 제작은 일단 미뤄두고 다른 유닛의 제작 방법을 마저 읽어볼까요?

# 프로세싱 유닛
컴퓨터는 여러 개의 작업을 수행합니다. 라즈베리파이로부터 데이터를 받아오고, 신경망을 학습하고 나아갈 방향을 예측하며, 표지판과 신호등을 인식하고, 영상으로부터 물체의 거리를 (대략적이나마) 측정한 뒤, USB로 연결된 아두이노로 이동 명령을 전달합니다. 주미 히어로즈에는 표지판, 신호등 대신 카드가 있습니다. 물체의 거리는 적외선으로만 측정해야 간단하겠네요. 나머지는 대체로 동일해보입니다.

## TCP Server
멀티쓰레드 TCP 서버 프로그램이 이미지와 초음파 데이터를 받아온다고 합니다. 이미지는 흑백으로 변환되고 numpy array로 디코드 됩니다.

## Neural Network
신경망의 이점 중 하나는, 일단 학습되고나면 이후에는 신경망 파라미터를 불러오기만 하면 된다는 점입니다. 다시 말해, 예측이 빠릅니다. 우리에게 당장 중요한 정보는 주미와 가까운 쪽, 화면의 하단에 있기 때문에 320x240 이미지의 아래쪽 절반만 사용하여 학습과 예측을 진행합니다. 신경망 구성은 다음과 같이 38,400의 노드(320 * 120)로 구성된 Input Layer(Flatten), 32개 노드로 구성된 Hidden Layer(Dense), 4개 노드로 구성된 Output Layer(Softmax)입니다. Hidden Layer의 노드 수는 임의로 고른 거라고 하네요. 반면 Output Layer가 4개인 이유는 각각 왼쪽, 오른쪽, 직진, 후진에 해당하는 명령을 수행해야 하기 때문입니다. (물론 후진은 어디에도 사용되지 않았지만 여전히 Output Layer에 포함하였다고 합니다.)

![신경망 구조](https://zhengludwig.files.wordpress.com/2015/07/mlp_half_32-2.jpg?w=660&h=343)

아래 그림은 트레이닝 데이터 수집 작업을 보여줍니다. 각각의 프레임은 아래쪽 절반만 크롭되어 numpy array로 변환됩니다. 이후 학습 이미지는 사용자가 입력하는 학습 라벨과 매칭됩니다. 모든 이미지와 라벨은 npz 파일로 저장됩니다. neural network는 back propagation 방식을 사용하여 학습되며, 학습이 끝나면 가중치는 xml 파일로 저장됩니다. 예측을 사용하려면 학습된 xml에서 가중치를 불러와 같은 neural network를 재구축합니다.

![데이터 수집 프로세스](https://zhengludwig.files.wordpress.com/2015/07/collect_train_data.jpg?w=660&h=263)

주미에는 주행 모드가 있고, 주행 모드는 카메라를 보여주고 마우스로 버튼을 눌러 조작할 수 있습니다. 사람이 조작하는 실시간 데이터를 모은 뒤 이를 그대로 학습 데이터로 사용하는 게 어떨까요? 데이터를 로컬에 npz 형태로 저장하고 컴퓨터에 전송할 수만 있다면, 복잡하게 멀티쓰레드 TCP 서버를 구현하지 않아도 될 것 같습니다. 주행을 위한 코드도 대부분이 구현되어 있고요. 다만 해상도나 조작법, 데이터 저장 등 조금 바꿔야할 부분이 보입니다.

## 객체 인식
이 프로젝트는 형태 기반으로 접근하여 Haar feature-based cascade classifier를 사용하여 객체를 인식합니다. 각각의 물체가 각각의 분류기가 필요하며, 학습과 예측에 같은 프로세스를 따르기 때문에 이 프로젝트는 정지 표지판과 신호등 검출에만 초점을 맞췄습니다.

![샘플 예시](https://zhengludwig.files.wordpress.com/2015/08/pos_neg_samples.jpg?w=660&h=258)

유형|positive 샘플 수|negative 샘플 수|샘플 크기(픽셀)
---|---|---|---
정지 표지판|20|400|25x25
신호등|26|400|25x45

신호등의 빨간불, 초록불을 다른 상태로 인식하기 위한 전처리 과정이 필요합니다. 이미지를 흑백 변환 -> 객체 인식 -> 가우시안 필터(흐림 효과) -> 가장 밝은 부분 인식 -> 검증의 과정을 거치네요.

![전처리 과정](https://zhengludwig.files.wordpress.com/2015/07/brightest_spot.jpg?w=660&h=110)

먼저, 학습된 cascade classifier는 신호등을 찾아 해당 영역을 사각형으로 표시합니다. 이 사각형이 ROI(Region Of Interest)가 되어 해당 영역 안에서만 가우시안 필터를 적용하여 노이즈를 제거하고, 가장 밝은 지점을 찾습니다. 해당 점의 위치에 따라 초록불, 빨간불을 인식하네요.

우리는 이미 ORB(색상) 기반 특징점 추출 및 FLANN 매칭 방식을 구현하였지만, 성능에 대한 확신이 없는 상태입니다. 시간이 난다면 Haar classifier를 학습시킨 이후 비교해보는 게 좋겠죠.

## 거리 측정
라즈베리파이는 하나의 카메라 모듈만 지원합니다. 파이 카메라 대신 두 개의 USB 웹캠을 사용하기에는 차체의 무게도 늘어나고 불필요합니다. 결국 하나의 카메라 비전을 사용하여 거리를 측정하는데, ![Chu, Ji, Guo, Li and Wang (2004)](http://ieeexplore.ieee.org/xpl/abstractAuthors.jsp?tp=&arnumber=1336478&url=http%3A%2F%2Fieeexplore.ieee.org%2Fxpls%2Fabs_all.jsp%3Farnumber%3D1336478)이 고안한 방법이라고 합니다.

![수식](https://zhengludwig.files.wordpress.com/2015/07/distant.jpg?w=660&h=310)

위치와 각도를 달리하면서 찾는 모양인데, 앞서 말했다시피 해당 부분은 제외하도록 하겠습니다.

# 모터 제어 유닛
실험에 사용된 RC 카는 on/off 스위치 타입의 컨트롤러가 있습니다. 버튼이 눌리면 핀과 그라운드 사이의 저항이 0이 됩니다. 이 사이에 아두이노 보드를 연결하여 신경망의 예측 결과에 따라 컨트롤러의 버튼이 눌리는 것처럼 신호를 전달합니다. 주미는 위와 같은 컨트롤러 조작이 불필요하니 넘어가도 되겠네요.

# 결과
학습 샘플의 예측 정확도가 96%인 것에 비해, 테스트 샘플에 대한 예측은 85%의 정확도를 보였습니다. 실제 주행 상황에서 예측은 1초에 10번 이루어집니다.

Haar 특징점은 회전에 민감하지만 이번 프로젝트에서는 실제 도심 환경과는 달리 여러 각도, 형태의 표지판이나 신호등이 사용되지 않았습니다.

![distance measure](https://zhengludwig.files.wordpress.com/2015/07/drive_test01.jpg?w=280&h=211)

초음파 센서는 RC 카 정면의 장애물 거리 측정에만 사용되었고, 파이 카메라를 이용한 거리 추정은 충분히 좋은 결과를 보였습니다. 실험 결과는 다음과 같습니다.

이미지 순서|1|2|3|4|5|6|7
실제 거리(cm)|15|20|25|30|35|40|45
측정 거리(cm)|15.5|19.7|24.1|27.5|32.0|35.2|39.0

![실험 이미지 순서](https://zhengludwig.files.wordpress.com/2015/07/camera_measure2.jpg?w=660&h=371)

이 프로젝트를 통해 모노 비전을 사용한 거리 측정의 정확도는 다음 요소의 영향을 받는다는 것을 알 수 있었습니다. (1) 실제 거리를 측정할 때 발생한 오차, (2) 객체 인식 과정에서 바운딩 박스의 위치, (3) 카메라 캘리브레이션 과정의 오차, (4) 거리와 카메라 좌표 간 비선형적인 관계: 거리가 멀수록 카메라 좌표가 더 급격히 변화하며 오차가 더 커집니다.

RC 카는 성공적으로 트랙을 주행하며 정면 충돌을 피하고 정지 표지판과 신호등에 대응할 수 있습니다.

# 주미에 적용할 사안
1. 주행 모드 코드 변형하여 데이터 수집
2. 신경망 생성 후 학습
3. 객체 인식 학습
4. \+ 거리 측정