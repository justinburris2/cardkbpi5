# Raspberry Pi Card Keyboard

A script for using the M5Stack CardKB with Raspberry Pi 5

Items I used in the build:
<ul><li>Monitor, </li>
<li>Wirelesss Keyboard and mouse, </li> 
<li>CardKB V1.1, </li>
<li>Jumper Wires M-F (Male to Female)</li></ul>

First i'd like to say thanks to ian-antking#4 and ViktorWalter

I am going to use some of what they used right here in this git, and rearrange it the way that i did it to git it to work. (git it?) hehe

Next let me say I did all this using TwisterOS but since it uses sudo it should not be much of an issue from say something like Ubuntu or Raspibian or anything else Debian.

# Lets start here:

## Setting your pi to us layout

In order for buttons to return the correct symbols, the keyboard layout will need to be set to us on you pi. You can do this by running:

```bash
sudo nano /etc/default/keyboard
```

and changing `XKBLAYOUT` to `us`:

```
# KEYBOARD CONFIGURATION FILE

# Consult the keyboard(5) manual page.

XKBMODEL="pc105"
XKBLAYOUT="us"
XKBVARIANT=""
XKBOPTIONS=""

BACKSPACE="guess"
```
For me this was already done, so if it is already done you will be fine.

## Enable i2C
The cardKB communicates over i2C, make sure this is enabled on your raspberry pi. You can find a tutorial on how to do so [here](https://www.raspberrypi-spy.co.uk/2014/11/enabling-the-i2c-interface-on-the-raspberry-pi/).

## Connect CardKB to Raspberry Pi

Connect the wires on the CardKB JST connector to the appropriate pin on the Raspberry Pi. 

![CardKB/Raspberry Pi i2C connection](https://github.com/ian-antking/cardkb/blob/master/docs/wiring.png?raw=true)

You may need to improvise a connection solution with breadboard wires like so:

![Assembled a raspberry pi and hyperpixel](https://github.com/ian-antking/cardkb/blob/master/docs/assembled-pi-keyboard.jpg?raw=true)

I Used male to female jumpers matching the colors of the wires coming from the cardkb. Below are the pins i used.
<ul><li>Black Wire 3.3v/VCC CardKB - pin 1 Pi5</li>
<li>Red Wire SDA CardKB - pin 3 Pi5</li>
<li>Yellow Wire SCL CardKB - pin 5 Pi5</li>
<li>White Wire Ground CardKB - pin 9 Pi5</li>

## Install Software

Install smbus, python, synaptic and uinput:

```bash
sudo apt install python3 2to3
```

```bash
sudo apt install python3-full
```

```bash
sudo apt install python3-smbus
```

To install uinput I had to install synaptic package manager then do a manual search for uinput to install it. Everytime I would install uinput through the terminal I would get an error and a message stating it was externally managed. So far this is the only way I found that worked.

```bash
sudo apt install synaptic
```
Then 

open "synaptic package manager" and search for "python3-uinput" and install it.

Return to the Terminal window.

## Clone the Git Repo

clone this repository:

```
git clone https://github.com/ian-antking/cardkb.git
```

## Load the uinput module (i had to do modprobeuinput first)

You will need to load the uinput module to allow python-uinput to input key presses. You can check if it is loaded with:

```bash
lsmod | grep uinput
```

If nothing is displayed, then the module is not loaded. To load the module, run:

```bash
modprobe uinput
```
## Add Module to run automattically on startup

To load the module automatically on startup, run:

```bash
sudo nano /etc/modules
```
add `uinput` at the bottom of the file. Save and then reboot.



## Run the script and check buttons return expected characters:

```bash
sudo python3 cardkb &
```

By default, the python script listens to `/dev/i2c-1`, you can change this by adding an argument to the start command.

```bash
sudo python3 cardkb 11 &
```

## Add KB Service to run at startup (Run CardKB when Raspberry Pi starts)

We can use systemd to run the CardKB script as a service. To do so, create a unit file:

```bash
sudo nano /lib/systemd/system/cardkb.service
```

Add the following:

```
[Unit]
Description=Service for using CardKB with Raspberry Pi
After=multi-user.target

[Service]
Type=idle
ExecStart=/usr/bin/python3 /home/pi/cardkb

[Install]
WantedBy=multi-user.target
```

Likewise, if you are running cardkb on a i2c bus other than one, then you will need to add the bus number to the end of the `ExecStart` line like so:

```
...
ExecStart=/usr/bin/python3 /home/pi/cardkb 11
...
```

Save the file and exit. 

Additional info to add...

Note: I would like to Omit this part but if you need it, i will not leave it out. it was not neccessary for my setup.


This service file assumes that you have cloned the cardkb repo to /home/pi. If this is not the case, you will need to change the file path. 

```
...
ExecStart=/usr/bin/python3 /home/ian/cardkb
...
```

Finally
Now run the following commands to reload the systemctl daemon, enable the cardkb service and restart the pi:

```bash
sudo systemctl daemon-reload
sudo systemctl enable cardkb.service
sudo reboot
```

When your Pi restarts, your cardkb should be working, allowing you to log in.


