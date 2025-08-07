# TL;DR
Sengled, a large suplier of LED smart bulbs on Amazon, has recently cutoff service and seemingly disbanded as a company. Amazon/Alexa has dropped official support from their products, and their _WiFI only_ cloud connected devices have essentially turned into e-waste. However, the "brains" of these bulbs are using an ESP8266 which is a popular development platform that supports many different types of firmware.

Using some basic soldering skills and a little bit of tinkering, we are able to remove Sengled's firmware and instead side-load Tasmota onto the bulb's ESP8266, a popular open-source MQTT platform used for controlling smart bulbs such as these.

# *DISCLAIMER*
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

## Dissassembling the bulb
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

### Removing the socket

Removing the socket on the bottom on the bulb can be a little tricky but nothing impossible. Using your flathead screwdriver or a small blade, wedge the tool between the white housing and the socket and begin to pull up the metal. Work your way around the socket trying to bend the metal up and away from the housing. As you continue to work around the area, you may notice the aluminum starting to separate from the housing - once you have enough room to wedge the flathead inwards, you can pry the aluminum socket away and it should pop off.

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/5%20-%20Removing%20socket.JPG?raw=true" width=50% height=50% align="center">
<br>

**To this:**

<br>
<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/6%20-%20Socket%20unclipped.JPG?raw=true" width=50% height=50% align="center">

<br><br>
Once the socket is disconnected from the bulb and hanging on only by the single red wire, **pull the socket directly away from the housing in the opposite direction of the PCB/top of the bulb.** If you pull it away at an angle, you may break the red wire off of the PCB and will need to resolder this piece bacn later.

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/7%20-%20Socket%20removed.JPG?raw=true" width=50% height=50% align="center">

<br>
Assuming the red wire disconnected and came away from the socket cleanly, you can now push the pin out from the socket assembly - this wil make reassembling the bulb much easier later on.

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/8%20-%20Pushing%20out%20pin.JPG?raw=true" width=50% height=50% align="center">
<br>

**To this:**

<br>
<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/9%20-%20Pin%20removed.JPG?raw=true" width=50% height=50% align="center">

### Removing the PCB assembly from the housing
Once the socket has been disconnected from the bottom of the bulb, you should be able to push the entire assembly up towards the top of the housing where the dome _would be_ and it should completely slide out. If it isn't sliding out, try removing more of the silicone the adheres the LED PCB to the housing.

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/10%20-%20Assembly%20removal.JPG?raw=true" width=50% height=50% align="center">
<br>

**To this:**

<br>
<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/11%20-%20Assembly%20out.JPG?raw=true" width=50% height=50% align="center">

With the complete assembly removed from the housing, disconnect the LED board from the power PCB - this should pull off with ease as it is connected with only 6 header pins. You can also remove any of the thermal paste to make give yourself more clearance, but keep this handy as we will want to put this back on the board before reassembling.

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/12%20-%20Full%20disasemb.JPG?raw=true" width=50% height=50% align="center">

# Soldering flashing Tasmota
## Getting access to the serial pads
### Removing the capcitor
In order to get the best access to the serial pads needed to flash our new firmware, I found it best to remove the capcitor in the way of the pads.

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/13%20-%20Cap%20to%20remove.JPG?raw=true" width=50% height=50% align="center">

To do this, start by desoldering the two pins circled below:

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/14%20-%20Points%20to%20unsolder.JPG?raw=true" width=50% height=50% align="center">

Using some solder wick or a desoldering pump, remove the solder on the two pins holding the capcitor in place and pull the cap away from the board.

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/15%20-%20Cap%20removed.JPG?raw=true" width=50% height=50% align="center">

### Creating the serial connectors
With the capacitor removed from the board, we now have clean access to the serial pads needed to flash Tasmota to our Sengled bulbs. Circled in red are the relevant pads for this install (3V3, GND, RX, TX, and GPIO0 (notated IO0 on the PCB):

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/16%20-%20Serial%20pads.JPG?raw=true" width=50% height=50% align="center">

Tin the pads with a little bit of solder to make joining a wire to them easier (no one said it has to be pretty).

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/17%20-%20Serial%20pads%20tinned.JPG?raw=true" width=50% height=50% align="center">

With the pads tinned, now solder your 5 wires into place:

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/18%20-%20Serial%20pads%20wleads.JPG?raw=true" width=50% height=50% align="center">

You are now ready to connect your USB to UART serial cable or ESP32 dev board and begin flashing Tasmota!

## Flashing Tasmota




