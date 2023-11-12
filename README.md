So you found a Compy EIH4 Smart Assistant in the wild, and you're trying to figure out how to make use of it.

Tools needed:
* The native power supply.  Note that it is originally rated 12V 2A...
* A small, sharp knife, or art knife, for lifting a label (alternatively a plastic spudger with a fine tip)
* Putty for serial line emulation of a TTY
* A USB cable with 
* Wifi connectivity nearby.  2.4Ghz is known to work.
* Your favorite SSH client (you can re-use putty here but there are others)

Unfortunately the only instructions I have at this time will be for Windows.  I will need to have Linux users chime in at a later time.

NOTE: while most of these units came with no warranty, I will caution that you will be making a minor modification to the unit you have.  If you don't feel comfortable I can only suggest you don't proceed.  It is, after all, a more or less unrealized product.

You most likely have the unit in front of you, powered on.  You saw the ring-circle of lights at the top during bootup, and it went dark.  So now what?  Well, the unit I have is running Ostro Linux with a busybox environment, I'm not sure but it seems to be a variant of Yocto, so your mileage may vary with these instructions.  This might be enough to work something out.

First we need to get into the unit itself.  For this, you will need to peel back a sticker that runs along the long spine of it.  This sticker covers various ports.  You'll know you have the right one when it has a hole punched in the sticker for the power port.

Using a spudger or small knife, carefully (!) peel back the sticker to expose the ports underneath.  It is stuck on their fairly well so be careful and deliberate.  When the sticker is removed, you will expose an SD card port, a USB type A port, and a microUSB port.  We are intrested in the later of the 3, as it is really a serial port in disguise.

Plug your USB adapter cable (any 2.0 cable with Type A to microUSB should work) into your machine, then plug the cable into the microUSB port.
* For Windows, you will get a prompt stating that a serial driver is being installed.  This is necessary and expected.

Once the serial cable is connected, and the correct serial port found, you will need to create a profile for terminal emulation.  Open Putty and proceed to create a profile with the following parameters:

* Session Section:
  * Connection type: Serial
  * Serial Line: COM7 (you will specify this in the next section)
  * Speed: 115200 (you will specify this in the next section)
* Connection Section:
  * Serial Subsection:
    * Serial line to connect to: COM7 (windows)
    * Speed (baud): 115200
    * Data bits: 8
    * Stop bits: 2
    * Parity: none
    * Flow control: RTS/CTS
* Terminal Section:
  * Keyboard Section:
    * Backspace: Ctrl-?
    * Function keys: Linux
* Window Section
  * Lines of scrollback: 2000 (handy for debugging purposes)
  * Translation Section:
    * Remote character set: UTF-8

Save these settings for later use.  I simply named mine "Compy-EIH4".

Once you have the settings, open them to start the command line.  You should see a blank window. If everything went well, you should be able to press enter and see a command line prompt.

Let's look around and see what is here:

    uname -a ; cat /proc/cpuinfo | grep "model name" ; echo ; free ; echo ; df ; iwconfig

# Wireless Connectivity

These devices come with Wifi and Bluetooth.  Wifi means the possibility of SSH access, and with it, remote file transfers.  First we need to configure wpa_supplicant to support the local Wifi access point.  We can store this permanently by appending the wpa_supplicant.conf file.  Be sure to use the double-greater-than and not just a single, a single one will wipe the file!  *This configuration change only needs to be done once as it will be saved to the file system.*

    wpa_passphrase WLAN_NAME WLAN_PASSWORD >> /etc/wpa_supplicant.conf

Now you need to add some lines inside the file.  Be sure to do this inside the `network={` block, one entry per line.  You can do this with the `vi /etc/wpa_supplicant.conf` command (sorry if you don't have your favorite editor available, vi seems to be what shipped with it):

    scan_ssid=1
    proto=RSN
    key_mgmt=WPA-PSK
    group=CCMP
    pairwise=CCMP
    priority=10

These additionals help to show hidden APs, and sets the encryption type.

Now that the definition for the access point is set, you will want to bring up the wireless interface.  To bring up a temporary interface (which will disappear on reboot or power loss) enter the following:

    ifconfig mlan0 up && wpa_supplicant -B -i mlan0 -c /etc/wpa_supplicant.conf -D wext && sleep 15 && udhcpc -i mlan0

# Build tools

Let's see what we have available:

    python --version ; echo ; gcc -v ; echo ; make -v

