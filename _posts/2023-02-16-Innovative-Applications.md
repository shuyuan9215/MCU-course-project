---
layout: post
title: 藍牙遙控機器人實作
author: [Lee Shu Yuan]
category: [homework]
tags: [jekyll, ai]
---

This project is to implement a bluetooth remote controlled robotcar.

---
## 藍牙遙控機器人
![](https://github.com/rkuo2023/MCU-project/blob/main/images/ESP32_RoboCar.jpg?raw=true)


### 應用功能說明
1. Bluetooth remote control App 
2. two-wheel robocar

### 設計考量與相關技術
**系統設計考量：**<br>
1. 操作方式:藍牙遙控手機App
2. 移動方式:兩輪 
3. 供電方式:鋰電池 3.7V x2
4. 聯網方式:藍牙

**所需相關技術：**
1. MIT App Inventor 2 手機程式設計 
2. Arduino程式設計

**所需相關套件:**
![](https://image.ruten.com.tw/g2/8/d4/16/21440347657238_872.jpg)

**系統方塊圖:**
![](https://github.com/shuyuan9215/MCU-course-project/blob/main/images/%E6%96%B0%E5%A2%9E%E5%B0%91%E9%87%8F%E5%85%A7%E6%96%87.jpg?raw=true)

### 手機藍牙遙控, 或WebUI 遙控(利用手機熱點WiFi連線)
<iframe width="320" height="560" src="https://www.youtube.com/embed/f2BniY4nAnU" title="手機藍牙遙控, 或WebUI 遙控(利用手機熱點WiFi連線)1" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

<iframe width="320" height="560" src="https://www.youtube.com/embed/3FaQAwD77h4" title="手機藍牙遙控, 或WebUI 遙控(利用手機熱點WiFi連線)2" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

### 程式碼
1. // PWM to DRV8833 dual H-bridge motor driver, PWM freq. = 1000 Hz
// ESP32 Webserver to receive commands to control RoboCar

#include <WiFi.h>
#include <WebServer.h>
#include <ESP32MotorControl.h> 

// DRV8833 pin connection
#define IN1pin 16  
#define IN2pin 17 
#define IN3pin 18 
#define IN4pin 19

#define motorR 0
#define motorL 1
#define FULLSPEED 100
#define HALFSPEED 50

ESP32MotorControl motor;


/* Set these to your desired credentials. */
const char *ssid = "Your_SSID";
const char *password = "Your_Password";

WebServer server(80); // Set web server port number to 80

const String HTTP_PAGE_HEAD = "<!DOCTYPE html><html lang=\"en\"><head><meta name=\"viewport\" content=\"width=device-width, initial-scale=1, user-scalable=no\"/><title>{v}</title>";
const String HTTP_PAGE_STYLE = "<style>.c{text-align: center;} div,input{padding:5px;font-size:1em;}  input{width:90%;}  body{text-align: center;font-family:verdana;} button{border:0;border-radius:0.6rem;background-color:#1fb3ec;color:#fdd;line-height:2.4rem;font-size:1.2rem;width:100%;} .q{float: right;width: 64px;text-align: right;} .button1 {background-color: #4CAF50;} .button2 {background-color: #008CBA;} .button3 {background-color: #f44336;} .button4 {background-color: #e7e7e7; color: black;} .button5 {background-color: #555555;} </style>";
const String HTTP_PAGE_SCRIPT = "<script>function c(l){document.getElementById('s').value=l.innerText||l.textContent;document.getElementById('p').focus();}</script>";
const String HTTP_PAGE_BODY= "</head><body><div style='text-align:left;display:inline-block;min-width:260px;'>";
const String HTTP_PAGE_FORM = "<form action=\"/cmd1\" method=\"get\"><button class=\"button1\">Forward</button></form></br><form action=\"/cmd2\" method=\"get\"><button class=\"button2\">Backward</button></form></br><form action=\"/cmd3\" method=\"get\"><button class=\"button3\">Right</button></form></br><form action=\"/cmd4\" method=\"get\"><button class=\"button4\">Left</button></form></br><form action=\"/cmd5\" method=\"get\"><button class=\"button5\">Stop</button></form></br></div>";
const String HTTP_WEBPAGE = HTTP_PAGE_HEAD + HTTP_PAGE_STYLE + HTTP_PAGE_SCRIPT + HTTP_PAGE_BODY + HTTP_PAGE_FORM;
const String HTTP_PAGE_END = "</div></body></html>";

**這是一段ESP32控制機器人小車的程式碼，它還有一個Web伺服器以接收控制命令。
程式碼的第一個部分包含了需要引用的庫。以下是每個庫的作用：
•	WiFi.h - 允許ESP32使用WiFi網路連線。
•	WebServer.h - 用於建立ESP32的Web伺服器，可以用來接收和回應HTTP請求。
•	ESP32MotorControl.h - 用於控制馬達的庫。
程式碼的下一個部分是將腳位與常量關聯起來，並初始化ESP32MotorControl物件以控制兩個馬達的方向和速度。
程式碼的下一部分包含WiFi的SSID和密碼。這些是用於連接ESP32到WiFi網路的憑據。
接下來是Web伺服器的初始化，它使用80作為埠號。
HTTP_PAGE_HEAD，HTTP_PAGE_STYLE，HTTP_PAGE_SCRIPT，HTTP_PAGE_BODY，HTTP_PAGE_FORM和HTTP_PAGE_END是網頁HTML的各種部分，最後合併起來形成完整的網頁，以用於Web伺服器回應HTTP請求。
程式碼的主要功能是建立Web伺服器和回應HTTP請求。在Web伺服器上設置了5個按鈕，分別代表向前、向後、向右、向左和停止馬達。當使用者按下其中一個按鈕時，Web伺服器會發送HTTP GET請求到ESP32上，然後ESP32會根據所接收到的命令來控制馬達的運動。
總體來說，這個程式碼實現了一個基本的ESP32控制機器人小車的系統，並提供了一個簡單的Web界面來控制它。**


// Current time
unsigned long currentTime = millis();
// Previous time
unsigned long previousTime = 0; 
// Define timeout time in milliseconds (example: 2000ms = 2s)
const long timeoutTime = 2000;

int speed = HALFSPEED;

void handleRoot() {
  String s  = HTTP_WEBPAGE; 
  s += HTTP_PAGE_END;
  server.send(200, "text/html", s);
}

void cmd1() {
  String s  = HTTP_WEBPAGE; 
  s += HTTP_PAGE_END;  
  server.send(200, "text/html", s);
  motor.motorForward(motorR, speed);  
  motor.motorForward(motorL, speed);
  Serial.println("Move Forward");   
}

void cmd2() {
  String s  = HTTP_WEBPAGE; 
  s += HTTP_PAGE_END;  
  server.send(200, "text/html", s);
  motor.motorReverse(motorR, speed);
  motor.motorReverse(motorL, speed);
  Serial.println("Move Backward");     
}

void cmd3() {
  String s  = HTTP_WEBPAGE; 
  s += HTTP_PAGE_END;  
  server.send(200, "text/html", s);
  motor.motorReverse(motorR, speed);  
  motor.motorForward(motorL, speed);
  Serial.println("Turn Right");    
}

void cmd4() {
  String s  = HTTP_WEBPAGE; 
  s += HTTP_PAGE_END;  
  server.send(200, "text/html", s);
  motor.motorForward(motorR, speed);
  motor.motorReverse(motorL, speed);
  Serial.println("Turn Left"); 
}

void cmd5() {
  String s  = HTTP_WEBPAGE; 
  s += HTTP_PAGE_END;  
  server.send(200, "text/html", s);
  motor.motorStop(motorR);
  motor.motorStop(motorL);
  Serial.println("Motor Stop");
}

**這段程式碼定義了幾個函數，用於處理來自網頁伺服器的不同HTTP請求。 handleRoot（） 函數會產生一個HTML網頁並將其作為回應發送。其他幾個函數（cmd1，cmd2，cmd3，cmd4和cmd5）分別處理前進、後退、右轉、左轉和停止的命令。在這些函數中，它們會向馬達驅動器發送特定的指令，以控制機器人移動方向和速度，並通過串口打印調試信息。**


void setup() {
  Serial.begin(115200);
  Serial.println("Motor Pins assigned...");
  motor.attachMotors(IN1pin, IN2pin, IN3pin, IN4pin);
  
  // Connect to Wi-Fi network with SSID and password
  Serial.print("Connecting to ");
  Serial.println(ssid);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  // Print local IP address and start web server
  Serial.println("");
  Serial.println("WiFi connected.");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());

  server.on("/", handleRoot);
  server.on("/cmd1", cmd1);
  server.on("/cmd2", cmd2);
  server.on("/cmd3", cmd3);
  server.on("/cmd4", cmd4);  
  server.on("/cmd5", cmd5);  

  Serial.println("HTTP server started");
  server.begin();

  motor.motorStop(motorR);
  motor.motorStop(motorL);
}

void loop() {
  server.handleClient();
}


**這個程式碼的功能是使用ESP8266控制兩個直流馬達，並透過網頁介面控制其移動方向和停止。程式碼使用了ESP8266庫和WiFi庫來設置Wi-Fi網絡，建立HTTP伺服器和處理客戶端請求。在setup()函數中，首先進行了串口和馬達的初始化，然後進行Wi-Fi網絡的連接。連接成功後，設置HTTP伺服器的路由，並啟動HTTP伺服器。最後，馬達被停止。在loop()函數中，持續地處理客戶端請求。**

<br>
<br>

*This site was last updated {{ site.time | date: "%B %d, %Y" }}.*


