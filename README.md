# WiSafe2-to-HA Bridge readme

## Project Background
Like some others, I have some expired Google Nest Protects, which I wanted to replace with a cheaper / more-scalable alternative. But above all, I wanted to realise a trustworthy fire/CO alarm system, which integrates with HomeAssistant.

In pursuit of this, I initially purchased the FireAngel Pro Connected Gateway, but found it to be a dreadfully unreliable piece of kit. In the end, I lost all faith in the Gateway and instead, I decided to build my own bridge from a donor radio module.

## Rant about the FireAngel Pro Connected Gateway
As part of my testing, I removed all the alarms at once, and it didn't notice! 

Pressing 'test' in the App still reported all alarms 'online'. 
In fact, after leaving the gateway completely unplugged for almost a year, today the FireAngel app still says one alarm is online, and shows some obnoxious message "Monitoring the risk to your home in real time using complex algorithms… Everything is OK!?!”.

![App after 9 months offline](https://github.com/C19HOP/WiSafe2-to-HomeAssistant-Bridge/blob/master/FireAngelProConnectedGateway/FireAngelApp.png)

So after being bitterly disappointed with the FireAngel Gateway (and not minding if I destroyed it), my next step was to attach it to a debugger, to see exactly what it was doing (with the mindset of intercepting its comms to create a local HA integration).
I could see some JSON state changes being sent to AWS. But I found the messages were very slow to report, and even worse was that it stopped updating locally if the internet connection went down. 


If you want to attach a serial debug logger to your Gateway, then you can use a 3.3v USB-to-serial adapter, and attach GND-to-GND and TX-to-RX
 
![Gateway Debug Pins](https://github.com/C19HOP/WiSafe2-to-HomeAssistant-Bridge/blob/master/FireAngelProConnectedGateway/Gateway_debug_02.jpg)

And if you're interested in seeing the debug log I trapped from my gateway, then you can check it out here:

[GitHub - FireAngel Pro Connected Gateway debug investigation](https://github.com/C19HOP/WiSafe2-to-HomeAssistant-Bridge/tree/master/FireAngelProConnectedGateway)

## OK. The FireAngel Gateway is bad. So what's next?

Thinking about how to capture the WiSafe2 data directly, without the FireAngel Connected Gateway, I considered using a generic 868MHz transceiver to intercept the comms. But I didn't find one for a good price. Plus, the data over-the-air is encrypted and and the specification is not documented for the public anyway.
So rather than trying to communicate with the WiSafe2 network from an unsupported radio, I looked at one of the radio modules from an alarm and noted that it uses SPI to communicate with the alarm board. For me, the path of least resistance was therefore to take a radio module as a donor and use it to build my own bridge. Letting the genuine radio module deal with the encryption and network pairing.

With the radio on the bench, I used PulseView to intercept and analyse the communication between the radio and an alarm. I reverse-engineered everything that I could figure out.
Eventually, I had all the information I needed to start listening to and taking back to the radio module.

If you're interested in the details of communication specification, then the link below shows what I recorded. Maybe you have some inside knowledge and can add more detail to it? Or would like to investigate the concept of using a generic 868MHz transceiver further...

[GitHub - WiSafeCommunicationAnalysis](https://github.com/C19HOP/WiSafe2-to-HomeAssistant-Bridge/tree/master/WiSafeCommunicationAnalysis)

![On the bench](https://github.com/C19HOP/WiSafe2-to-HomeAssistant-Bridge/blob/master/build/reverse-engineering%20wi-safe2%20radio%202.jpg)

With the the communication sussed out, I then built a driver for the radio, to allow us to communicate with it over USB via an Arduino Nano. 

![Prototype](https://github.com/C19HOP/WiSafe2-to-HomeAssistant-Bridge/blob/master/build/my%20gateway.jpg)


## Birth of the WiSafe2-to-HA Bridge (This Project)

![HA LoveLace](https://github.com/C19HOP/WiSafe2-to-HomeAssistant-Bridge/blob/master/HA/HA-lovelace.png)

At this point, I considered the safety and reliability of a DIY approach... As should you!
The primary function of the alarms is to work as designed locally, and the 'smart' connection is always just a 'nice-to-have'. 

For me, that's why I chose a solution which extends the functionality on a known brand, but does not modify the actual alarms in any way. But this solution is of course shared without any warranty or support. And I am not responsible if you damage anything, or if doesn't work the way you expected it to.
That said, with FireAngel's commercial solution being such a disaster, they have set the bar very low (last rant about the gateway, I promise).

With this WiSafe2-to-HA Bridge, if it's all configured properly in HomeAssistant, you'll have voice alerts in your home, telling you which room the emergency is in... Which is something FireAngel's solution doesn't offer. Yes; all the alarms beep. But they don't tell you where the problem originated from.

So if you're still reading, I'll explain how I used a WiSafe2 radio module, along with an Arduino Nano, to bridge my network of FireAngel WiSafe2 alarms into HomeAssistant.

This WiSafe2-to-HA Bridge should work with any WiSafe2 alarm. But it has only been tested with the devices I actually have, which are:
* FP2620W2
* FP1720W2
* WST-630
* W2-SVP-630
* W2-CO-10X

With this project, via Home Assistant, I am able to receive notifications when:
* An alarm is tested (including the type and result)
  * heat, smoke, CO 
  * OK / Not-OK
* An emergency event occurs (including the type of event)
  * Fire
  * CO
* An alarm is removed from / attached to its base
* An alarm's battery is running low
* The other alarms are complaining about a paired (but missing) device

I can also send data back to the network, including:
* Perform a Fire Test
* Perform a CO Test
* Silence an alarm (takes a moment to propergate through the network, just like pressing the actual silence button on a physical alarm)

And I can perform some basic configuration, like check and enable network pairing mode.
The bridge also produces a heart-beat, so we can be confident that it is communicating with HA.

I highly recommend that you configure your Home Assistant instance to use Google Home / Amazon Alexa for TTS & that you have the Mobile Phone Companion App, working over the internet for remote phone notifications.

![HA Notification](https://github.com/C19HOP/WiSafe2-to-HomeAssistant-Bridge/blob/master/HA/HA%20App%20notifications.png)
[![Video of my Google Home announcing an event](https://github.com/C19HOP/WiSafe2-to-HomeAssistant-Bridge/blob/master/HA/ha-fireangel-googlehome.png)](https://youtu.be/Ph-9fv3ursU)


## Shopping List
To build the WiSafe2-to-HA Bridge, you'll need:
* A WiSafe2 radio module
* An Arduino Nano
* PCB
* 4ch Level Converter (x2)
* 2.54mm jumper
* 2.54mm male header (2-pins)
* Enclosure


## Obtaining the parts

### Radio
The WiSafe2 radio modules can be obtained from:
* Any old alarm. Perhaps you can find one which has expired, or has gone faulty.
* The W2-SVP-630 strobe units can regularly be found for sale on ebay and are otherwise worthless 2nd-hand... I picked mine up for £5 :)
* The radio can be bought as a stand-alone module. But expect to over-pay if buying it new.

### The Arduino Nano
Just use a regular Arduino Nano (or clone). Make sure it is 5v @ 16MHz.

### PCB
Here's a link to the PCB I made for this:
* [JSON file (for loading into easy-EDA):](https://github.com/C19HOP/WiSafe2-to-HomeAssistant-Bridge/blob/master/PCB/WiSafe2HA-bridge-PCB.json)
* [Gerber files (for ordering direct from JLCPCB, etc):](https://github.com/C19HOP/WiSafe2-to-HomeAssistant-Bridge/blob/master/PCB/Gerber_EASYEDA-WiSafe2HA-bridge-PCB.zip)

You can load the JSON into easyEDA and order some PCBs using the built-in JLCPCB ordering utility. It cost me a couple of USD, plus delivery.
Or in case it's easier for you, you can use the Gerber files to place the order yourself. 

![PCB design](https://github.com/C19HOP/WiSafe2-to-HomeAssistant-Bridge/blob/master/PCB/pcb01.png)

### Level Converter
You need 2x 4ch level converters. Just search ebay for "4ch Logic Level Shifter Converter Module 5V to 3.3V"

### 2.54mm jumper
Rummage through your drawers. There's definitely one in there somewhere!

### 2.54mm male header
Search ebay for 2.54mm male header. You only need 2 pins for attaching the jumper to.
If this is a problem, you can always just bridge the contacts together with solder.

### Enclosure
Here's a link to the STL file for the enclosure I made: [Enclosure](https://github.com/C19HOP/WiSafe2-to-HomeAssistant-Bridge/blob/master/Enclosure/WiSafe2-to-HA-Bridge-enclosure.stl)

![Enclosure-Open](https://github.com/C19HOP/WiSafe2-to-HomeAssistant-Bridge/blob/master/Enclosure/enclosure-open.jpg)
![Enclosure-Closed](https://github.com/C19HOP/WiSafe2-to-HomeAssistant-Bridge/blob/master/Enclosure/enclosure-closed.jpg)


## Why a Nano?
I built this bridge using an Arduino Nano, as opposed to an ESP8266 because:
* I liked the reliability of a direct USB connection (no dependency on the wifi network)
* I needed to use SPI slave mode (the radio is designed to be the master)... The ESP8266 only supports SPI as a master


# Building the project
Program the Arduino Nano, using the sketch code provided. You shouldn't need to change anything.

[Arduino Sketch](https://github.com/C19HOP/WiSafe2-to-HomeAssistant-Bridge/blob/master/Arduino/FireAngelNano.ino)

Although there are some variables declared for the 'EMBEDDED DEVICE ID' and 'MODEL', which you can change if you like. Basically, the radio thinks it's connected to an alarm. The parameters we set here define the type of alarm that the Nano will simulate.
This is used when we initiate a test from HA, or when the alarms are silenced.
The information here just needs to be unique on your network. Chances are you can just leave it all alone.

The only library dependency is the SPI library, which should be installed into the Arduino IDE by default.
If you encounter any trouble uploading the sketch, try setting the processor to ATmega328P (old bootloader). I found is nearly always required if using an Arduino Nano 'compatible'.
![Enclosure-Closed](https://github.com/C19HOP/WiSafe2-to-HomeAssistant-Bridge/blob/master/Arduino/Nano%20bootloader.png)

When you're ready to solder all the components to the PCB, you should find the layout is self-explanatory. The PCB silk-screen shows the correct way to insert the level converters and the Arduino Nano.
The WiSafe2 radio can only be soldered in one way.

![PCB board](https://github.com/C19HOP/WiSafe2-to-HomeAssistant-Bridge/blob/master/PCB/pcb02.png)

The antenna is just a a wire connected to the 'ANT' pin. It should be cut to 17.27cm (1/2 wavelength), or 8.64cm (1/4 wavelength).

Note: Radio modules found in alarms have a backup battery. Whereas the modules from the Strobe units don't (as they are mains powered). If your radio has a battery, I recommend removing it. This ensures the Arduino will initialise the radio from a cold-start if the power is ever cycled.
Removing the battery is also required if you want to use the 3D-printed enclosure I provide.


# Configuring the hardware
There is a jumper which toggles the driver mode. For normal operation with HomeAssistant, the jumper needs to be on.

If you want to receive raw WiSafe2 hex data from the alarm network, for the purpose of further development (to add support for other alarms or new functions, etc. or to use this bridge with something other than HA) then you can remove the jumper. After removing the jumper, the bridge driver will send/receive raw hex data, instead of JSON formatted messages and preset CMDs. 


# Setting the bridge up in HomeAssistant
You'll be plugging the Arduino Nano into your HA server via USB. If it's the only USB-to-Serial device attached, it will show up as /dev/ttyUSB0
If you have more USB-to-serial devices attached, it could be /dev/ttyUSB1, etc. You can check this in your HomeAssistant's Hardware Page.
[http://your-ha-ip:8123/hassio/system](http://your-ha-ip:8123/hassio/system)

Then click the '⋮' icon, and then 'Hardware'

I'll assume you're using /dev/ttyUSB0 for the example configuration.

The following config is done inside the HA configuration.yaml file.
Note: Whilst this HA configuration (yaml) works fine for me, I'm sure the HA yaml gurus amongst you will be able to improve/optimise this configuration significantly.
Please feel free to contribute any improvements and optimisations to these HA configuration files if that's your thing.

First, we'll adjust the recorder. The heartbeat of the bridge updates every 25 seconds or so. It does not seem logical to have HomeAssistant record the history of the heartbeat. We can disable entity recording by adding this:

 ```yaml
recorder:
  exclude:
    entity_globs: 
      - sensor.fireangeldata   
 ```

The input from the bridge is read by a sensor template. This takes the serial data and makes individual sensors out of it, which we can use inside HA.

Here we create the sensor template, with the first two sensors; 'heartbeat' and 'messages'.
Below the 'messages' sensor, I've included 3 example alarms; Smoke, Heat and CO.
Using these 3 examples, hopefully you can see how you could add as many alarms as you need to.

We just give each alarm a name, so we can identify it in HA.
And we set the ID of each alarm, so we can identify it from the WiSafe2 network (the id is the hex number, used several times in each sensor). I'll explain how to get the IDs of your alarms shortley.

 ```yaml
#-------------------------
#FireAngel template sensor
#-------------------------
sensor:
  - platform: serial
    name: fireangeldata
    serial_port: /dev/ttyUSB0
    baudrate: 115200       
  - platform: template
    sensors:
    
      #HeartBeat 
      #Result is Online or Offline based on heartbeat changes!
      fireangel_radio_heartbeat:
        friendly_name: 'HeartBeat'
        icon_template: mdi:heart-pulse
        value_template: >
          {% if states.sensor.fireangeldata.state !='unknown' and (now() - states.sensor.fireangeldata.last_changed).total_seconds() < 125 %}
          Online
          {% else %}
          Offline!
          {% endif %}

      #Messages from the radio to the user
      fireangel_radio_messages:
        friendly_name: 'Messages'
        icon_template: mdi:message-processing
        value_template: >          
          {% if not '{' in states.sensor.fireangeldata.state %}
          {{states.sensor.fireangeldata.state}}
          {% endif %}
          
      #-----------------------------------    
      #2d8d01 | FP2620W2 | Smoke | Bedroom
      #-----------------------------------  
      #EVENT
      fireangel_event_2d8d01: 
        unique_id: 'fireangel_event_2d8d01'
        friendly_name: 'Bedroom Smoke Alarm'
        icon_template: mdi:fire
        value_template: >
          {% if states.sensor.fireangeldata.attributes.device=='2d8d01'.upper() %}
          {{ states.sensor.fireangeldata.attributes.event }}
          {% elif 'MISSING' in states.sensor.fireangel_event_2d8d01.state %}
          {{ states.sensor.fireangel_event_2d8d01.state }}
          {% elif states.sensor.fireangeldata.attributes.event=='SILENCE' %}
          {{ states.sensor.fireangeldata.attributes.event }}
          {% elif 'EMERGENCY' in states.sensor.fireangel_event_2d8d01.state %}
          {{ states.sensor.fireangel_event_2d8d01.state }}
          {% endif %}
      #BATTERY
      fireangel_battery_2d8d01:
        unique_id: 'fireangel_battery_2d8d01'
        friendly_name: 'Bedroom Smoke Alarm'
        icon_template: >
          {% if states.sensor.fireangel_battery_2d8d01.state=='OK' %}
          {{'mdi:battery'}}
          {% elif states.sensor.fireangel_battery_2d8d01.state=='unknown' %}
          {{'mdi:battery-unknown'}}
          {% else %}
          {{'mdi:battery-low'}}
          {% endif %}       
        value_template: >
          {% if states.sensor.fireangeldata.attributes.device=='2d8d01'.upper() and states.sensor.fireangeldata.attributes.battery is defined %}
          {{ states.sensor.fireangeldata.attributes.battery }}
          {% else %}
          {{ states.sensor.fireangel_battery_2d8d01.state }}
          {% endif %}
      #BASE
      fireangel_onbase_2d8d01:
        unique_id: 'fireangel_onbase_2d8d01'
        friendly_name: 'Bedroom Smoke Alarm'
        icon_template: >
          {% if states.sensor.fireangel_onbase_2d8d01.state=='ON' %}
          {{'mdi:home'}}
          {% elif states.sensor.fireangel_onbase_2d8d01.state=='unknown' %}
          {{'mdi:help-circle'}}
          {% else %}
          {{'mdi:alert-circle'}}
          {% endif %}       
        value_template: >
          {% if states.sensor.fireangeldata.attributes.device=='2d8d01'.upper() %}
          {{ states.sensor.fireangeldata.attributes.base }}
          {% else %}
          {{ states.sensor.fireangel_onbase_2d8d01.state }}
          {% endif %}
          
      #-----------------------------------  
      #a76f18 | FP1720W2-R | Heat | Kitchen
      #-----------------------------------  
      #EVENT
      fireangel_event_a76f18: 
        unique_id: 'fireangel_event_a76f18'
        friendly_name: 'Kitchen Heat Alarm'
        icon_template: mdi:fire
        value_template: >
          {% if states.sensor.fireangeldata.attributes.device=='a76f18'.upper() %}
          {{ states.sensor.fireangeldata.attributes.event }}
          {% elif 'MISSING' in states.sensor.fireangel_event_a76f18.state %}
          {{ states.sensor.fireangel_event_a76f18.state }}
          {% elif states.sensor.fireangeldata.attributes.event=='SILENCE' %}
          {{ states.sensor.fireangeldata.attributes.event }}
          {% elif 'EMERGENCY' in states.sensor.fireangel_event_a76f18.state %}
          {{ states.sensor.fireangel_event_a76f18.state }}
          {% endif %}
      #BATTERY
      fireangel_battery_a76f18:
        unique_id: 'fireangel_battery_a76f18'
        friendly_name: 'Kitchen Heat Alarm'
        icon_template: >
          {% if states.sensor.fireangel_battery_a76f18.state=='OK' %}
          {{'mdi:battery'}}
          {% elif states.sensor.fireangel_battery_a76f18.state=='unknown' %}
          {{'mdi:battery-unknown'}}
          {% else %}
          {{'mdi:battery-low'}}
          {% endif %}       
        value_template: >
          {% if states.sensor.fireangeldata.attributes.device=='a76f18'.upper() and states.sensor.fireangeldata.attributes.battery is defined %}
          {{ states.sensor.fireangeldata.attributes.battery }}
          {% else %}
          {{ states.sensor.fireangel_battery_a76f18.state }}
          {% endif %}
      #BASE
      fireangel_onbase_a76f18:
        unique_id: 'fireangel_onbase_a76f18'
        friendly_name: 'Kitchen Heat Alarm'
        icon_template: >
          {% if states.sensor.fireangel_onbase_a76f18.state=='ON' %}
          {{'mdi:home'}}
          {% elif states.sensor.fireangel_onbase_a76f18.state=='unknown' %}
          {{'mdi:help-circle'}}
          {% else %}
          {{'mdi:alert-circle'}}
          {% endif %}       
        value_template: >
          {% if states.sensor.fireangeldata.attributes.device=='a76f18'.upper() %}
          {{ states.sensor.fireangeldata.attributes.base }}
          {% else %}
          {{ states.sensor.fireangel_onbase_a76f18.state }}
          {% endif %}

      #-----------------------------------  
      #ad8003 | W2-CO-10X | CO | Kitchen
      #-----------------------------------  
      #EVENT
      fireangel_event_ad8003: 
        unique_id: 'fireangel_event_ad8003'
        friendly_name: 'Kitchen CO Alarm'
        icon_template: mdi:molecule-co
        value_template: >
          {% if states.sensor.fireangeldata.attributes.device=='ad8003'.upper() %}
          {{ states.sensor.fireangeldata.attributes.event }}
          {% elif 'MISSING' in states.sensor.fireangel_event_ad8003.state %}
          {{ states.sensor.fireangel_event_ad8003.state }}
          {% elif states.sensor.fireangeldata.attributes.event=='SILENCE' %}
          {{ states.sensor.fireangeldata.attributes.event }}
          {% elif 'EMERGENCY' in states.sensor.fireangel_event_ad8003.state %}
          {{ states.sensor.fireangel_event_ad8003.state }}
          {% endif %}
      #BATTERY
      fireangel_battery_ad8003:
        unique_id: 'fireangel_battery_ad8003'
        friendly_name: 'Kitchen CO Alarm'
        icon_template: >
          {% if states.sensor.fireangel_battery_ad8003.state=='OK' %}
          {{'mdi:battery'}}
          {% elif states.sensor.fireangel_battery_ad8003.state=='unknown' %}
          {{'mdi:battery-unknown'}}
          {% else %}
          {{'mdi:battery-low'}}
          {% endif %}       
        value_template: >
          {% if states.sensor.fireangeldata.attributes.device=='ad8003'.upper() and states.sensor.fireangeldata.attributes.battery is defined %}
          {{ states.sensor.fireangeldata.attributes.battery }}
          {% else %}
          {{ states.sensor.fireangel_battery_ad8003.state }}
          {% endif %}   
 ```


We can also send commands to the bridge to perform actions. To enable this, you can add this configuration:

 ```yaml
#---------------------------------------------------------
#FireAngel shell commands (used to send data to the alarms)
#---------------------------------------------------------
shell_command:
   fireangel_test_co: /bin/bash -c "echo -e -n "1~" > /dev/ttyUSB0"
   fireangel_test_smoke: /bin/bash -c "echo -e -n "2~" > /dev/ttyUSB0"
   fireangel_test_all: /bin/bash -c "echo -e -n "3~" > /dev/ttyUSB0"
   fireangel_emergency_co: /bin/bash -c "echo -e -n "4~" > /dev/ttyUSB0"
   fireangel_emergency_smoke: /bin/bash -c "echo -e -n "5~" > /dev/ttyUSB0" 
   fireangel_silence_co: /bin/bash -c "echo -e -n "6~" > /dev/ttyUSB0"    
   fireangel_silence_smoke: /bin/bash -c "echo -e -n "7~" > /dev/ttyUSB0"    
   fireangel_get_pairing: /bin/bash -c "echo -e -n "8~" > /dev/ttyUSB0"
   fireangel_start_pairing: /bin/bash -c "echo -e -n "9~" > /dev/ttyUSB0"
 ```   
   
   
Now you can start adding some of those entities to your dashboard.

Here's my lovelace configuration. In case you want to borrow anything from it:

 ```yaml
  - title: FireAngel
    path: fireangel
    panel: false
    theme: ''
    badges: []
    cards:
      - type: entities
        entities:
          - entity: sensor.fireangel_event_13f43e
            secondary_info: last-updated
          - entity: sensor.fireangel_event_06f93e
            secondary_info: last-updated
          - entity: sensor.fireangel_event_2d8d01
            secondary_info: last-updated
          - entity: sensor.fireangel_event_a76f18
            secondary_info: last-changed
          - entity: sensor.fireangel_event_ad8003
            secondary_info: last-changed
          - entity: sensor.fireangel_event_11ce01
            secondary_info: last-updated
        title: Alarm Events
        show_header_toggle: true
        state_color: false
      - type: entities
        entities:
          - entity: sensor.fireangel_battery_13f43e
          - entity: sensor.fireangel_battery_06f93e
          - entity: sensor.fireangel_battery_2d8d01
          - entity: sensor.fireangel_battery_a76f18
          - entity: sensor.fireangel_battery_ad8003
          - entity: sensor.fireangel_battery_11ce01
        title: Battery Status
      - type: entities
        entities:
          - entity: sensor.fireangel_onbase_13f43e
          - entity: sensor.fireangel_onbase_06f93e
          - entity: sensor.fireangel_onbase_2d8d01
          - entity: sensor.fireangel_onbase_a76f18
        show_header_toggle: true
        state_color: false
        title: Base Status
      - type: entities
        entities:
          - entity: sensor.fireangel_radio_heartbeat
          - entity: sensor.fireangel_radio_messages
          - entity: sensor.fireangeldata
        title: Gateway Information
        state_color: false
      - name: Check Radio Pairing
        show_icon: true
        show_name: true
        tap_action:
          action: call-service
          service: shell_command.fireangel_get_pairing
        type: button
        icon: mdi:signal-variant
        show_state: false
        icon_height: 50px
      - name: Initiate Radio Pairing
        show_icon: true
        show_name: true
        tap_action:
          action: call-service
          service: shell_command.fireangel_start_pairing
        type: button
        icon: mdi:signal-distance-variant
        show_state: true
        icon_height: 50px
      - name: Test Fire
        show_icon: true
        show_name: true
        tap_action:
          action: call-service
          service: shell_command.fireangel_test_smoke
        type: button
        icon: mdi:fire
        show_state: false
        icon_height: 50px
      - name: Test Carbon-monoxide
        show_icon: true
        show_name: true
        tap_action:
          action: call-service
          service: shell_command.fireangel_test_co
        type: button
        icon: mdi:molecule-co
        show_state: false
        icon_height: 50px
      - name: Silence Alarms
        show_icon: true
        show_name: true
        tap_action:
          action: call-service
          service: shell_command.fireangel_silence_smoke
        type: button
        icon: mdi:volume-off
        show_state: false
        icon_height: 50px
 ```

At this point, you probably want to make sure that you at least configure an entities-card on your dashboard called 'Gateway Information' and include: 
* sensor.fireangel_radio_heartbeat 
* sensor.fireangel_radio_messages
* sensor.fireangeldata

If the bridge is communicating with HA, you should see that the heartbeat reports 'online'.
If it isn't working, check the USB connection and configuration.

Next, add yourself some buttons for each of the shell commands you're interested in.
The buttons just call services. e.g. service shell_command.fireangel_start_pairing
Again, see my lovelace configuration if you need examples.

Now you'll want to pair your bridge with your WiSafe network.
Depending where you got your radio from, it may or may not be paired already. 

If you used a new (or reset) radio, you can place the bridge into pairing mode, by pressing the button you created on the dashboard for pairing (fireangel_start_pairing).
Once in pairing mode, press the physical button one one of your alarms (as normal). The messages sensor will show the result.

If you find the network is already paired (because your radio module is 2nd hand), then you should reset it with a double push of the physical button on the back of the radio.
Google "FireAngel unlearn instructions" if you're not sure.
Then, use the 'Check Radio Pairing' dashboard button and observe the message to confirm that the radio is unpaired. You can proceed to pair with your network via HomeAssistant, as previously described.

At this point, when you test any alarm in your network, 'sensor.fireangeldata' will show a JSON string, which starts with the ID of the alarm. Take note of this ID to create the sensors you need for each alarm, based on my sample configuration.

It should just be a case of taking my example, and using search/replace with your alarm ID.

Note: Use lower-case when using the ID in the HA configuration file.

After adding sensors for each of your alarms, and you're pretty much done.

# Value Add
For the full experience, I recommend making sure you have TTS (Google/Alexa) and Mobile Phone notifications setup and working. I won't go into detail here, as this is just general HomeAssistant stuff, and not specific to this project. 
But I'll quickly share the configuration I use for Google wavenet:

 ```yaml
tts:
  - platform: google_cloud
    base_url: http://192.168.0.200:8123
    key_file: googlecloud.json
    voice: en-GB-Wavenet-B
    encoding: linear16
    speed: 0.9
    pitch: -1.0
 ```
 
And for mobile Phone notifications:

 ```yaml
notify:
  - platform: group
    name: all_mobile_app_devices
    services:
      - service: mobile_app_iphone_1
      - service: mobile_app_iphone_2
 ```

With that, I use these automations (automations.yaml) to trigger alerts and notifications.

I receive notifications if the gateway goes offline, as well as when any alarm event occurs.
I also have the events configured as 'critical', so that my phone will make a sound, even if it is on silent.

 ```yaml
- id: '1620428541168'
  alias: FireAngel Test Fail Notifications
  description: FireAngel Test Fail Notifications
  trigger:
  - platform: state
    entity_id: sensor.fireangel_event_13f43e
  - platform: state
    entity_id: sensor.fireangel_event_2d8d01
  - platform: state
    entity_id: sensor.fireangel_event_06f93e
  - platform: state
    entity_id: sensor.fireangel_event_601a03
  - platform: state
    entity_id: sensor.fireangel_event_11ce01
  - platform: state
    entity_id: sensor.fireangel_event_a76f18
  - platform: state
    entity_id: sensor.fireangel_event_ad8003
  condition:
  - condition: template
    value_template: '{{ ''TEST'' in trigger.to_state.state }}'
  - condition: template
    value_template: '{{ states.sensor.fireangeldata.attributes.result==''FAIL''}}'
  action:
  - data_template:
      data:
        apns_headers:
          apns-collapse-id: fireangel-test-notification
      message: A {{ trigger.to_state.state }} event was triggered by the {{ trigger.to_state.attributes.friendly_name}}.
        There is a fault with the alarm! Check or replace it as necessary. See the
        user manual for full details.
      title: FireAngel Alarm Test
    service: notify.all_mobile_app_devices
  - service: tts.google_cloud_say
    data:
      entity_id: media_player.broadcast_group
      message: A {{ trigger.to_state.state }} event was triggered by the {{ trigger.to_state.attributes.friendly_name}}.
        There is a fault with the alarm. Check or replace it as necessary.
  mode: single
- id: '1620428541169'
  alias: FireAngel Test Pass Notifications
  description: FireAngel Test Pass Notifications
  trigger:
  - platform: state
    entity_id: sensor.fireangel_event_13f43e
  - platform: state
    entity_id: sensor.fireangel_event_2d8d01
  - platform: state
    entity_id: sensor.fireangel_event_06f93e
  - platform: state
    entity_id: sensor.fireangel_event_601a03
  - platform: state
    entity_id: sensor.fireangel_event_11ce01
  - platform: state
    entity_id: sensor.fireangel_event_a5b813
  - platform: state
    entity_id: sensor.fireangel_event_a76f18
  - platform: state
    entity_id: sensor.fireangel_event_ad8003
  condition:
  - condition: template
    value_template: '{{ ''TEST'' in trigger.to_state.state }}'
  - condition: template
    value_template: '{{ states.sensor.fireangeldata.attributes.result==''PASS''}}'
  action:
  - data_template:
      data:
        apns_headers:
          apns-collapse-id: fireangel-test-notification
      message: A {{ trigger.to_state.state }} event was triggered by the {{ trigger.to_state.attributes.friendly_name}}.
        The alarm is functioning normally.
      title: FireAngel Alarm Test
    service: notify.all_mobile_app_devices
  - service: tts.google_cloud_say
    data:
      entity_id: media_player.broadcast_group
      message: A {{ trigger.to_state.state }} event was triggered by the {{ trigger.to_state.attributes.friendly_name}}.
        This is only a test. The alarm is functioning normally.
  mode: single
- id: '1620428541170'
  alias: FireAngel Base Notifications
  description: FireAngel Base Notifications
  trigger:
  - platform: state
    entity_id: sensor.fireangel_onbase_13f43e
    to: 'OFF'
    for: '1'
  - platform: state
    entity_id: sensor.fireangel_onbase_2d8d01
    to: 'OFF'
    for: '1'
  - platform: state
    entity_id: sensor.fireangel_onbase_06f93e
    to: 'OFF'
    for: '1'
  - platform: state
    entity_id: sensor.fireangel_onbase_a76f18
    to: 'OFF'
    for: '1'
  condition: []
  action:
  - data_template:
      data:
        apns_headers:
          apns-collapse-id: fireangel-off-base-notification
      message: Warning! The {{ trigger.to_state.attributes.friendly_name }} was removed
        from its base
      title: FireAngel Alarm Off Base
    service: notify.all_mobile_app_devices
  - service: tts.google_cloud_say
    data:
      entity_id: media_player.broadcast_group
      message: Warning! The {{ trigger.to_state.attributes.friendly_name }} was removed
        from its base
  mode: single
- id: '1620428541171'
  alias: FireAngel Emergency Notifications
  description: FireAngel Emergency Notifications
  trigger:
  - platform: state
    entity_id: sensor.fireangel_event_13f43e
  - platform: state
    entity_id: sensor.fireangel_event_2d8d01
  - platform: state
    entity_id: sensor.fireangel_event_06f93e
  - platform: state
    entity_id: sensor.fireangel_event_601a03
  - platform: state
    entity_id: sensor.fireangel_event_11ce01
  - platform: state
    entity_id: sensor.fireangel_event_a5b813
  - platform: state
    entity_id: sensor.fireangel_event_a76f18
  - platform: state
    entity_id: sensor.fireangel_event_ad8003
  condition:
  - condition: template
    value_template: '{{ ''EMERGENCY'' in trigger.to_state.state }}'
  action:
  - data_template:
      data:
        push:
          sound:
            name: default
            critical: 1
            volume: 1.0
      message: WARNING! A {{ trigger.to_state.state }} has been reported by the {{
        trigger.to_state.attributes.friendly_name}}. Take action now!
      title: FireAngel Alarm Emergency
    service: notify.all_mobile_app_devices
  - service: tts.google_cloud_say
    data:
      entity_id: media_player.broadcast_group
      message: WARNING! WARNING! A {{ trigger.to_state.state }} has been reported
        by the {{ trigger.to_state.attributes.friendly_name}}. Take action now! This
        is not a test. Repeat. A {{ trigger.to_state.state }} has been reported by
        the {{ trigger.to_state.attributes.friendly_name}}. Take action now! This
        is not a test.
  mode: single
- id: '1620428541172'
  alias: FireAngel Offline Notifications
  description: FireAngel Offline Notifications
  trigger:
  - platform: state
    entity_id: sensor.fireangel_radio_heartbeat
    to: Offline!
    for: '35'
  condition: []
  action:
  - data_template:
      data:
        push:
          sound:
            name: default
            critical: 1
            volume: 1.0
      message: WARNING! The FireAngel Gateway is Offline
      title: FireAngel Gateway Offline
    service: notify.all_mobile_app_devices
  mode: single
- id: '1620428541173'
  alias: FireAngel Missing Notifications
  description: ''
  trigger:
  - platform: state
    entity_id: sensor.fireangel_event_13f43e
    to: MISSING
    for: '1'
  - platform: state
    entity_id: sensor.fireangel_event_2d8d01
    to: MISSING
    for: '1'
  - platform: state
    entity_id: sensor.fireangel_event_06f93e
    to: MISSING
    for: '1'
  - platform: state
    entity_id: sensor.fireangel_event_601a03
    to: MISSING
    for: '1'
  - platform: state
    entity_id: sensor.fireangel_event_11ce01
    to: MISSING
    for: '1'
  - platform: state
    entity_id: sensor.fireangel_event_a76f18
    to: MISSING
    for: '1'
  - platform: state
    entity_id: sensor.fireangel_event_ad8003
    to: MISSING
    for: '1'
  condition: []
  action:
  - data_template:
      data:
        apns_headers:
          apns-collapse-id: fireangel-missing-notification
      message: Warning! The {{ trigger.to_state.attributes.friendly_name }} has been
        reported as missing by other alarms in the network
      title: FireAngel Alarm Off Base
    service: notify.all_mobile_app_devices
  - service: tts.google_cloud_say
    data:
      entity_id: media_player.broadcast_group
      message: Warning! The {{ trigger.to_state.attributes.friendly_name }} has been
        reported as missing by other alarms in the network 
 ```


