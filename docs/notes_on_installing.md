In Arch Linux when installing this module, brltty interfears with the creation a of ttyUSB0. The is that brltty is a program that provides access to the LINUX/UNIX console (when in text mode).

to see this run:

sudo dmesg | grep ch34x 

OUTPUT:
[ 284.973396] ch34x 3-4.2:1.0: ch34x converter detected
[  284.973894] usb 3-4.2: ch34x converter now attached to ttyUSB0
[  285.019333] usb 3-4.2: usbfs: interface 0 claimed by ch34x while 'brltty' sets config #1
[  285.019883] ch34x ttyUSB0: ch34x converter now disconnected from ttyUSB0
[  285.019895] ch34x 3-4.2:1.0: device disconnected

To get around this issue you can disable brltty in two ways:

#Removing udev rules
Because brltty uses udev rules to get permissions to mess with the TTYs without being root.
However you can disable these rules by overriding the rules shipped by your distro with "/dev/null":

for f in /usr/lib/udev/rules.d/*brltty*.rules; do
    sudo ln -s /dev/null "/etc/udev/rules.d/$(basename "$f")"
done
sudo udevadm control --reload-rules

#Disabling service
The BRLTTY service is launched by the brltty.path service. This service can be completely prevented from ever starting by running by doing the following:

$ sudo systemctl mask brltty.path
Created symlink /etc/systemd/system/brltty.path → /dev/null.

#Another Method


Shooting down a bit too much there. A much less invasive solution was contributed by user Blackisle to this post ("Arduino not working adter brltty update") on ArchLinux BBS.

brltty has a rule for idVendor=1a86, idProduct=7523, which is the same as the CH340 serial converter on my Mega clone.

You can see your device id by using lsusb to get a list of your devices (unplug your Arduino, run lsusb then plug in your Arduino and run lsusb again to see which device appears).

In my case:

Bus 003 Device 005: ID 1a86:7523 QinHeng Electronics CH340 serial converter

Take a note of the ID and then open the brltty rules file:

sudo nano /usr/lib/udev/rules.d/90-brltty-device.rules

Search through the file until you find the entry for your ID:

# Device: 1A86:7523
# Baum [NLS eReader Zoomax (20 cells)]
ENV{PRODUCT}=="1a86/7523/*", ENV{BRLTTY_BRAILLE_DRIVER}="bm", GOTO="brltty_usb_run"

Now comment out the line:

# Device: 1A86:7523
# Baum [NLS eReader Zoomax (20 cells)]
# ENV{PRODUCT}=="1a86/7523/*", ENV{BRLTTY_BRAILLE_DRIVER}="bm", GOTO="brltty_usb_run"

Save and close the file then reboot.

After the reboot the /dev/ttyUSB0 port was available again in the Arduino IDE.

