# TL;DR
Sengled, a large suplier of LED smart bulbs on Amazon, has recently cutoff service and seemingly disbanded as a company. Amazon/Alexa has dropped official support from their app, and their WiFI _only_ cloud connected devices have essentially turned into e-waste. However, the "brains" of these bulbs are using an ESP8266 which is a popular development platform that supports many different types of Firmware.

Using some basic soldering skills and a little bit of tinkering, we are able to remove Sengled's proprietary firmware and instead side-load Tasmota onto the bulb's ESP8266, a popular open-source MQTT platform used for controlling smart bulbs such as these.

# DISCLAIMER
This guide is purely for educational purposes and peforming the steps outlines below could potentially damage your product if done incorrectly. Working on/with electronics of any kind and soldering carries inherent risks and proceeding with the steps below is completely up to the user (you). I am making this guide just to show proof of concept and share knowledge from my own experiences, proceed at your own risk.
