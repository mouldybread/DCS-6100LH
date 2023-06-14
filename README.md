# Hacking the D-Link DCS-6100LH
![DCS-6100LH](DCS-6100LH_A1_Image_front.png)
## Intro

The D-Link DCS-6100LH is a 2MP Wifi-only 5V IP Camera in a decent hardware package. D-Link advertises this model with RTSP but provides no information and has apparently stripped the local RTSP functionality from the device via forced firmware upgrades, allowing only remote access to the camera feed via their app and website.

My camera arrived with firmware version 1.04.05(3.5.23-b01)

I can confirm the RTSP stream works after downgrading to version 1.01 of the firmware, allowing me to use the camera with Home Assistant. It is possible to do so without hardware modification.

Downgrading also enabled debugging output (accessed via TX and RX pins on the board) which allowed me to isolate the stream url. The stream url does not match any other published URLs. 

I only needed this one simple thing(thanks d-link), so I won't be upgrading to newer firmware to test if it is just a case of nobody knowing the url. It may still work after upgrading as nmap shows the port is still open. If you test this on later firmware and it works please let me know.

**As always: messing around with the firmware of these things risks irreparably bricking them. You do so at your own risk. I can't help if things go wrong.**

## TL:DR

RTSP VLC Stream url:<br>
  rtsp://@192.168.3.53:554/live/profile.1/video

Inline credentials are deprecated. You will be prompted for a username and password:<br>
Username: admin<br>
Password: pincode from the bottom of your device<br>

Pincode can be recovered by flashing old firmware which has a bunch of debugging stuff left on.

## Firmware downgrade

It is possible to boot the device into recovery mode and downgrade the firmware as of firmware version 1.04.05

Allow the device to boot and use a pin to hold down the reset button for around 10 seconds. The device will boot into AP mode. You can then connect to the camera's wifi access point using the password on the sticker on the base of the device(not the pin).

The onboard DHCP server will give you an IP resembling 192.168.0.40. You can access the recovery interface at http://192.168.0.20/<br>

You can then upload a firmware file, which you can download from:<br>
  [DCS-6100LH A1 V1.00](https://pmdap.dlink.com.tw/PMD/GetAgileFile?itemNumber=FIR2000285&fileName=DCS6100LHAx_FW100B09.bin&fileSize=673988.0;1085472.0;)<br>
  [DCS-6100LH A1 V1.01](https://pmdap.dlink.com.tw/PMD/GetAgileFile?itemNumber=FIR2000413&fileName=DCS6100LHAx_FW101B09.bin&fileSize=1136672.0;677931.0;)<br>
  [DCS-6100LH A1 V1.02](https://pmdap.dlink.com.tw/PMD/GetAgileFile?itemNumber=FIR2100011&fileName=DCS6100LHAx_FW102B02.bin&fileSize=751930.0;1146912.0;)<br>
  [DCS-6100LH A1 V1.03](https://pmdap.dlink.com.tw/PMD/GetAgileFile?itemNumber=FIR2100137&fileName=DCS6100LHAx_FW103B03.bin&fileSize=1167392.0;756811.0;)<br>

If those links fail, then you can also find the firmware via their GPL portal, page 5 as of writing:<br>
	https://tsd.dlink.com.tw/downloads2008list.asp?t=1&Category=Product%20Data%20II%3EIP%20Surveillance%3EIP%20Cameras&pagetype=G

## The long story:
As usual, onboarding the device was painful. It required full internet access to complete the process and a mydlink account. My device wasn't new so it needed to be reset. 

The quickstart guide[^0] says: 
	"Reset and reinstall your device. Use a paperclip to press the recessed Reset button and the LED will turn solid red"

However: if you do so when the device is turned on it will instead go into recovery mode - allowing direct upload of firmware via a recovery portal as outlined above. What the guide doesn't say is that to initiate a factory reset you need to press the recessed reset button *while* you connect the power.

After onboarding I discovered the device wouldn't function without a working internet connection and also didn't expose anything to do with RTSP. Unless you are d-link. In Which case it happily streams RTSP offsite where they kindly allow us peons to stream it via their servers in an app or web browser. 

I initially attempted to gain root access via the hardware pins[^1]. 
	
Remove the single black screw below the micro-usb power port. Squeeze the sides of the device and work the back off. Remove the remaining three silver screws. The board can now be removed from its chassis.

There are three obvious, but very tiny through holes next to the usb port. The hole next to the usb port is ground. Middle pin is TX. End pin is RX. I removed and trimmed some metal prongs from a female pcb connector, and carefully soldered them into the through holes.

I couldn't log in as root using any of the available pins or passwords and the latest firmware gives a lot less output.

Next I tried to use the instructions from bmorks defogger[^2] to get a root console. This failed because it couldn't find /bin/sh or /bin/bash. I don't know enough about this stuff to push any further so on a whim I decided to see if I could find old versions of the firmware.

As luck would have it I did, direct from D-Link. Not only that but the recovery mode allowed me to downgrade the firmware. Earlier versions have much more verbose output from their RTSP server and I was eventually able to figure out a working stream URL.

The Taiwanese D-Link support portal also links to versions of the firmware[^3]

[^0]: https://media.dlink.eu/support/products/dcs/dcs-6100lh/documentation/dcs-6100lh_qig_reva1_1-00_eu_multi_20201102.pdf
[^1]: https://github.com/wuseman/DLink_6100LH/
[^2]: https://github.com/bmork/defogger#u-boot
[^3]: https://www.dlinktw.com.tw/techsupport/ProductInfo.aspx?m=DCS-6100LH
