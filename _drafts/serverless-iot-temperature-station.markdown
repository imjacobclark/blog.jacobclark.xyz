---
layout: post
title: Temperature Visualisation with IoT, Serverless & Machine Learning
---

When learning something new, having genuine data to build something useful with is usually the hardest part. So I figured I'd build a full stack project, from microcontroller through to machine learning algorithms to learn bits of AWS I haven't touched in the past.

I bought a bunch of electronics hardware from Aliexpress to build an IoT WiFi enabled Temperature Station. In this blog post, I'm going to outline how to build your own IoT Temperature Station and visualise/analise that data in the cloud.

Specifically, this post will outline how to collect temperature and humidity data every 10 seconds and persist it into the cloud, using machine learning to detect anomalies in the data such as a sudden drop in temperature or temperature not rising when it is expected to do so, useful to detect a window that has been left open or the heating being accidentially turned off.

![An ESP8266 with a display showing the current temperature, humidity and time](/images/serverless-iot-temperature-station/iot-temperature-station.jpg)

I'm going to attempt to make this post as accessible to all as possible (just skip over the bits you know), however I assume you have _some_ familiarity with programming in this post, we're going to be using a wide variety of technologies, so I will inevitably skip explaining something for brevity.

![A temperature dashboard](/images/serverless-iot-temperature-station/temperature-dashboard.png)

## High level architecture overview

This is what we're building end to end!

![High level architecture overview](/images/serverless-iot-temperature-station/iotx-temperature-dashboard-architecture.png)

## Serverless stack

