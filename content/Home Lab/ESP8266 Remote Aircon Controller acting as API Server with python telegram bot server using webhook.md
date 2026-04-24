---
title: ESP8266 Remote Aircon
description: Creating a telegram aircon remote
draft: false
tags:
  - WiFi
  - wireless
  - ESP32
  - Microcontroller
  - Electronics
created: 12/13/2025 14:13
updated: 23/06/2026 21:06
---
# ESP8266 Code: 
```cpp
#include <Arduino.h>
#include <IRremoteESP8266.h>
#include <IRsend.h>
#include <ir_Mitsubishi.h>
#include <ESP8266WiFi.h>
#include <ESP8266WebServer.h>


const char* ssid = "myHomeNetwork";
const char* password = "password";

ESP8266WebServer server(80);


const uint16_t kIrLed = 4;  // ESP8266 GPIO pin to use. Recommended: 4 (D2).
IRMitsubishiAC ac(kIrLed);  // Set the GPIO used for sending messages.

void setup() {
  ac.begin();
  pinMode(LED_BUILTIN, OUTPUT);
  digitalWrite(LED_BUILTIN, false);  // False to turn LED light ON.
  Serial.begin(115200);
  WiFi.begin(ssid, password);

  Serial.println();
  Serial.println("Connecting to wifi now");
  while (WiFi.status() != WL_CONNECTED) 
  {
     delay(500);
     Serial.print("*");
  }
  
  Serial.println("");
  Serial.println("WiFi connection Successful");
  Serial.print("The IP Address of ESP8266 Module is: ");
  Serial.println(WiFi.localIP());// Print the IP address
  delay(200);

  ac.setFan(kMitsubishiAcFanAuto);
  ac.setMode(kMitsubishiAcCool);
  ac.setTemp(24);
  ac.setVane(kMitsubishiAcVaneAuto);

  server.on("/on", handle_AirOn);
  server.on("/off", handle_AirOff);
  server.onNotFound(handle_NotFound);
  
  server.begin();
  Serial.println("HTTP server started");
}

void loop() {
  server.handleClient();

  if (WiFi.status() == WL_CONNECTED) 
  {
    digitalWrite(LED_BUILTIN, true);
  } else {
    digitalWrite(LED_BUILTIN, false);
  }
}

void ac_on(){
  ac.on();
  ac.send();
}

void ac_off(){
  ac.off();
  ac.send();
}

void handle_NotFound(){
  server.send(404, "text/plain", "Not found");
}

void handle_AirOn() {
  digitalWrite(LED_BUILTIN, false);  // false is LED on
  ac_on();
  digitalWrite(LED_BUILTIN, true);
  server.send(200, "text/html", "Turning on"); 
}

void handle_AirOff() {
  digitalWrite(LED_BUILTIN, false);  // false is LED on
  ac_off();
  digitalWrite(LED_BUILTIN, true);
  server.send(200, "text/html", "Turning off"); 
}
```

## Resource: IR Remote
https://github.com/crankyoldgit/IRremoteESP8266

1. Click the _"Sketch"_ -> _"Include Library"_ -> _"Manage Libraries..."_ Menu items.
2. Enter `IRremoteESP8266` into the _"Filter your search..."_ top right search box.
3. Click on the IRremoteESP8266 result of the search.
4. Select the version you wish to install and click _"Install"_.

Resource code can be found here for Mitsubishi AC: https://github.com/crankyoldgit/IRremoteESP8266/blob/master/examples/TurnOnMitsubishiAC/TurnOnMitsubishiAC.ino

## Resource: Web API server reference: 
- https://lastminuteengineers.com/creating-esp8266-web-server-arduino-ide/

## Resource Wi-Fi
- https://avantmaker.com/references/esp32-arduino-core-index/esp32-arduino-core-wifi/esp32-wifi-library-station-class/esp32-wifi-library-wifi-status/

---
# Schematic:
![[ESP8266 Remote aircon schematic.png]]


# Choosing resistor for the Infrared (IR) LED
The IR LED is the medium that communicates with the aircon.

Choosing the right resistors determines the current it has and how bright it shines. the more current means the more range you have from the controller to the aircon. 

$$
R = \frac{Vs - Vf}{I}
$$
R = Resistor
Voltage source (Vs) = source or voltage output
Forward voltage of LED (Vf) = Voltage needed for the LED to operate
I = Targeted Current

