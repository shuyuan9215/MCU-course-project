---
layout: post
title: ESP32 webserver & client
author: [Shuyuan Lee]
category: [Lecture]
tags: [jekyll, ai]
---



---
## 實作照片
![](https://github.com/shuyuan9215/MCU-course-project/blob/main/images/S__46858246.jpg?raw=true)

--

## 方塊圖

![](https://github.com/shuyuan9215/MCU-course-project/blob/main/images/iot.png?raw=true)


## 程式碼 ESP32_Webserver_Ngrok.ino
"""
//
// Access ESP32 Web server from anywhere in the world
//
// ESP32 run this webserver example
// PC run CMD.exe (Command Prompt)
// C:\Users\rkuo> ngrok tcp 192.168.1.12:80 --authtoken Your_Ngrok_Authtoken

#include <WiFi.h>

const char* ssid = "your_ssid";  
const char* password = "your_password";

WiFiServer server(80);
String header;


void setup() {
  Serial.begin(115200);
  
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
  server.begin();
}

void loop(){
  WiFiClient client = server.available();   // Listen for incoming clients
  
    if (client) {                             // If a new client connects,
    Serial.println("New Client.");          // print a message out in the serial port
    String currentLine = "";                // make a String to hold incoming data from the client
    while (client.connected()) 
    {            // loop while the client's connected
      if (client.available()) {             // if there's bytes to read from the client,
        char c = client.read();             // read a byte, then
        Serial.write(c);                    // print it out the serial monitor
        header += c;
        if (c == '\n') {                    // if the byte is a newline character
          // if the current line is blank, you got two newline characters in a row.
          // that's the end of the client HTTP request, so send a response:
          if (currentLine.length() == 0) {
            client.println("HTTP/1.1 200 OK");
            client.println("Content-type:text/html");
            client.println("Connection: close");
            client.println();
            client.println("<!DOCTYPE html>\n");
            client.println("<html>\n");
            client.println("<body>\n");
            client.println("<center>\n");
            client.println("<h1 style=\"color:blue;\">ESP32 webserver</h1>\n");
            client.println("<h2 style=\"color:green;\">Hello World Web Sever</h2>\n");
            client.println("<h2 style=\"color:blue;\">Password protected Web server</h2>\n");
            client.println("</center>\n");
            client.println("</body>\n");
            client.println("</html>");
            break;                       
          } 
          else { // if you got a newline, then clear currentLine
            currentLine = "";
          }
        } else if (c != '\r') {  // if you got anything else but a carriage return character,
          currentLine += c;      // add it to the end of the currentLine
        }
      }
    }
    // Clear the header variable
    header = "";
    // Close the connection
    client.stop();
    Serial.println("Client disconnected.");
    Serial.println("");
  }
}
"""
---
### 程式碼 ESP32_Webclient_IoT_HTU21DF.ino
"""

// Webclient to read HTU21DF and send data to Webserver
#include <WiFi.h>
#include <WiFiClient.h>
#include <HTTPClient.h>
#include <Wire.h>
#include <Adafruit_HTU21DF.h>

// Connect Vin to 3-5VDC
// Connect GND to ground
// Connect SCL to I2C clock pin (A5 on UNO, D1 on NodeMCU)
// Connect SDA to I2C data pin (A4 on UNO, D2 on NodeMCU)

Adafruit_HTU21DF htu = Adafruit_HTU21DF();

const char* ssid     = "your_ssid";
const char* password = "your_password";
String      webserverIP = "http://192.168.1.12"; // Your Webserver IP address

void setup() {
  Serial.begin(115200);
  Serial.println("HTU21D-F test");  
  if (!htu.begin()) {
    Serial.println("Couldn't find HTU21DF sensor!");
    while (1);
  }
  
  // We start by connecting to a WiFi network
  Serial.println();
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
  
  WiFi.begin(ssid, password);
  
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi connected");  
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void loop() {
  delay(5000);

  float temp  = htu.readTemperature();
  float humid = htu.readHumidity();
  Serial.print(temp);
  Serial.print(" ");
  Serial.print(humid);
  Serial.println();
  
  String url = webserverIP + "/htu21?";
  url += "T=";
  url += String(temp);
  url += "&H=";
  url += String(humid);
  
  if(WiFi.status()== WL_CONNECTED){   //Check WiFi connection status

     WiFiClient client;

     HTTPClient http;    //Declare object of class HTTPClient
 
     http.begin(client, url);    //Specify request destination
     http.addHeader("Content-Type", "text/plain");  //Specify content-type header
 
     int httpCode = http.POST("Message from ESP32");   //Send the request
     String payload = http.getString();                //Get the response payload
 
     Serial.println(httpCode);   //Print HTTP return code
     Serial.println(payload);    //Print request response payload

     http.end();  //Close connection
 
   }else{
      Serial.println("Error in WiFi connection");   
   }
  
  Serial.println();
  Serial.println("closing connection. going to sleep...");
  // go to deepsleep for 1 minutes
  //system_deep_sleep_set_option(0);
  //system_deep_sleep(1 * 60 * 1000000);
  delay(1*60*1000);
}
"""
<br>
<br>

*This site was last updated {{ site.time | date: "%B %d, %Y" }}.*


