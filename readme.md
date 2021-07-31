<b>Update</b>, I also now have a Eufy robovac 30C, these IR commands work with the 30C as well.

<h1>Welcome to my latest project</h1> Taking a Eufy robovac 11S and integrating it with Homeassistant, with a little help from an Wemos/Lolin mini d1 clone, and of course ESPHome.

<br>The robovac is well designed, very modular, and easy to take apart.

Being the 11S this is not a smart vac, but instead more in line with the original robot vacuum cleaners which just bumble around the house until its cleaned, I’m overall very impress with it out the box, it works very effectively, however I don’t like having lots of IR remote controls, and would rather have this robot integrated with Homeassistant, so I figured put a basic ESPHome IR blaster inside to send the commands, and maybe get some feedback via the LED light(s) in the top button.


After opening up the mainboard is nice and visible, including what appears to be pins for a ‘serial wire debug’ interface, now using SWD on an arm chip is well beyond my realms of experience, but it does provide a handy source of 3.3V and ground pins, which remain live all the time the main power switch (underneath the robot) is on.
![mainboard](\pictures\main_board.jpg)

Probing the top LED board, it seems there are actually 3 LED lights there, one Red, one Orange, and One blue, and 3 different wires to trigger each one, which is great news as we can use that to provide some feedback to our ESP device.
We’ve got 3 LED trigger wires which also run at 3.3V, I guess the main arm chip is 3.3V internally, which is great as now we can provide some feedback.
![led board](\pictures\top_led_panel_wires.jpg)

Sadly I could find no trace of a standard UART interface, looking at some disassembly’s of Eufy Robovac 30c and 15c, both models sharing the same chassis as our 11s but having WiFi, it seems the WiFi modules on those two are on a separate board with TxRx pins, so I may try modding one of those in the future.  (update and now I have [link](https://github.com/sid-the-squid/eufy_robovac_30c))

Anyway back to our 11S, so we’ve found our power source, I found a nice mounting point for our mini d1, at the back near the main power switch (Edit; this is where the wifi antenna on the 30C lives)
![minid1 and IR led](\pictures\mini_d1_and_led_installed.jpg)

On the ESP mini d1, we have
* 3.3V, GND
* D6 for Red LED
* D7 for Orange LEd
* D8 for Blue LED
* D2 for our IR LED

On the LED board I used
* Brown wire for D8 (Blue)
* Orange for D7 (Orange LED)
* Yellow for D6 (Red LED)
![mini d1 wires](\pictures\mini_d1_wires.jpg)

You’ll see in the wiring on the mini d1, I connected 7 wires, instead of the 6 we actually require, I had it in my mind originally to have an IR Receiver as well as a transmitter, but as the robovac never actually talks back to the remote control, the receiver part is pointless.


Sadly these wires alone were not long enough to reach the mini d1 in the position I placed it, so I had to connect a further 3 wires, brown to blue, orange to purple, and yellow to gray, apologies for the confusion, I really should buy more assorted lengths of wire, oh my.
![wires from LED](\pictures\front_view.jpg)

For our IR LED transmitter we don’t really need or want any great range, hence we can run the LED at 3.3v straight from our GPIO pins, no need for a 5V supply or a transistor to make it work.

The actual IR command codes need to be captured with ESPHome, as I already have an ESPHome based IR Blaster, I used that to capture the raw command codes, there are many guides on how to build one of these, so I won’t cover it here, and I’m assuming, perhaps erroneously, that all the Eufy Robovac 11s use exactly the same IR commands, I can confirm the 11S IR commands work for the 30C as well, so I would assume would work for all 11S/30C models, and for the 15c too.

With the codes captured and our wiring completed, I created the ESPHome yaml with all the commands as virtual switches, and two text_sensor’s for feeding back the robot status and power levels.

I found the blue LED could be a simple binary_sensor, but the Red and Orange LED’s could not as they pulse, I did originally try using a ESPHome pulse_width sensor, but it proved inappropriate, instead I settled on a pulse_meter sensor and that worked the way I needed it to.

I also found out the start commands, (auto start, edge mode, spot clean, single room) actually also set the vacuum power level as part of the command, so I captured all at max power level, my house is small so I don’t need the battery savings of Boost IQ, and found ‘standard’ power level a bit too low.
I captured almost all the commands (at max_power level), except the timeset, and schedule set commands, as not sure how they breakdown, and with Homeassistant automations I really don’t need them.
![my HA controls](\pictures\HA controls.PNG)


Possible future ideas.

Now perhaps if one was being really fancy, one could also capture the error beeps too, however I figured that was overkill, and on error its best to simply check the robot.
I also considered the possibility of putting an ESPCam on the front behind the bumper bar, as I figured that would be fun, but that would need some actual cutting and might interfere with the robots distance sensors (basic as they are).

Either way have fun modding your robovac, hope you found this helpful.
