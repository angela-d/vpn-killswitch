# VPN Killswitch for Torrents on Linux Desktop
When you're torrenting cooking recipes for your grandmother, you deserve your right to privacy.  It isn't your ISPs business what type of cookies she likes to bake.

This script will bind to your torrent application and terminate it's process if your VPN tunnel is lost.

I've only tested it with Deluge on Gnome desktop on Debian, but it should work with any torrent client in any desktop environment (with minor adjustments).

This script can be *easily* modified to work with any application you want to bind to a particular network interface.

In most circumstances, iptables would be the easiest method to bind applications to the desired network interface, but based on my trials, the desktop gtk version of Deluge does not play nice with it, so this method is application-based binding to the interface, as an alternative.

## Pre-requisites
* This script will not work in Windows.  Pick a [cooler operating system](https://www.debian.org/distrib/) to use it.
***
- ifconfig

Check for it's existence in your terminal, by running:
```bash
whereis ifconfig
```
If you don't see a path returned, you'll need to install the net-tools package (this command is for Debian-based Linux systems.  If you're running a different distro/operating system, research what package offers ifconfig and install it):
```bash
apt update && apt install net-tools
```
***
- The user that runs the torrent client needs sudo permissions.

Launch the torrent application and then run the following in your terminal (replace *deluge* for the name of your torrent client -- also take note of the application launcher that the torrent client runs as; in my case, it's *deluge-gtk*):
```bash
ps aux | grep deluge
```
Returns the following output:

> *angela*    2913  0.0  0.0  11176  2988 tty1     S+   05:07   0:00 **deluge-gtk**

For my situation, **angela** needs sudo permissions.

You can double-check (or add) permissions by running:
```bash
visudo
```
And look for **NOPASSWD** alongside the user your torrent client runs as; if such a line doesn't exist, add it beneath `root    ALL=(ALL:ALL) ALL`:
```bash
angela ALL=(ALL) NOPASSWD: ALL
```
(replace *angela* for your user that runs your torrent client (remember earlier when you ran the `ps aux` command?))
***
### Once these requirements are met, you're ready to use the VPN Killswitch.

* Download/clone the **vpn-check** script somewhere.. I put mine in /home/angela/.config/vpn-killswitch/

### Use Git to install the VPN Killswitch to ~/.config
```bash
git clone https://github.com/angela-d/vpn-killswitch.git ~/.config/vpn-killswitch
```

### Bind VPN Killswitch to Your Torrent Application
This method utilizes the desktop application launcher; which will be different if you're running the torrent client as a headless application.
- See if you have an existing launcher:
```bash
ls -l ~/.local/share/applications
```
Output will look similar to the following, if you have one already:
```bash
-rw-r--r-- 1 angela angela 421 Oct 21 04:34 deluge.desktop
```

If you don't have one, you might be launching your application differently than this script was tested with, so you'll probably have to tweak it a bit to get it binded.

### Open the existing .desktop file for your torrent client
Example command.  As always, adjust the specific path relative to your system:
```bash
pico ~/.local/share/applications/deluge.desktop
```

Look for this line (replace deluge.gtk for your torrent client's executable):
```bash
Exec="deluge-gtk %U"
```

Change that line to:
```bash
Exec=bash -c "deluge-gtk %U;  /home/angela/.config/vpn-killswitch/vpn-check"
```
*If your vpn-killswitch directory wasn't saved to your /home/user/ folder, modify the path in my example to suit your environment.*
***
# Mandatory Config
Before you can use the script, you must modify the config variables to reflect your torrent client.
- Open the vpn-check file and modify the following variables (if necessary):

```bash
IFCONFIG=/sbin/ifconfig   # whereis ifconfig in your terminal will tell you where ifconfig's path is
INTERFACE=tun0            # if you're not sure what your tunnel interface is, run ip a while connected to a vpn.  wlan0 = wifi eth0 = ethernet, etc. (Names will vary by interface type)
CLIENT=deluge-gtk         # the torrent client path exposed in the ps aux command from earlier
```
***
## That's it, you can now torrent without fear of leaking info to seeders or ISPs due to a lost VPN connection!


### Optional Debug Mode
If you want to tweak your VPN Killswitch, you can enable debug mode to get verbose logging and follow the script's movements.
- Open `vpn-check`
- Change `DEBUG=0` to `DEBUG=1`
- Launch your torrent application
- View log output at `/var/log/[your torrent-client]-kill.log`

*Be sure to disable debug mode when you're done with it, or you'll find your log directory eating space on your hard drive!*

**In normal (non-debug) mode, torrent client terminations initiated by VPN killswitch are always logged to `/var/log/[your torrent-client]-kill.log`.**

### Known Bugs
If you launch your torrent application while offline (ie. no internet connection at all), even though VPN Killswitch is triggered, it doesn't carry out its duties.  It needs to be launched on a live connection -- it starts working right away, so this bug is really only limited to testing scenarios.

**Launch your torrenting application *after* your VPN has been turned on.**

# Thoroughly test this setup before you leave it unattended.
ie. Unplug your ethernet or disable your wifi while it's running, switch off your VPN (doing so you run the risk of exposure, so if possible, set your wifi/ethernet DNS to 127.0.0.1 to loopback, rather than reach your seeders while testing)

Debug behaves differently than non-debug!  If you encounter bugs or issues, [please submit a bug report](https://notabug.org/angela/vpn-killswitch/issues) detailing the torrent client/application and operating system you're using, as well as:
- Output from `ps aux | grep [torrent client name]`
- and `ps S`