So, my source is `5V`. the current I want is about `200mA`. The IR LED has a Vf of `1.8V`. plugging it into the formula:
$$
\begin{align}
R = \frac{5V - 1.8V}{200mA} \\
= 16 \ohm
\end{align}
$$
## Notes on my learning points for IR LED
### Why am I not using the GPIO from ESP8266?
by default, an ESP8266 GPIO only provides `3.3V` with a max current `Imax` value of `12mA`. not enough to drive my IR LED across the room.

Imax value was taken from the following datasheet: https://documentation.espressif.com/0a-esp8266ex_datasheet_en.pdf

Thus, using the transistor as a switch to drive LED at a higher current. Voltage source of the LED is now connected to the VIN or powered from the USB at `5V 1A` for USB 3.1.

### Getting Value of Forward voltage of LED (Vf):
I am using a 5mm Infrared LED with a wavelength of `λp=940nm`. 

Values such as forward voltage, Forward Current, and Peak Forward Current can be taken from the datasheet: https://www.mouser.com/datasheet/2/143/EAILP05RDDB1-708504.pdf

Forward voltage is the voltage "eaten/needed" by the LED.
Forward Current is the current that the LED can be use safety. `100mA` seems pretty high but it could be due to the LED being infrared. will update if it burns out.
Peak Forward Current is the max it can handle for a short period of time which is rated for `1A`.

### Downsides of Cheap IR modules: 
The module limits the IR to `3mA`. I had to be right in front of my air con just for it to work. 
![[cheap IR Module.png]]

It had a `1Kohm` and a `1.2Kohm` resistor on it. also the VCC pin does not seem to do anything for my particular module. it was the cheapest so I guess mine does not have modulation features.
### Resources: 
https://ohmslawcalculator.com/led-resistor-calculator

---
# The Transistors: 
The transistor is acting as a switch to drive a bigger current through the IR LED. 

## Overview:
Transistors have 3 pins. 🗑️Collector, 🎚️base, 💡emitter. 
I think of them in the following way:

🎚️Base is the trigger switch. turns it on and off.
🗑️collector is the start or input of the transistor. 
💡emitter emitter so it is the output of the transistor. 

I am using P2N2222A NPN. NPN/PNP determines the direction. transistors are like diodes where it controls the flow of current. for NPN is it from 🗑️collector to 💡emitter. with the 🎚️base accepting a raising current to activate the switch. PNP is the reverse. 

## Calculating the base resistor.
To choose the resister we first need to know the 🎚️base current needed.

The 🎚️base of the transistor need to have a certain current for it to fully saturate and allow current to flow from 🗑️collector to 💡emitter. 

The value needed for the 🎚️base depends on the current of the 🗑️collector and `the amplification factor`/`forward current transfer ratio (hFE)`. 

The hFE is provided in the transistors datasheet: https://www.onsemi.com/pdf/datasheet/p2n2222a-d.pdf

`P2N2222` has the hFE of `100` when the current of 🗑️collector is around `150 mAdc`. 

Current needed for the base can be calculated with the following formula.
$$
Ib = \frac{Ic}{\beta}
$$

Ib = 🎚️base current needed to switch on
Ic = 🗑️collectors current. 
beta = hFE

so in out example of `Ic = 200mA`, `hFE = 100`:
$$
\begin{align}
Ib = \frac{Ic}{\beta} \\
= \frac{200mA}{100} \\
= 2mA
\end{align}
$$
The 🎚️base needs about `2mA` to fully allow 200mA to flow from 🗑️collector to💡emitter.

From there we can calculate the resistor needed for the 🎚️base using ohm's law.
Vin is connected to to GPIO pin of ESP8266 at 3.3V: 
$$
\begin{align}
R_B = \frac{Vin}{Ib} \\
= \frac{3.3V}{2.5mA} \\ 
= 1320 \ohm \\ 
= 1.32K\ohm
\end{align}
$$
The value `2.5mA` was chosen instead of `2mA` to ensure that the transistor is able to fully saturate by surpassing the required `2mA` for the 🎚️base. 

### Resources:
https://www.learningaboutelectronics.com/Articles/How-to-calculate-beta-of-a-transistor
https://www.watelectronics.com/choosing-base-resistance-for-transistors/

