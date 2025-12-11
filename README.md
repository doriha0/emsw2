# 📡 스마트 알림 시계 시스템  
Arduino + Processing + App Inventor + 조도센서(LDR) 기반 스마트 알람 시스템

---

## 📌 1. 시스템 개요
본 프로젝트는 **스마트폰(App Inventor)** → **PC(Processing)** → **Arduino Uno** 의 구조로 알람을 처리하는 시스템이다.

스마트폰에서 원하는 알람 시간을 입력하면 Processing이 PC의 시간을 기준으로 해당 시각을 체크한 후 Arduino로 명령을 전송한다. Arduino는 Piezo 부저와 LED, 조도센서를 활용하여 알람을 울리고 주변 밝기를 감지한다.

---

## 📦 2. 구성 요소 (Hardware)
| 부품 | 기능 |
|------|------|
| Arduino Uno | 메인 마이크로컨트롤러 |
| Piezo Buzzer | 알람 사운드 출력 |
| LED + 220Ω 저항 | 알람 시 시각적 신호 |
| 조도센서(LDR) + 10kΩ 저항 | 주변 밝기 측정 |
| USB 케이블 | Arduino ↔ PC 연결 |
| 스마트폰 | App Inventor 앱 실행 |

---

## 🔌 3. 회로 연결도

### 📍 Piezo 부저  
- + → D8  
- - → GND  

### 📍 LED  
- 긴 다리(+) → D7  
- 짧은 다리(-) → 220Ω 저항 → GND  

### 📍 조도센서(LDR)  
```
(LDR 한쪽) ---- A0 입력  
(LDR 다른쪽) ---- 5V  
A0 ~ GND 사이 10kΩ 저항 연결 (분압 회로)
```

LDR 구성:
```
5V --- LDR --- A0 --- 10kΩ --- GND
```

---

## 🧠 4. 시스템 동작 방식
1️⃣ 스마트폰에서 알람 시간(HH:MM)을 입력 후 전송  
2️⃣ Processing 서버가 알람 시간을 메모리에 저장  
3️⃣ Processing이 PC 시간을 1초 간격으로 체크  
4️⃣ 설정된 시간과 같아지면 Arduino에 `"ALARM"` 송신  
5️⃣ Arduino:  
- Piezo 소리 출력  
- LED 점등  
- 조도센서 값 출력  

---

## 🧩 5. Arduino 코드 (조도센서 + LED + Piezo 포함)

```cpp
int buzzer = 8;
int led = 7;
int ldrPin = A0;

void setup() {
  Serial.begin(9600);
  pinMode(buzzer, OUTPUT);
  pinMode(led, OUTPUT);
}

void loop() {

  // 조도센서값 읽기 (Processing 또는 디버깅용)
  int lightValue = analogRead(ldrPin);
  Serial.println(lightValue);
  delay(100);

  // 알람 명령 처리
  if (Serial.available()) {
    String msg = Serial.readStringUntil('\n');
    msg.trim();

    if (msg == "ALARM") {
      digitalWrite(led, HIGH);

      for (int i = 0; i < 10; i++) {
        tone(buzzer, 1000, 400);
        delay(500);
      }

      noTone(buzzer);
      digitalWrite(led, LOW);
    }
  }
}
```

---

## 💻 6. Processing 코드 (HTTP 서버 + 시간확인 + Arduino 전송)

라이브러리 설치:  
`Processing → Sketch → Import Library → Add Library → "http.requests"`

```java
import http.requests.*;
import processing.serial.*;
import java.time.LocalTime;

Serial arduino;
String alarmTime = null;
boolean sent = false;

void setup() {
  println(Serial.list());
  arduino = new Serial(this, Serial.list()[0], 9600);
  startServer();
}

void draw() {
  if (alarmTime != null) {
    LocalTime now = LocalTime.now();
    String nowStr = String.format("%02d:%02d", now.getHour(), now.getMinute());

    if (nowStr.equals(alarmTime) && !sent) {
      println("Alarm activated → Sending to Arduino");
      arduino.write("ALARM\n");
      sent = true;
    }
  }
  delay(1000);
}

void startServer() {
  HTTPServer server = new HTTPServer(8080);
  server.addPostHandler("/alarm", (req, res) -> {
    alarmTime = req.getContent().trim();
    sent = false;
    println("New alarm set: " + alarmTime);
    res.send(200, "OK");
  });
  server.start();
}
```

---

## 📱 7. App Inventor 설정

### 필요한 컴포넌트
- TextBox (알람시간 입력)
- Button (전송)
- Web (HTTP POST)

### 블록 코드
```
when Button1.Click
    call Web1.PostText(
        URL = "http://<PC_IP>:8080/alarm",
        Text = TextBox1.Text
    )
```

---

## 📝 8. 시연 방법

1. Arduino 코드 업로드  
2. Processing 실행  
3. 스마트폰·PC 동일 Wi-Fi 연결  
4. App Inventor 앱에서 알람 시간 입력  
5. 설정한 시간 → Arduino에서 LED 점등 + 부저 알람

---

## 🎯 9. 주요 기능 요약

- 스마트폰에서 알람 시간 설정  
- Processing PC 시간 기반 체크  
- Arduino로 알람 명령 전송  
- Piezo + LED 알람 출력  
- 조도센서로 환경 밝기 측정  
- 와이파이/블루투스 모듈 없이 구현  
## 🙋 문의
궁금한 점이나 기능 추가가 필요하면 언제든지 문의하세요!