We will use [Amazon Web Services (AWS)](http://aws.amazon.com/) for compute. If you don't have an AWS account, set one up now.

* AWS Serverless Architecture Model
* API Gateway
* AWS Lambda (Node.js, JavaScript)
* AWS CloudWatch
* AWS TimeStream
* AWS SakeMaker with a Random Cut Forrest algorithm, Jupyter Notebook, Matplotlib, Numpy & Pandas

This architecture will cost around $1 per day to run on Amazon Web Services.

## Hardware required

* [ESP8266 development board](https://www.aliexpress.com/item/32665100123.html?spm=a2g0o.productlist.0.0.5372682dUP4eIP&algo_pvid=963d4034-9019-4758-bf30-8b4e9656a0d7&algo_expid=963d4034-9019-4758-bf30-8b4e9656a0d7-0&btsid=0b0a119a16164134093905636e6c30&ws_ab_test=searchweb0_0,searchweb201602_,searchweb201603_) (approx £2.85 including shipping to UK)
* [AM2302 Digital Temperature Sensor](https://www.aliexpress.com/item/32517645825.html?spm=a2g0s.9042311.0.0.6c294c4dRnPTSx) (approx £2.47 including shipping to the UK)
* [0.96 OLED IIC Display SSD1306](https://www.aliexpress.com/item/32830523451.html?spm=a2g0s.9042311.0.0.6c294c4dRnPTSx) (approx £2.67 including shipping to the UK)
* [Male-Male Jumper Wires](https://www.aliexpress.com/item/4000203371860.html?spm=a2g0o.productlist.0.0.63054a6437RVZE&algo_pvid=e41d8564-1946-49a2-98e7-aa3e1d6945eb&algo_expid=e41d8564-1946-49a2-98e7-aa3e1d6945eb-0&btsid=2100bdde16164137918016191e3945&ws_ab_test=searchweb0_0,searchweb201602_,searchweb201603_) (approx £1.63 inclusing shipping to the UK)

Total cost of hardware should be approx £9.62 including shipping.

Note: You do not need a pull-up resistor for the AM2302, it has a built in pull-up resistor.

### Connection guide

Plug your ESP8266 into a breadboard along with your OLED, you'll need **4** jumper wires in total along with a micro-usb cable capable of data transfer, ensure the cable isn't just a power cable, we will use this cable to program the ESP8266.

**Ground, G or GND** is a reference point for all other electronic parts, it is essentially the "exit" flow for the electricity, otherwise known as 0 vaults. It is the negative part of the circuit. 

**3v** is the positive part of the circuit, this is where the electricity flows from and what provides our power. On the ESP8266 it is rated at 3 volts. If you go on to building more elaborate circuits with the ESP8266, it is worth noting **3v** is not always sufficient to power certain components.

#### AM2302

Plug the red wire on the AM2302 into a **3v** pin and the black wire into a ground pin (abbreviated **G** or **GND**). The yellow pin should be plugged into D5.

#### OLED SSD1306

Plug a jumper from a ground pin (abbreviated **G** or **GND**) into GND, another jumper from **3v** into **VCC** (voltage common collector, otherwise known as the positive pin in our circuit), another jumper from **D1** to **SCL** (the clock) and finally another jumper from **D2** into **SDA** (the data pin).

__Clocks__ are used to coordinate actions within the circuit. 

### Programming the ESP8266

We program the ESP8266 using C++ and the Arduino IDE. The Arduino IDE is a mostly open souece, community led project with hundreds of libraries, drivers and board software, it will enable us to program our microcontroller rapidly.

As this project involves connecting to a wireless network, making https requests, reading and writing data to external peripherals, we will use Visual Studio Code with the Arduino extension installed to enable us to write our program in a modular way so it is easy to extend (if we added more sensors, for example).

Download and install the following software: 

* [Arduino IDE](https://www.arduino.cc/en/software)
* [Visual Studio Code](https://code.visualstudio.com/Download)
* [Visual Studio Code extension for Arduino](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.vscode-arduino)

Next you'll need to install the correct drivers for the ESP8266, SiLabs have worked well for me on OS X. [Download them here](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers).

Finally, we need to tell the Arduino IDE how to program the ESP8266. 

In the Arduino Menu find Preferences, there will be a field named "Additional board manager URLs", paste `http://arduino.esp8266.com/stable/package_esp8266com_index.json` into that field and hit OK. This configures the Arduino IDE with a pointer to the instructions on how to program the ESP8266, by default the Arduino IDE only comes with the instructions on how to program Arduino Microcontrollers.

Next, find `Tools -> Board -> Boards Manager`, search for ESP8266, a result named "esp8266" by "ESP8266 Community" should be returned, select the latest version and click install then close. This installs the instructions into the Arduino IDE and gives us the ability to program the ESP8266.

We can now burn a Hello World program onto our ESP8266. Firstly, lets tell the Arduino IDE we want to program the ESP8266. `Tools -> Board -> ESP8266 Boards -> NodeMCU 1.0`.

Plug your ESP8266 into a USB port on your computer, again, ensure the Micro USB you're using is capable of data transfer as well as power. 

Select the correct serial port for your ESP8266 in `Tools -> Port -> cu.wchusbserial1420`. The serial port is most likely named `cu.wchusbserial1420`, however, it may be different on your system, give all the options a try until you get a successful upload if required.

Finally, we can compile and upload a program to our ESP8266. 

```cpp
void setup() {
  Serial.begin(9600);
}

void loop() {
  Serial.println("Hello World");
}
```

At the top of the IDE, there is a tick and a right arrow. The tick is verify, this checks the program for errors, the right arrow is upload, this uploads the code onto the ESP8266.

Hit the right arrow, once you get a success message, you can then open the serial monitor `Tools -> Serial Monitor` and should see "Hello World" printed to the screen many times.

![Hello World printed to the serial monitor](/images/serverless-iot-temperature-station/hello-world-serial.png)

Congratulations, you've successfully programmed your ESP8266.

#### Setup

The setup function runs only once when the ESP8266 boots. You should perform operations like connecting to WiFi or loading the file system here.

#### Loop

The loop function is called approximately once per second, you should put your main body of code here, such as reading temperature at regular intervals or informing something that the device is still online.

## Reading the temperature

Before we even connect the ESP8266 to the internet, we're going to get the temperature and humidity from the AM2302 sensor and output it to the serial monitor!

Like with most forms of software development, we have some dependencies we need to install into Arduino in order to communicate with our temperature sensor correctly. From time to time we will need to navigate to `Tools -> Manage Libraries` within the Arduino IDE so we can interface with external components. 

Head over to `Tools -> Manage Libraries` and search for `DHTesp`, select the latest version and hit install.

![Install DHTesp](/images/serverless-iot-temperature-station/install-dhtesp.png)

This library auto detects which temperature sensor we're using and then reads the digital input as a series of bits from the thermometer, then subsequently converting it into a useful temperature (celcius or fahrenheit) and humidity reading (percentage).

Open an empty folder in Visual Studio Code. Create `src` folder and within it create another `app` folder. 

Open a new file named `app.ino` and create a bare bones program:

```cpp
void setup() {
}

void loop() {
}
```

At this point the Arduino extension should be installed in your VSCode, next you'll need to configure it with your board, in the status bar the bottom, you should have the ability to select a board and a port. Select `NodeMCU 1.0` and the `cu.wchusbserial1420` serial port you used last time.

In app.ino in VSCode, you should have a verify and upload button at the top right of the editor, next to the "Split Editor" option. Hit upload and watch your program find its way to the ESP8266. This is how we will upload code to our ESP8266 from here on out, so you can go ahead and close the Arduino IDE.

![Verify and Compile buttons in VSCode](/images/serverless-iot-temperature-station/vscode-upload-buttons.png)

Next, we can actually write the code that interfaces with the sensor, when writing code, we have loads of options around how we structure and lay it out, most Arduino "sketches" reside in a single file, they can get quite large though and difficult to maintain. We're going to split our code out into classes, using `app.ino` to orchestrate these classes into objects. This will make it super easy to extend and add more functionality later.

In the `src/app` folder you created earlier, create two new files, `AM2302Manager.cpp` and `AM2302Manager.h`. 

Open `AM2302Manager.h`, files ending in `.h` are header files, these files contain class definitions and functions, they allow us to seperate the what from the how, we're going to define a class `AM2302Manager` with two member functions, `get_temperature()` and `get_humidity()`. These functions both return `floats` which we outline in the header file too. Within the header file we will include the `DHTesp` library we installed earlier.

```cpp
#include "DHTesp.h"

class AM2302Manager {
  public:
      float get_temperature();
      float get_humidity();
};

extern AM2302Manager am2303Manager;
```