---
# Python Telegram Bot API Server using webhook
Disclaimer. this is a quick and dirty code, so modify it as you please.
```python
#modified code from https://github.com/eternnoir/pyTelegramBotAPI/blob/master/examples/webhook_examples/webhook_fastapi_echo_bot.py

import logging
import fastapi
import uvicorn
import telebot
import requests

API_TOKEN = 'aaaa-bbbb-cccc'

WEBHOOK_HOST = 'my.website.com'
WEBHOOK_PORT = 8443  # 443, 80, 88 or 8443 (port need to be 'open')
WEBHOOK_LISTEN = '0.0.0.0'  # In some VPS you may need to put here the IP addr

WEBHOOK_SSL_CERT = './webhook_cert.pem'  # Path to the ssl certificate
WEBHOOK_SSL_PRIV = './webhook_pkey.pem'  # Path to the ssl private key

# Quick'n'dirty SSL certificate generation:
#
# openssl genrsa -out webhook_pkey.pem 2048
# openssl req -new -x509 -days 3650 -key webhook_pkey.pem -out webhook_cert.pem
#
# When asked for "Common Name (e.g. server FQDN or YOUR name)" you should reply
# with the same value in you put in WEBHOOK_HOST

WEBHOOK_URL_BASE = "https://{}:{}".format(WEBHOOK_HOST, WEBHOOK_PORT)
WEBHOOK_URL_PATH = "/{}/".format(API_TOKEN)

logger = telebot.logger
telebot.logger.setLevel(logging.INFO)
bot = telebot.TeleBot(API_TOKEN)
app = fastapi.FastAPI(docs=None, redoc_url=None)

@app.post(f'/{API_TOKEN}/')
def process_webhook(update: dict):
    """
    Process webhook calls
    """
    if update:
        update = telebot.types.Update.de_json(update)
        bot.process_new_updates([update])
    else:
        return


@bot.message_handler(func=lambda message: True, content_types=['text'])
def echo_message(message):
    """
    Handle all other messages
    """
    print("User: ", message.from_user.id)
    print("Message: ", message.text)
    if message.from_user.id == 123456789:
        if message.text.lower() == "airon":
            response = requests.get("http://192.168.1.2/on")
            if response.status_code == 200:
                print("Turning on")
                bot.reply_to(message, "Turning aircon ON Now!!!")
            else:
                bot.reply_to(message, "Error!!!")

        if message.text.lower() == 'airoff':
            response = requests.get("http://192.168.1.2/off")
            if response.status_code == 200:
                print("Turning off")
                bot.reply_to(message, "Turning aircon OFF Now!!!")
            else:
                bot.reply_to(message, "Error!!!")
    print()

# Remove webhook, it fails sometimes the set if there is a previous webhook
bot.remove_webhook()

# Set webhook
bot.set_webhook(
    url=WEBHOOK_URL_BASE + WEBHOOK_URL_PATH,
    certificate=open(WEBHOOK_SSL_CERT, 'r')
)

uvicorn.run(
    app,
    host=WEBHOOK_LISTEN,
    port=WEBHOOK_PORT,
    ssl_certfile=WEBHOOK_SSL_CERT,
    ssl_keyfile=WEBHOOK_SSL_PRIV
)
```

As stated in the comments. try using the commands provided to to generate the certs. my initial attempt at using certbot did not work.
```bash
openssl genrsa -out webhook_pkey.pem 2048
openssl req -new -x509 -days 3650 -key webhook_pkey.pem -out webhook_cert.pem
```
Enter the appropriate Fully Qualified Domain Name (FQDN).

## Troubleshooting API:
using the following commands can indicate some possible reasons why your telegram webhook is not working.
```bash
curl https://api.telegram.org/bot$BOT_TOKEN/getWebhookInfo
```

# Setting up port forwarding and firewall rules notes:
Inbound: IP address range to allow: `149.154.160.0/20` and `91.108.4.0/22`.
outbound: `api.telegram.org` 

If you are using Pfsense. allow outbound by the FQDN. In the firewall web UI: `firewall -> alias -> IP -> add`:
type = `host`
ip or FQDN = `api.telegram.org`

add the appropriate NAT and firewall rule and you should be good.

# Resource:
https://github.com/eternnoir/pyTelegramBotAPI/blob/master/examples/webhook_examples/webhook_fastapi_echo_bot.py

https://core.telegram.org/bots/webhooks