# [Security Growler](https://pirate.github.io/security-growler)  <img src="https://pirate.github.io/security-growler/alert.png" height="20px"/>  [![Build Status](https://img.shields.io/github/stars/pirate/security-growler.svg)](https://github.com/pirate/security-growler) [![Twitter URL](https://img.shields.io/twitter/url/http/shields.io.svg?style=social)](https://twitter.com/thesquashSH)

This menubar app for OS X will notify you via Notification Center (or Growl) when various security events occur ([see list](https://github.com/pirate/security-growler#documentation)).

<img src="http://pirate.github.io/security-growler/screenshots/portscan_event.PNG" width="45%"/>
<img src="http://pirate.github.io/security-growler/screenshots/vnc_event.PNG" width="45%"/>

It's very useful if you're paranoid about people trying to hack into your computer.  Or... if you simply like having information about people using your computer's resources.

It's extremely lightweight, the app is 3MB including the icon, with <0.01% CPU and <15MB of RAM used when running.
It's easily extensible in Python, you can add parsers that detect new TCP connetions or poll logfiles.
You can even forward alerts as push notifications to your iOS devices using [Prowl](http://prowlapp.com/).


## Install:
1. Download and run [Security Growler.app >>](https://github.com/pirate/security-growler/raw/master/Security%20Growler.app.zip) (dark mode)
2. Click on the menubar icon once to start detecting events.

<img src="http://pirate.github.io/security-growler/screenshots/menubar_2.PNG" width="45%"/>
<img src="http://pirate.github.io/security-growler/screenshots/menubar_1.PNG" width="45%"/>

Download [Security Growler Light.app](https://github.com/pirate/security-growler/raw/master/Security%20Growler%20Light.app.zip) if you don't use OS X Dark Mode.
If you prefer [Growl](http://growl.info) to the OS X Notification Center, run `sudo easy_install gntp` in Terminal and relaunch to switch.

Note: the app **must** be run under an account with admin-permissions.  It will not function under a standard-permissions account, as it needs access to several root-owned logfiles to be of any use.

## It can do cool things like:

**Alert you of attempted and succesfull SSH logins:**

<img src="http://pirate.github.io/security-growler/screenshots/ssh_fail_event.PNG" width="40%"/>
<img src="http://pirate.github.io/security-growler/screenshots/ssh_key_event.PNG" width="40%"/>

**Notify you of incoming & outgoing TCP connections: FTP, VNC, SMB, MySQL, etc.:**

(using [less RAM](https://github.com/pirate/security-growler#background) than Little Snitch)  

<img src="http://pirate.github.io/security-growler/screenshots/vnc_event.PNG" width="40%"/>
<img src="http://pirate.github.io/security-growler/screenshots/connection_event.PNG" width="40%"/>

**Notify you whenever a command is run with `sudo`:**

<img src="http://pirate.github.io/security-growler/screenshots/sudo_context.PNG" height="350px"/>

**Let you know when you're being portscanned:**

<img src="http://pirate.github.io/security-growler/screenshots/portscan_context.PNG" height="350px"/>

[More Screenshots...](https://github.com/pirate/security-growler/tree/gh-pages/screenshots)


## Documentation:

The currently working alert types are:

 * SSH
 * FTP
 * SMB
 * AFP
 * MySQL
 * PostgreSQL
 * iTunes Sharing
 * sudo commands
 * [Ostiarius](https://objective-see.com/products/ostiarius.html)
 * port-scans (e.g. if you're on the receiving end of nmap)
 * VNC (detailed alerts require app is run as root)

TODO:
 * keychain auth events (`/var/log/authd.log`, `/var/log/accountpolicy.log`)
 * new listening sockets under port 1000 opened
 * Wifi events?

### Config:

Settings are changed by editing a text file `settings.py`, accesible via the menubar dropdown item 'Settings...'.

**To enable or disable alert types:**

You can enable and disable certain alerts by editing the `WATCHED_SOURCES` section of the file.
Add or remove event sources on the left (either port numbers or logfile paths), and put the parser names you want to enable for each source on the right.  Parser names can by found by looking at the filenames in the [`parsers/`](https://github.com/pirate/security-growler/tree/master/parsers) folder.

```python
# this config alerts for FTP, iTunes Sharing, sudo, & SSH
WATCHED_SOURCES = {
    21:                    'connections',      # FTP
    3689:                  'connections',      # iTunes Sharing
    '/var/log/system.log': ('sudo', 'ssh'),    # sudo & ssh
}
```

**To enable or disable alert methods, such as Notifcation Center or Growl:**

Change the `LOGGERS` section to suit your needs.

```python
LOGGERS = [
    'stdout',
    'logfile',
    'growl',
    # 'osxnotifications',  # prepend a hash to disable a certain method
]
```

**To change notification preferences:**

Change `POLLING_SPEED` to make the app update more or less frequently (2-10 seconds is recommended).

Change the `INFO_` and `ALERT_` items to modify properties such as alert sounds, icons, and text.


## Developer Info:

This app is composed of 3 main parts: `sources`, `parsers`, and `loggers`.

 * [`sources`](https://github.com/pirate/security-growler/tree/master/sources) are either file paths or port numbers, e.g. `/var/log/system.log` or `5900`
 * [`parsers`](https://github.com/pirate/security-growler/tree/master/parsers) e.g. `ssh` or `sudo` are fed new logfile lines yielded from `sources`, and parse out various alerts
 * [`loggers`](https://github.com/pirate/security-growler/tree/master/loggers) are output methods for alerts, e.g. `stdout`, `osxnotifications`, or `growl`

The main runloop is in [`growler.py`](https://github.com/pirate/security-growler/blob/master/growler.py), it reads lines out of the sources, passes them through parsers, then dispatches alerts before waiting a short delay and then looping.

The [menubar app](https://github.com/pirate/security-growler/tree/master/Security%20Growler.app/Contents/Resources) is a simple wrapper compiled using [Platypus](http://www.macupdate.com/app/mac/12046/platypus).  `Security Growler.app` is packaged with copies of `growler.py` and all the other files it needs.
To make changes to the app, change the files you need, test using `sudo python growler.py` and `sudo ./menubar.sh`, then re-run Platypus to generate a new app.

The menubar app works by displaying the output of `menubar.sh`, and spawning a `growler.py` agent in the background to write new events to a logfile.
See [`menubar.sh`](https://github.com/pirate/security-growler/blob/master/menubar.sh) for more details.

The python Foundation library that provides access to OS X API's like notification center is not available by default for python3.5, so for the moment only 2.7 is supported.

## Background:

I was tired of not being able to find an app that would quell my paranoia about open ports, so I made one myself. Now I can relax whenever I'm in a seedy internet cafe or connected to free Boingo airport wifi because I know if anyone is trying to connect to my computer.

[Little Snitch](https://www.obdev.at/products/littlesnitch/index.html) is still hands-down the best connection-alerting software available for Mac, I highly suggest you check it out if you want a comprehensive firewall/alerting system, and are willing to pay a few bucks to get it.  Security Growler is centered around parsing logfiles for any kind of generic pattern, not just monitoring the TCP connection table like Little Snitch.  For example, my app can alert you of `sudo` events, keychain auth events, and anything else you can think of that's reported to a logfile.  This app is significantly more lightweight than Little Snitch, it comes in at <15mb of RAM used, because it aims to solve a simpler problem than Little Snitch.  This app is not designed to *prevent* malicious connections, that's what firewalls are for, it's just meant to keep an unobtrusive log, and alert you whenever important security events are happening.  The more informed you are, the better you can protect yourself.

This app is meant for developers who frequenly run services that are open to their LAN, and just want to keep tabs on usage to make sure they aren't being abused by some local script kiddie.  Since the target audience is developers, I opted to leave some parts a little less user-friendly, such as the `settings.py` config system.  It's also just plain fun to enable lots of alerts types if you like to see every little detail of your computer's operation.

Feel free to submit a [pull-request](https://github.com/pirate/security-growler/pulls) and add a [new parser](https://github.com/pirate/security-growler/blob/master/parsers/vnc.py) (e.g. try writing one for nginx http-auth)!

Basic Linux support will be finished soon, in the meantime check out a similar project written by [@benjojo](https://github.com/benjojo): [PushAlotAuth](https://github.com/benjojo/PushAlotAuth), it uses the [PushALot](https://pushalot.com/) push-notification platform.

Also check out our growing list of community-shared [useful Mac menubar apps](https://github.com/pirate/security-growler/issues/32)!

## License:

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, pulverize, distribute, synergize, compost, defenestrate, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

If the Author of the Software (the "Author") needs a place to crash and you have a sofa available, you should maybe give the Author a break and let him sleep on your couch.

If you are caught in a dire situation wherein you only have enough time to save one person out of a group, and the Author is a member of that group, you must save the Author.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO BLAH BLAH BLAH ISN'T IT FUNNY HOW UPPER-CASE MAKES IT SOUND LIKE THE LICENSE IS ANGRY AND SHOUTING AT YOU.



<img src="http://pirate.github.io/security-growler/screenshots/menubar_3.PNG" width="100%"/>
