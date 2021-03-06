# Installing in virtualenv
* https://packaging.python.org/guides/installing-using-pip-and-virtual-environments/

### Setup

Start by making sure that you are running a fully updated Raspian install (Buster as of this time of writing), have git, python3, python3-pip and python3-dev installed, and have cloned the repo to `/~/HAT/pi-overwatch` eg:

```console
pi@pi:~$ sudo apt update
pi@pi:~$ sudo apt install python3 python3-dev python3-pip git
pi@pi:~$ mkdir ~/HAT
pi@pi:~$ git clone https://github.com/easytarget/pi-overwatch.git ~/HAT/pi-overwatch
; Alternatively, if you do not use git and have downloaded a zip or tarball, you should unpack it to: ~/HAT/pi-overwatch
pi@pi:~$ cd ~/HAT/pi-overwatch
```

### Install and Upgrade Requirements

In the cloned repo upgrade our local (user) modules of `pip` and `virtualenv`
```console
pi@pi:~/HAT/pi-overwatch $ pwd
/home/pi/HAT/pi-overwatch

pi@pi:~/HAT/pi-overwatch $ python3 -m pip --version
pip 18.1 from /usr/lib/python3/dist-packages/pip (python 3.7)

pi@pi:~/HAT/pi-overwatch $ python3 -m pip install --user --upgrade pip
pi@pi:~/HAT/pi-overwatch $ python3 -m pip --version
pip 21.3 from /home/pi/.local/lib/python3.7/site-packages/pip (python 3.7)

pi@pi:~/HAT/pi-overwatch $ python3 -m pip install --user --upgrade virtualenv
pi@pi:~/HAT/pi-overwatch $ python3 -m virtualenv --version
virtualenv 20.8.1 from /home/pi/.local/lib/python3.7/site-packages/virtualenv/__init__.py
```

Create the virtual environment and activate it
- The virtual environment will be located at `/home/pi/HAT/pi-overwatch/env`
- TL;DR: (Quick primer for the unitiated and curious):
  - A python virtual environment is, simply put, a complete and self-contained copy of python and all it's utilities, libraries, and packages.
  - It is installed into a folder (which you specify when creating it)
  - *Everything* is located in that folder, nothing gets installed to the machines OS, and you can do this as an ordinary user without needing root privileges.
  - This means that your virtualenv can have, say, a different version of python in it than the main 'os' version, either higher or lower, most useful when the OS python version is lagging behind the version you want to use.
  - You can also install python modules into the virtual environment directly, this allows you to use modules and module versions that are different or unsupported on your main OS
  - [This Video](https://www.youtube.com/watch?v=N5vscPTWKOk) and [This](https://www.youtube.com/watch?v=4jt9JPoIDpY) explain it quite well.

```console
pi@pi:~/HAT/pi-overwatch $ python3 -m virtualenv env

pi@pi:~/HAT/pi-overwatch $ source env/bin/activate

(env) pi@pi:~/HAT/pi-overwatch $ which python
/home/pi/HAT/pi-overwatch/env/bin/python

(env) pi@pi:~/HAT/pi-overwatch $ python --version
Python 3.7.3
```

Now we install/upgrade the requirements
```console
(env) pi@pi:~/HAT/pi-overwatch $ pip install --upgrade pip
(env) pi@pi:~/HAT/pi-overwatch $ pip install --upgrade wheel

(env) pi@pi:~/HAT/pi-overwatch $ sudo apt install rrdtool librrd-dev
(env) pi@pi:~/HAT/pi-overwatch $ pip install psutil schedule setproctitle rrdtool

; If you wish to control a gpio pin via a button or url you need to install RPi.GPIO
; - this is not necesscary if you just want to monitor (not control) pins.
; RPi.GPIO is (currently, november'21) broken on BULLSEYE unless you use a pre-release. sigh. 
(env) pi@pi:~/HAT/pi-overwatch $ pip install RPi.GPIO==0.7.1a4

; Only if you plan to use a BME280 Temperature/Humidity/Pressure sensor:
(env) pi@pi:~/HAT/pi-overwatch $ pip install adafruit-circuitpython-bme280

; Only if you plan to use a SSD1306 OLED display:
(env) pi@pi:~/HAT/pi-overwatch $ pip install adafruit-circuitpython-ssd1306
(env) pi@pi:~/HAT/pi-overwatch $ sudo apt install libjpeg-dev libopenjp2-7-dev libtiff-dev fonts-liberation
(env) pi@pi:~/HAT/pi-overwatch $ pip install image
```

Copy the `defaults.ini` file to `config.ini` and edit as required.
- See the comments in the file
- The default configuration is sufficient for testing, but screens, sensors and GPIO settings need to be enabled in the settings
- Some other parameters for the web server, logging and display can be set there too

If using either a screen, BME sensor or GPIO pin monitoring you must make sure the pi user is in the gpio group:
```console
pi@pi:~$ sudo usermod -a -G gpio pi
```

Then test run with:

```console
(env) pi@pi:~/HAT/pi-overwatch $ python overwatch.py
```
The file `overwatch.log` should be created in the pi-overwatch directory, and contain a startup log

Debug messages, errors, etc are printed to the console

The web server should be available on `http://<machines-address>:7080/` (or whatever is configured in the settings)

Note; If you want to leave the virtualenv at any time you can do so with `$ deactivate`, if you want to delete the virtualenv it is as simple as deleting the 'env' folder and all it's sunfolders.

## Set up as a service

Once you have everything installed, configured and tested by running on the console you should start running this as a system service. The OverWatch will then run automatically at boot and operate in the background like other services.

```console
pi@pi:~/HAT/pi-overwatch $ sudo ln -s /home/pi/HAT/pi-overwatch/OverWatch.service /etc/systemd/system/
pi@pi:~/HAT/pi-overwatch $ sudo systemctl daemon-reload
pi@pi:~/HAT/pi-overwatch $ sudo systemctl enable OverWatch.service
pi@pi:~/HAT/pi-overwatch $ sudo systemctl start OverWatch.service

pi@pi:~ $ sudo systemctl status OverWatch.service
??? OverWatch.service - OverWatch script for PI Hat
   Loaded: loaded (/etc/systemd/system/OverWatch.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2021-10-13 19:37:49 CEST; 10min ago
 Main PID: 20376 (python)
    Tasks: 2 (limit: 4164)
   CGroup: /system.slice/OverWatch.service
           ??????20376 /home/pi/HAT/pi-overwatch/env/bin/python /home/pi/HAT/pi-overwatch/overwatch.py

Oct 13 19:37:49 pi.easytarget.org systemd[1]: Started OverWatch script for PI Hat.
```

## Upgrading
Quick notes; to be expanded later as required, assumes you use git.
- go to the pi-overwatch repo
- git pull
- merge any changes from defaults.ini to config.ini
- stop the service
- start the service

Pip packages should not need upgrading any time soon if everything is working properly, but I'll investigate pip freeze and how to do this properly in the future as it becomes necessary.
