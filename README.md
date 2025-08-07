# TL;DR
Sengled, a large supplier of LED smart bulbs on Amazon, has recently cut off service and appears to have disbanded as a company. Amazon/Alexa has dropped official support for their products, and their WiFi-only, cloud-connected devices have essentially turned into e-waste. However, the "brains" of these bulbs use an ESP8266, which is a popular development platform that supports various types of firmware.

Using some basic soldering skills, and a little bit of tinkering, we are able to remove Sengled's firmware and instead flash Tasmota, a popular open-source firmware for ESP-based devices, onto the bulb's ESP8266.

# DISCLAIMER
This guide is purely for educational purposes and performing the steps outlined below could potentially damage your product if done incorrectly. Working with electronics of any kind carries inherent risks, and proceeding with the steps below is entirely at your own risk. I am making this guide just to show proof of concept and share knowledge from my own experiences; I waive any and all responsibility associated with this guide.

# Table of Contents
- [Supplies Needed](#supplies-needed)
- [Disassembling the Bulb](#disassembling-the-bulb)
- [Soldering and Flashing Tasmota](#soldering-and-flashing-tasmota)
- [Reassembling the Bulb](#reassembling-the-bulb)
- [Basic Tasmota Configuration](#basic-tasmota-configuration)
- [Additional Resources](#additional-resources)

# Supplies Needed
- Soldering iron & solder
- Desoldering pump or wick
- Small gauge (AWG) wire
- Small flathead screwdriver or pry tool
- USB to UART serial adapter (alternatively, you can use an ESP32 dev board - this is explained later in the guide)
- Computer with Tasmotizer (GUI) or `esptool` installed
- `tasmota.bin` file (more information can be found here: https://tasmota.github.io/docs/Getting-Started/#needed-hardware)

<br>

# Disassembling the Bulb

### Removing the "Dome":
The dome of the bulb is held in place mostly by silicone adhesive. There appears to be some latch mechanism but in my experience, prying your way around the bulb seems to be just fine in the removal process.

Start by taking your flathead screwdriver and pushing it between the housing and the dome. The PCB is recessed below the rim where the dome meets the housing so you can push the screwdriver fairly far in without risking damage to the PCB.

Once inserted, work your way around the base of the dome while prying upwards to separate the two pieces. This can take some time and patience if you want to keep the rim of the housing looking nice as it tends to bend and crack pretty easily. Otherwise if you are not concerned, you may be a little more forceful - the actual plastic of the dome seems to be quite sturdy and it is pretty hard to break/crack.

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/1%20-%20Removing%20dome.JPG?raw=true" width=50% height=50% align="center">

Once enough of the silicone is removed, you should be able to pull the dome off of the housing.

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/2%20-%20Dome%20removed.JPG?raw=true" width=50% height=50% align="center">

### Cleaning Up the Silicone:
After the dome is removed, in order to separate the PCB from the housing, you will want to remove as much of the surface adhesive as possible. Using your flathead screwdriver, work your way around the PCB scraping as much of the silicone away as possible.

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/3%20-%20Expoxy.JPG?raw=true" width=50% height=50% align="center">

With a little bit of patience, it should start to peel off relatively easily.

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/4%20-%20Expoxy%20removed.JPG?raw=true" width=50% height=50% align="center">

### Removing the Socket

Removing the socket on the bottom on the bulb can be a little tricky but nothing impossible. Using your flathead screwdriver or a small blade, wedge the tool between the white housing and the socket and begin to pull up the metal. Work your way around the socket trying to bend the metal up and away from the housing. As you continue to work around the area, you may notice the aluminum starting to separate from the housing - once you have enough room to wedge the flathead inwards, you can pry the aluminum socket away and it should pop off.

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/5%20-%20Removing%20socket.JPG?raw=true" width=50% height=50% align="center">
<br>

**To this:**

<br>
<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/6%20-%20Socket%20unclipped.JPG?raw=true" width=50% height=50% align="center">

<br><br>
Once the socket is disconnected from the bulb and hanging on only by the single red wire, **pull the socket directly away from the housing in the opposite direction of the PCB/top of the bulb.** If you pull it away at an angle, you may break the red wire off of the PCB and will need to resolder this piece back later.

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/7%20-%20Socket%20removed.JPG?raw=true" width=50% height=50% align="center">

<br>
Assuming the red wire disconnected and came away from the socket cleanly, you can now push the pin out from the socket assembly - this makes reassembling the bulb much easier later on.

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/8%20-%20Pushing%20out%20pin.JPG?raw=true" width=50% height=50% align="center">
<br>

**To this:**

<br>
<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/9%20-%20Pin%20removed.JPG?raw=true" width=50% height=50% align="center">

### Removing the PCB Assembly from the Housing
Once the socket has been disconnected from the bottom of the bulb, you should be able to push the entire assembly up towards the top of the housing where the dome _would be_ and it should completely slide out. If it isn't sliding out, try removing more of the silicone that adheres the LED PCB to the housing.

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/10%20-%20Assembly%20removal.JPG?raw=true" width=50% height=50% align="center">
<br>

**To this:**

<br>
<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/11%20-%20Assembly%20out.JPG?raw=true" width=50% height=50% align="center">

With the complete assembly removed from the housing, disconnect the LED board from the power PCB - this should pull off with ease as it is connected with only 6 header pins. You can also remove any of the thermal paste to make more room, but keep this handy as we will want to put this back on the board before reassembling.

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/12%20-%20Full%20disasemb.JPG?raw=true" width=50% height=50% align="center">

# Soldering and Flashing Tasmota
## Getting Access to the Serial Pads
### Removing the Capacitor
In order to get the best access to the serial pads needed to flash our new firmware, I found it best to remove the capacitor in the way of the pads.

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/13%20-%20Cap%20to%20remove.JPG?raw=true" width=50% height=50% align="center">

To do this, start by desoldering the two pins circled below:

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/14%20-%20Points%20to%20unsolder.JPG?raw=true" width=50% height=50% align="center">

Using some solder wick or a desoldering pump, remove the solder on the two pins holding the capacitor in place and pull the cap away from the board.

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/15%20-%20Cap%20removed.JPG?raw=true" width=50% height=50% align="center">

### Creating the Serial Connectors
With the capacitor removed from the board, we now have clean access to the serial pads needed to flash Tasmota to our Sengled bulbs. Circled in red are the relevant pads for this install (3V3, GND, RX, TX, and GPIO0 (notated IO0 on the PCB):

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/16%20-%20Serial%20pads.JPG?raw=true" width=50% height=50% align="center">

Tin the pads with a little bit of solder to make joining a wire to them easier (no one said it has to be pretty).

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/17%20-%20Serial%20pads%20tinned.JPG?raw=true" width=50% height=50% align="center">

With the pads tinned, now solder your 5 wires into place:

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/18%20-%20Serial%20pads%20wleads.JPG?raw=true" width=50% height=50% align="center">

<br>
You are now ready to connect your USB to UART serial cable or ESP32 dev board and begin installing Tasmota!

## Flashing Tasmota

Once you have your wires/leads connected to the board, flashing the firmware is a pretty smooth process. Here is the pinout used to connect to your interface of choice:

|Sengled PCB | USB to UART Adapter |
| :---------- | :------------------ |
| 3V3       | 3.3V              |
| GND       | GND               |
| IO0       | GND               |
| RX        | RX                |
| TX        | TX                |

### Flashing Using a USB to UART Adapter

If you are using a USB to UART cable, the connection process is pretty straightforward following the pinout diagram above - **just make sure your adapter supports 3.3V logic output and not 5V, if using 5V, you will need to use a level shifter!**

After connecting your Sengled PCB to your adapter, you may begin the flashing process. I won't go too in depth on how to actually flash Tasmota as there are many other, much better guides to doing this but in my experience, downloading the Tasmotizer GUI interface was by far the easiest to work with.

You can find more information and download the application here: https://github.com/tasmota/tasmotizer#installation-and-how-to-run

_The ESP8266 supports `tasmota.bin` and you do not need to use `tasmota-lite.bin`_

Once the reading, erasing, and flashing is complete, you should be prompted with the following screen - you now have Tasmota installed to your bulb and you can begin the reassembly process!

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/19%20-%20Tasmo%20push.PNG?raw=true" width=50% height=50% align="center">

### Flashing Using an ESP32 _Dev Board_

If you do not have a USB to UART serial adapter laying around (me), you can use an ESP32 dev board for the flashing process as well - the important part is that it is a dev board. Dev boards use the ESP32 chipset, but come with the pins pre-soldered and have other integrated electronics already in place - the easiest giveaway to tell if your ESP32 is a dev board is that it will have some kind of micro-USB or USB-C to plug in to your computer.

ESP32 dev boards already have the USB to UART controller built into the board, and we just need to bypass the ESP chip to use the serial communicator already built in. In order to do this, the easiest way is to identify if you have an "EN" pin already accessible on your board. If so, follow the pinout below for wiring:

|Sengled PCB | ESP32 Dev Board |
| :---------- | :-------------- |
| 3V3       | 3V3         |
| GND       | GND         |
| EN (ESP32)| GND         |
| IO0       | GND         |
| RX        | RX (RX0)    |
| TX        | TX (TX0)    |

Connect your dev board via USB to your computer once wired, and follow the same instructions above for flashing.

# Reassembling the Bulb
Assuming flashing Tasmota via your install method of choice went smoothly, the hard part is over! All that's left to do is disconnect your leads from the board, and repeat the steps we used to take apart the board but - _in reverse_ :)

### Removing the Leads
Start by desoldering and removing the leads from the Sengled PCB - you can just heat up the solder and pull the wire off, leaving the beads we made earlier on the board still leaves plenty of clearance for the capacitor we need to put back on.

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/20%20-%20Serial%20pad%20lead%20removed.JPG?raw=true" width=50% height=50% align="center">

### Re-solder the Capacitor Back On

Feed the two capacitor leads in to the through-holes from earlier, and resolder them back in place. **Please note, the capacitor appears to be polarized so we must install it in the same orientation as we removed it, with the black arrows facing outwards towards the antenna/where the LED PCB would be installed.**

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/21%20-%20Resolder%20cap.JPG?raw=true" width=50% height=50% align="center">

### Re-apply Thermal Paste and LED PCB
Replace the thermal paste/pads we took off from the board earlier and put the LED PCB back on to the power PCB by feeding through the pin headers

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/22%20-%20Reapply%20therm.JPG?raw=true" width=50% height=50% align="center">

### Putting the PCB Back Into the Housing
Slide the full assembly with the LED PCB back into the white shell of the bulb as below

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/23%20-%20PCB%20back%20in%20housing.JPG?raw=true" width=50% height=50% align="center">

### Clipping the "Socket" Back On and Replacing the Pin
Clipping the aluminum socket back on can be a little tricky but with a little force, it should snap back in to place. **Before doing this, make sure the black wire from the PCB is bent downwards so it can make contact with the "outer ring" of the aluminum socket, and feed the red wire through the center hole of the socket before snapping back into place.**

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/24%20-%20Socket%20back%20on.JPG?raw=true" width=50% height=50% align="center">

With the aluminum socket clipped back into place, you can now click the pin we took out earlier back in as well. Make sure the red wire that you fed through the center hole is a little bit of a bend so the pin being snapped back in is making contact with the red lead.

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/25%20-%20Pin%20back%20in%20socket.JPG?raw=true" width=50% height=50% align="center">

### Putting the "Dome" Back On

With the PCB assembly back into the housing and the aluminum socket clipped back into place, you may now put the dome/diffuser back on to the housing - it should kinda clip back on to the housing and then you can rotate it to make sure it is flush

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/26%20-%20Dome%20back%20on.JPG?raw=true" width=50% height=50% align="center">

**And just like that, you now have a Sengled WiFi bulb flashed with Tasmota and ready to connect to your network!**

# Basic Tasmota Configuration

**Plug your bulb back in, navigate on your phone or computer to available WiFi networks, and select "tasmota_xxx" - this should open a webpage where you can select your WiFi network and input the access credentials** 

I wasn't planning to go into much detail with the configuring of Tasmota once installed, as there are many resources available to do so, but I do want to highlight a couple of Sengled specific configurations.

## Importing the Correct Sengled Bulb Template
In order to tell Tasmota which pins to use to correctly control the LED's, you must import the Sengled specific template.

You can do this by following the steps below:

1. On the web portal, go to **Configuration -> Other**

2. In the template parameter box, delete the default text and enter the following:

`{"NAME":"Sengled RGBW","GPIO":[0,0,0,0,0,0,0,0,417,416,419,418,0,0],"FLAG":0,"BASE":18}`

3. Select the "Activate" check box and click save. This will restart your device and the colors should now be correctly controllable via the Web UI

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/29%20-%20Sengled%20Template.png?raw=true" width=50% height=50% align="center">

## Enabling RGBW

By default, the white LED's on your Sengled bulb (if W31-15 model as shown on this guide) will not work by default. This is easy to enable via a command in the Tasmota console.

From the homepage of the Web UI on your Tasmota device, click the Console tab and paste the following line of code in the command line:

`SetOption105 1`

Press enter, and you should now have a white light control bar on the homepage.

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/31%20-%20WWhite%20bar.PNG?raw=true" width=50% height=50% align="center">

# Additional Resources

- [Tasmota Homepage](https://tasmota.github.io/)
- [Tasmotizer Flashing Tool](https://github.com/tasmota/tasmotizer)
- [Additional Tasmota Commands](https://tasmota.github.io/docs/Commands/)
- [Getting Started with Tasmota in Home Assistant – Local Bytes](https://blog.mylocalbytes.com/kb/2022-10-01/getting-started-with-tasmota/) – A great step-by-step guide to integrating MQTT and Tasmota with Home Assistant.
