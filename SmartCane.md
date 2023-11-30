# I. 전방 장애물 탐지 기능 알고리즘 소스코드

```arduino
#include <SoftwareSerial.h> //serial 통신을 위한 라이브러리
#include <DFRobotDFPlayerMini.h> //Micro SD 사용을 위한 라이브러리

DFRobotDFPlayerMini mp3;
const int HC06TX = 2; // HC-06의 RX Pin - Arduino Uno의 TX
const int HC06RX = 3; // HC-06의 TX Pin - Arduino Uno의 RX
const int s1 = 10;
const int s2 = 11;
SoftwareSerial blt(HC06TX, HC06RX);
SoftwareSerial spk(10, 11);
String s, sarr[4];
int pre;
char tmp;

//소리 출력 함수
void beep(int n){
//n은 물체의 번호 (1은 신호등, 2는 보행자, 3은 차량)
//물체의 종류를 뜻하는 번호를 입력받고 이에 맞는
//음성 신호를 출력한다
  spk.listen();
  mp3.play(n);
  delay(2000);
  mp3.pause();
  blt.listen();
  Serial.println(n);
  pre = n;
}
void setup() {
  s = "";
  tmp = "";
  sarr[1] = "traffic light detected ";
  sarr[2] = "person detected ";
  sarr[3] = "car detected ";
  pre = 0;
  blt.listen();
  Serial.begin(9600);
  blt.begin(9600);

  spk.listen();
  mp3.begin(spk);
  spk.begin(9600);
  mp3.volume(10);

  blt.listen();
  Serial.println("Start");
}
void loop() {
  blt.listen();
  if(blt.available()){
    tmp = blt.read();
   //블루투스 통신으로 읽어온 문자를 tmp에 저장
    if(tmp == '!'){
   //앱에서 보내는 문자열의 마지막은 항상 '!'임
   //따라서 tmp가 '!'라면 한 줄을 입력받았다는 뜻
      if(!s.equals(sarr[pre])){
     //문자열이 저장된 배열 sarr과 현재 문자열 s를 비교하여
     //현재 객체인식 앱에서 감지한 물체의 종류를 파악함 
        for(int i = 1; i<=3; i++)
          if(s.equals(sarr[i])){
            beep(i); break;
          }
      }
      s = "";
    }
    else{
   //tmp가 '!'가 아니라면 한 줄을 다 입력받지 않았다는 뜻이므로
   //입력받은 문자 tmp를 현재 문자열 s에 더함
      s += tmp;
    }
  }
  if(Serial.available()){
    blt.write(Serial.read());
  }
}
```

# II. 점자블록 감지 기능 알고리즘 소스코드
```arduino
#define S0 4
#define S1 5
#define S2 6
#define S3 7
#define sensorOut 8

int redfrequency = 0;
int greenfrequency = 0;
int bluefrequency = 0;

#include <Servo.h>
Servo sg90;

int sg90Pin = 9;
int Angle = 0;
int val = 0;

bool yellowDetected = false; // 노란색 감지 여부를 확인하는 변수

void setup() {
  pinMode(S0, OUTPUT); // S0부터 S3까지: 컬러 센서의 제어 핀
  pinMode(S1, OUTPUT);
  pinMode(S2, OUTPUT);
  pinMode(S3, OUTPUT);
  pinMode(sensorOut, INPUT); // 컬러 센서의 출력 핀
  sg90.attach(sg90Pin); // 9번 핀에 서보 모터를 연결
  
  // 주파수 스케일링을 20%로 설정
  digitalWrite(S0, HIGH);
  digitalWrite(S1, LOW);
  
  Serial.begin(9600);
}

void loop() {
  // 빨간색 구성 요소 읽기
  digitalWrite(S2, LOW);
  digitalWrite(S3, LOW);
  redfrequency = pulseIn(sensorOut, LOW);
 
  delay(100);
  
  // 초록색 구성 요소 읽기
  digitalWrite(S2, HIGH);
  digitalWrite(S3, HIGH);
  greenfrequency = pulseIn(sensorOut, LOW);
 
  delay(100);
  
  // 파란색 구성 요소 읽기
  digitalWrite(S2, LOW);
  digitalWrite(S3, HIGH);
  bluefrequency = pulseIn(sensorOut, LOW);

  // 색상이 노란색인지 확인
  if (redfrequency > 25 && redfrequency < 77 && greenfrequency < 100) {
    Serial.println("YELLOW COLOUR");
    yellowDetected = true;
  } else {
    Serial.println("NO COLOUR DETECTION");
    yellowDetected = false;
  }

  if (yellowDetected) {
    loop2(); // 노란색 감지 시 loop2 함수 호출
  }

  delay(1000);  // 가독성을 위한 딜레이
}

void loop2() {
  // 아날로그 입력에 따라 서보 모터 제어
  while (true) {
    val = analogRead(A1);
    Angle = map(val, 0, 1023, 0, 179);
    sg90.write(Angle);
    Serial.print("Angle: ");
    Serial.println(Angle);
    delay(10);
    
    // 노란색이 더 이상 감지되지 않으면 루프 종료
    if (!yellowDetected) {
      break;
    }
  }
}
```
