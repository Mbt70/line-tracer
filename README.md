## Line Tracer Project (공학 입문 설계)

### 프로젝트 설명
Arduino를 활용한 Line Tracer 제작 및 분석 프로젝트입니다. 이 프로젝트는 주어진 라인을 따라가며 장애물을 피하는 로봇을 제작하는 것을 목표로 합니다.

### 주요 구조도
![image](https://github.com/Mbt70/line-tracer/assets/57335420/d689df0d-e253-47d0-b2b3-92d7d986f440)

### 작동 영상
[프로젝트 작동 영상 보기](https://youtube.com/shorts/AVQpFn3z9r4?si=4qH7P01z5sV1_nCx)

![image](https://github.com/Mbt70/line-tracer/assets/57335420/25ae97ab-c8ed-4b23-8a4b-e08f9a4a3a75)

### 팀 구성
- 전상환 교수님
- 10376 공학입문설계
- 6조: 종박사님을 아세요

### 목차
1. 설계 요건 및 제약점 정리
2. 자료 조사 분석
   - Arduino를 활용한 Line Tracer 제작 사례 및 부품 조사 분석
   - Motor shield 사용 방법
3. 창의적 아이디어 도출 및 제작 계획
4. 제작 및 시험
5. 수정 보완 및 최종 성과물

### [ASK] 설계 요건 및 제약점 정리
#### 설계 요건
- 주어진 라인을 따라 장애물을 피하는 Line Tracer 제작
- 센서, 모터, 액츄에이터 등 재료 사용

#### 제약점
- 주어진 교재의 재료만 사용
- 모든 부품은 분해가 가능해야 함
- 환경 및 하드웨어 제약 고려

### [IMAGINE] 창의적 아이디어 도출
#### 기능성
- 스텝 모터 및 초음파 센서 사용
- 조도 센서, 진동 충격 센서, 적외선 센서 적용

#### 심미성
- 5파이 그린 LED, 10mm RGB LED 사용
- 텍스트 LED 및 피에조 스피커로 경고 기능 구현

### [PLAN] 제작 계획
1. 하드웨어 요구사항 분석
2. 서보 모터 및 바퀴 배치 계획
3. 적외선 센서 및 위치 선정
4. 초음파 센서 위치 및 각도 조정
5. 아두이노 Uno 설치 위치 및 전원 관리

### [CREATE] 제작 및 시험
1. **기능성 관점에서 제작 방향**
   - 전진, 후진, 좌우회전 기능 구현
   - 장애물 감지 및 회피 기능 구현
2. **심미성 관점에서 제작 방향**
   - 구급차 컨셉의 LED 및 경고음 구현
   - 초음파 센서를 활용한 장애물 감지 후 경고음 발생

### [IMPROVE] 수정 보완 및 최종 성과물
1. **결함 및 오류 분석**
   - 센서 캘리브레이션 오류 수정
   - 모터 제어 오류 수정
   - 전원 공급 및 전압 문제 해결
2. **최종 성과물 설명**
   - 라인 트레이서가 선을 따라 주행하며 장애물을 회피하는 동작을 완성

### 하드웨어 및 소프트웨어 설명
#### 하드웨어 구성
- 아두이노 Uno, 적외선 센서, 초음파 센서, LED, 피에조 스피커 등 사용

#### 소프트웨어 코드
```cpp
#include <NewPing.h>
#include <AFMotor.h>

// hc-sr04 sensor
#define TRIGGER_PIN A0
#define ECHO_PIN A1
#define MAX_DISTANCE 50

// ir sensor
#define irLeft A5
#define irRight A4   
#define irbLeft A2
#define irbRight A3
#define LED_PIN 10
#define SPEAKER_PIN 11

NewPing sonar(TRIGGER_PIN, ECHO_PIN, MAX_DISTANCE);
AF_DCMotor motor1(1, MOTOR12_1KHZ);
AF_DCMotor motor2(4, MOTOR12_1KHZ);
AF_DCMotor motor3(2, MOTOR34_1KHZ);
AF_DCMotor motor4(3, MOTOR34_1KHZ);

int distance = 0;
int leftDistance;
int rightDistance;
boolean object;
int bLeftValue;
int bRightValue;

void setup() {
  Serial.begin(9600);
  pinMode(irLeft, INPUT);
  pinMode(irRight, INPUT);
  pinMode(irbLeft, INPUT);
  pinMode(irbRight, INPUT);
  pinMode(LED_PIN, OUTPUT);
  motor1.setSpeed(93);
  motor2.setSpeed(93);
  motor3.setSpeed(93);
  motor4.setSpeed(93);
}

void loop() {
  if (isPathClear()) {
    if (digitalRead(irLeft) == 0 && digitalRead(irRight) == 0) {
      moveForward();
    } else if (digitalRead(irLeft) == 0 && digitalRead(irRight) == 1) {
      Serial.println("TL");
      moveLeft();
    } else if (digitalRead(irLeft) == 1 && digitalRead(irRight) == 0 ){
      Serial.println("TR");
      moveRight();
    } else if (digitalRead(irLeft) == 1 && digitalRead(irRight) == 1) {
      Stop();
      delay(100);
    }
  } else {
    motor1.setSpeed(93);
    motor2.setSpeed(93);
    motor3.setSpeed(93);
    motor4.setSpeed(93);
    int check = 0;
    while (1) {
      digitalWrite(LED_PIN, HIGH);
      bLeftValue = digitalRead(irbLeft);
      bRightValue = digitalRead(irbRight);
      if (bLeftValue == 0 && bRightValue == 0) {
        moveBackward();
      } else if (bLeftValue == 0 && bRightValue == 1) {
        motor1.run(BACKWARD);
        motor2.run(BACKWARD);
        motor3.run(FORWARD);
        motor4.run(FORWARD);
      } else if (bLeftValue == 1 && bRightValue == 0) {
        motor1.run(FORWARD);
        motor2.run(FORWARD);
        motor3.run(BACKWARD);
        motor4.run(BACKWARD);
      } else {
        Stop();
        dest();
        delay(1000);
        break;
      }
    }
    digitalWrite(LED_PIN, LOW);
  }
}

void dest() {
  int cnt = 0;
  while (cnt < 30) {
    digitalWrite(LED_PIN, HIGH);
    delay(500);
    digitalWrite(LED_PIN, LOW);
    delay(500);
    cnt += 1;
  }
}

boolean isPathClear() {
  distance = getDistance();
  if (distance >= 15) {
    return true;
  } else {
    digitalWrite(LED_PIN, HIGH);
    Stop();
    delay(100);
    distance = getDistance();
    if (distance < 15) {
      return false;
    } else {
      return true;
    }
  }
}

int getDistance() {
  delay(50);
  int cm = sonar.ping_cm();
  if (cm == 0) {
    cm = 100;
  }
  return cm;
}

void Stop() {
  motor1.run(RELEASE);
  motor2.run(RELEASE);
  motor3.run(RELEASE);
  motor4.run(RELEASE);
}

void moveForward() {
  motor1.run(FORWARD);
  motor2.run(FORWARD);
  motor3.run(FORWARD);
  motor4.run(FORWARD);
}

void moveBackward() {
  motor1.run(BACKWARD);
  motor2.run(BACKWARD);
  motor3.run(BACKWARD);
  motor4.run(BACKWARD);
}

void moveRight() {
  motor1.setSpeed(115);
  motor2.setSpeed(115);
  motor3.setSpeed(115);
  motor4.setSpeed(115);
  motor1.run(BACKWARD);
  motor2.run(BACKWARD);
  motor3.run(FORWARD);
  motor4.run(FORWARD);
}

void moveLeft() {
  motor1.setSpeed(115);
  motor2.setSpeed(115);
  motor3.setSpeed(115);
  motor4.setSpeed(115);
  motor1.run(FORWARD);
  motor2.run(FORWARD);
  motor3.run(BACKWARD);
  motor4.run(BACKWARD);
}
```

### 참고 문헌
1. 김철민, 신호준, 조희영, 조희영, 윤태성. (2022). RFID 기반 최단시간 알고리즘 라인트레이서. 한국전자통신학회 논문지, 17(6), 1221-1228.
2. [러봇랩 - [아두이노기초] 거리감지 센서의 종류와 원리](https://youtu.be/io7RqNrnGpI?si=yVB3531seymtVgBJ)
3. [미래인재연구소 - 아두이노고급 2강 아두이노 이론 액츄에이터 사용하기](https://www.youtube.com/watch?v=AV2QZ6

zT9sQ)
4. [DIY Builder - How To Make A DIY Arduino Line Follower Car At Home](https://www.youtube.com/watch?v=t7k9D1jDEtk)
5. [Mr ProjectsoPedia - Arduino obstacle avoiding line follower robot projects code 2021](https://www.youtube.com/watch?v=ZiqAyuLpS3o)
