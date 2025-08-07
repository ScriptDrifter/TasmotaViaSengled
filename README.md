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

# Soldering and flashing Tasmota
## Getting access to the serial pads
### Removing the capacitor
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

<br>
You are now ready to connect your USB to UART serial cable or ESP32 dev board and begin installing Tasmota!

## Flashing Tasmota

Once you have your wires/leads connected to the board, flashing the firmware is a pretty smooth process. Here is the pinout used to connect to your interface of choice

3V3  ->  3V
<br>
GND  ->  GND
<br>
IO0  ->  GND
<br>
RX   ->  RX
<br>
TX   ->  TX
<br>

### Flashing using a USB to UART adapter

If you are using a USB to UART cable, the connection process is pretty straightfoward following the pinout diagram above - **just make sure your adapter supports 3.3V logic output and not 5V, if using 5V, you will need to use a level shifter!**

After connecting your Sengled PCB to your adapter, you may begin the flashing process. I won't go to in depth on how to actually flash Tasmota as there are many other, much better guides to doing this but in my experience, downloading the Tasmotizer GUI interface was by far the easiest to work with.

You can find more information and download the application here: https://github.com/tasmota/tasmotizertab=readme-ov-file#installation-and-how-to-run

_The ESP8266 supports tamota.bin and you do not need to use tasmota-lite.bin_

Once the reading, erasing, and flashing is complete, you should be prompted with the following screen - you now have Tasmota installed to your bulb and you can begin the reassembly process!

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/19%20-%20Tasmo%20push.PNG?raw=true" width=50% height=50% align="center">

### Flashing using an ESP32 _dev board_

If you do not have a USB to UART serial adapter laying around (me), you can use an ESP32 dev board for the flashing process as well - the important part is that it is a dev board. Dev boards use the ESP32 chipset, but come with the pins pre-soldered and have other integrated electronics already in place - the easiest giveway to tell if your ESP32 is a dev board is that it will have some kinda of micro-USB or USB-C to plug in to your computer.

ESP32 dev boards already have the USB to UART controller built into the board, and we just need to bypass the ESP chip to use the serial communicator already built in. In order to do this, the easiest way is to identify if you have an "EN" pin already accessible on your board. If so, follow the the pinout below for wiring:

3V3  ->  3V
<br>
GND  ->  GND
<br>
**EN -> GND**
<br>
IO0  ->  GND
<br>
RX   ->  RX (may be be labeled RX0 on your board)
<br>
TX   ->  TX (may be be labeled TX0 on your board)
<br>

Connect your dev board via USB to your computer once wired, and follow the same intructions above for flashing.

# Reassembling the bulb
Assuming flashing Tasmota via your install method of choice went smoothly, the hard part is over! All that's left to do is disconnect your leads from the board, and repeat the steps we used to take apart the board but - _in reverse_ :)

### Removing the leads
Start by desoldering and removing the leads from the Sengled PCB - you can just heat up the solder and pull the wire off, leaving the beads we made earlier on the board still leave plenty of clearance for the capacitor we need to put back on.

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/20%20-%20Serial%20pad%20lead%20removed.JPG?raw=true" width=50% height=50% align="center">

### Resolder the capacitor back on

Feed the two capacitor leads in to the through-holes we made earlier, and resolder them back into place. **Please note, the capacitor appears to be polarized so we must install it in the same orietation as we removed it, with the black arrows facing outwards towards the antenna/where the LED PCB would be installed.

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/21%20-%20Resolder%20cap.JPG?raw=true" width=50% height=50% align="center">

### Re-apply thermal paste and LED PCB
Replace the thermal paste/pads we took off from the board earlier and put the LEDs PCB back on to the power PCB by feeding through the pin headers

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/22%20-%20Reapply%20therm.JPG?raw=true" width=50% height=50% align="center">

### Putting the PCB back into the housing
Slide the full assembly with the LED PCB back into the white shell of the bulb as below

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/23%20-%20PCB%20back%20in%20housing.JPG?raw=true" width=50% height=50% align="center">

### Clipping the "socket" back on and replacing the pin
Clipping the alumnium socket back on can be a little tricky but with a little force, it should snap back in to place. **Before doing this, make sure the black wire from the PCB is bent downwards so it can make contact with the "outer ring" of the aluminum socket, and feed the red wire through the center hole of the socket before snapping back into place.**

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/24%20-%20Socket%20back%20on.JPG?raw=true" width=50% height=50% align="center">

With the aluminum socket clipped back into place, you can now click the pin we took out earlier back in as well. Make sure the red wire that you fed through the center hole is a little bit of a bend so the pin being snapped back in is making clean contact the the red lead.

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/25%20-%20Pin%20back%20in%20socket.JPG?raw=true" width=50% height=50% align="center">

### Putting the "dome" back on

With the PCB assembly back into the housing and the aluminum socket clipped back into place, you may now put the dome/diffuser back on to the housing - it should kinda clip back on to the housing and then you can rotate it to make sure it is flush

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/26%20-%20Dome%20back%20on.JPG?raw=true" width=50% height=50% align="center">

**And just like that, you now have a Sengled WiFi bulb flashed with Tasmota and ready to connect to your network!**

**Plug your bulb back in, navigate on your phone or computer to available WiFi networks, and select "tasmota_xxx" - this should open a webpage where you can select your WiFi network and input the access credentials** 

# Basic Tasmota configuration

I wasn't planning to go into much detail with the configuring of Tasmota once installed, as there are many resources available to do so, but I do want to highlight a couple of Sengled specific configurations.

## Importing the correct Sengled bulb template
In order to tell Tasmota which pins to use to correctly control the LEDs, you must import the Sengled specific template.

You can do this by following the steps below:

On the web portal, go to **Configuration -> Other**

In the template parameter box, delete the default text and enter the following:

```{"NAME":"Sengled RGBW","GPIO":[0,0,0,0,0,0,0,0,417,416,419,418,0,0],"FLAG":0,"BASE":18}```

Select the "Activate" check box and click save. This will restart your device and the colors should now be correctly controllable via the Web UI

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/29%20-%20Sengled%20Template.png?raw=true" width=50% height=50% align="center">

## Enabling RGBW

By default, the white LED's on your Sengled bulb (if W31-15 model as shown on this guide) will not work by default. This is easy to enable via a command in the Tasmota console.

From the homepage of the Web UI on your Tasmota device, click the Console tab and paste the following line of code in the command line:

```SetOption105 1```

Press enter, and you should not have a white light control bar on the homepage.

<img src="https://github.com/ScriptDrifter/TasmotaViaSengled/blob/main/images/31%20-%20WWhite%20bar.PNG?raw=true" width=50% height=50% align="center">
