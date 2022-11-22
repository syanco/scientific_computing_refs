# Disabling touchscreen and trackpad in Ubuntu 20.04

I recently wiped an old (kind of broken) windows laptop and installed Ubuntu 20.04 to play around with.  This laptop has a cracked screen and a messed up trackpad.  Thus, I needed to disable them. Easy, right? No, Linux...

I messed around with a *lot* of ways to do this before finding a solution that works for me.  I' not going to comprehensively list these here but breifly: most solutions do not anticipate Wayland, and therefore suggest using `xinput` to solve the issue.  A couple also made use of udev rules because their machines mapped the input device through USB (or something).  None of that work on my Asus...

Instead, I created a `systemd` service that calls a script which basially diverts the input from both devices into a null script using evtest.  Kind of hack, but whtever, it works...

**Note:**  I have a like  C- understanding of this stuff so there could easily be mistakes here, there could be better ways, this is just what I did - mostly written out so I remember, less so as advice to others...

## The bash script

The bash sctipt I used is conceptually simple - it uses `evtest` with the `--grab` flag to "grab" the imput from the correct "event" (as in "event7" or whatever).  The trick, though is that devices can change event#s on reboot.  So this needs to be paired with a `grep` and some other manipulations to extract the event# given I know the device name - whichis actually the first step:

```
#display all the devices
cat /proc/bus/input/devices
```
Scroll through this looking for the names of the devices you want to intercept. In my case I needed "FTSC1000:00 2808:5120 Touchscreen" and "ELAN1300:00 04F3:3028 Mouse".

Now we can write commands to find the event# and use evtest to intercet their input (you may need to `sudo apt install evtest` first). I put the shebang at the top of the script to make it executable and I also added `&` at the ned of the lines to background the processes (i.e., keep them running but release the terminal).

**disable_touch**
```
#!/bin/sh

# Disable the touchscreen and trackpad on my computer.  Devices managed on Wayland so not as simple as simply turning off a service.
# This script figures out which device event the devices are on and diverts the input to a null file.  Meant to be called via systemd at startup.


sudo evtest --grab /dev/input/$(grep "FTSC1000:00 2808:5120 Touchscreen" -A 5 /proc/bus/input/devices | grep "Handlers=" | cut -d" " -f 3) > /dev/null &

sudo evtest --grab /dev/input/$(grep "ELAN1300:00 04F3:3028 Mouse" -A 5 /proc/bus/input/devices | grep "Handlers=" | cut -d" " -f 3) > /dev/null &

```

Save this file to `/usr/bin` and don't forget to `chmod +x /usr/bin/disable_touch`

##systemd service

Now we need to create and enable a systemd service that will automatically run at startup.

First create a new file:

```
[Unit]
Description=disable touchscreen

[Service]
Type=forking
ExecStart=/usr/bin/disable_touch

[Install]
WantedBy=multi-user.target

```

Breaking down the elements here:

The `Description` is just that, call it whatever.

Under `[Service]`, we tell it type `forking` which tells it to anticipate that the script called will background itself (remember the `&` at the end of each line???)
The `ExecStart` just supplies the path to the script we want to run (created above).

Finally, `WantedBy` just makes it available for the whole machine (I think).

We save this to `/etc/systemd/system/disable_touch.service`

The we need to enable it (we can also start it right then so no reboot required:

```
systemctl enable --now disable_touch.service
```

Touchscreen and trackpad should no longer work and that (dis)functionality should persist on reboot.  Bingo bango. 
