# TL;DR
Sengled, a large suplier of LED smart bulbs on Amazon, has recently cutoff service and seemingly disbanded as a company. Amazon/Alexa has dropped official support from their products, and their _WiFI only_ cloud connected devices have essentially turned into e-waste. However, the "brains" of these bulbs are using an ESP8266 which is a popular development platform that supports many different types of firmware.

Using some basic soldering skills and a little bit of tinkering, we are able to remove Sengled's firmware and instead side-load Tasmota onto the bulb's ESP8266, a popular open-source MQTT platform used for controlling smart bulbs such as these.

# DISCLAIMER
This guide is purely for educational purposes and peforming the steps outlines below could potentially damage your product if done incorrectly. Working on/with electronics of any kind carries inherent risks and proceeding with the steps below is completely up to the user (you). I am making this guide just to show proof of concept and share knowledge from my own experiences, I waive any and all responsibility associated with this guide.

# Introduction
## Supplies Needed
- Soldering iron & solder
- Desoldering pump or wick
- Small AWG wire
- Small flathead screwdriver or pry-tool
- USB to UART serial cable (alternatively, you can use an ESP32 dev board - will explain more later in this guide)
- Computer with Tasmotizer (GUI) or esptools installed
- tasmota.bin file (more information can be found here: https://tasmota.github.io/docs/Getting-Started/#needed-hardware)

## Step 1: Dissassembling the bulb
### Removing the "dome":
The dome of the bulb is held in place mostly by silicone adhesive. There appears to be some latch mechanism but in my experience, prying you way around the bulb seems to be just fine in the removal process.

Start by taking your flathead screwdriver and pushing it between the housing and the dome. The PCB is recessed below the rim where the dome meets the housing so you can push the screwdriver farily far in without risking damage to the PCB.

Once inserted, work you way around the base of the dome while prying upwards to seperate the two pieces. This can take some time and patience if you want to keep the rim of the housing looking nice as it tends to bend and crack pretty easily. Otherwise if you are not concered, you may be a little more forceful - the actual plastic of the dome seems to be pretty study and it pretty hard to break/crack.

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/1%20-%20Removing%20dome.JPG?raw=true" width=50% height=50% align="center">

Once enough of the silicone is removed, you should be able to pull the dome off of the housing.

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/2%20-%20Dome%20removed.JPG?raw=true" width=50% height=50% align="center">

### Cleaning up the silicone:
After the dome is removed, in order to seperate the PCB from the housing, you will want to remove as much of the surface adhesive as possible. Using your flathead screwdriver, work your way around the PCB scrapping as much of the silicone away as possible.

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/3%20-%20Expoxy.JPG?raw=true" width=50% height=50% align="center">

With a little bit of patience, it should start to peel off relatively easily.

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/4%20-%20Expoxy%20removed.JPG?raw=true" width=50% height=50% align="center">


