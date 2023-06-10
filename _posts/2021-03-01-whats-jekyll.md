---
layout: post
title: PID　搖控小車
category: [Example]
tags: [jekyll]
---


## PID遙控小車


### 系統方塊圖
![](https://github.com/shuyuan9215/MCU-course-project/blob/main/images/pid%E6%96%B9%E5%A1%8A%E5%9C%96.png?raw=true)

### ESP32_Robocar_PID程式碼
![](https://github.com/hjgyjg123/MCU-project/blob/main/images/ESP32_PID%E7%A8%8B%E5%BC%8F%E7%A2%BC1.jpg?raw=true)
![](https://github.com/hjgyjg123/MCU-project/blob/main/images/ESP32_PID%E7%A8%8B%E5%BC%8F%E7%A2%BC2.jpg?raw=true)
![](https://github.com/hjgyjg123/MCU-project/blob/main/images/ESP32_PID%E7%A8%8B%E5%BC%8F%E7%A2%BC3.jpg?raw=true)
![](https://github.com/hjgyjg123/MCU-project/blob/main/images/ESP32_PID%E7%A8%8B%E5%BC%8F%E7%A2%BC4.jpg?raw=true)

### ESP32_Robocar_PID程式功能
1. 先include必要的函式庫，Wire.h、WebServer.h、ESP32MotorControl.h 和 MPU6050_6Axis_MotionApps20.h
2. 定義變數和常數，各種控制機器人車的變量、PID調整、WiFi設定和馬達控制腳位等等
3. 程式碼建立了一個HTTP伺服器，用於透過網頁指令遙控機器車
4. setup() 函式初始化了MPU6050感應器，設定了DMP來追蹤方向，設定陀螺儀偏移量，並初始化HTTP伺服器
5. loop() 函式不斷檢查傳入的HTTP請求並加以處理，從MPU6050感應器讀取當前方向，根據當前方向與目標方向之間的差異計算PID馬達功率，並相應地調整馬達功率，以便調整方向
6. 程式碼定義了不同的函式（cmd1()、cmd2()、cmd3()、cmd4()、cmd5()、handleRoot()）來處理不同方向的HTTP請求，以此來控制機器人車的運動（向前、向後、向右、向左、停止）

### 實作影片
<iframe width="329" height="586" src="https://www.youtube.com/embed/1DGSCxA9pBQ" title="藍芽遙控車" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

<br>
<br>

*This site was last updated {{ site.time | date: "%B %d, %Y" }}.*



