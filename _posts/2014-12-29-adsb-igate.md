---
layout: post
title: "APRS iGate and ADS-B plane tracker"
excerpt: "Building an APRS iGate and ADS-B plane tracker using Raspberry Pi and RTL SDR dongles."
categories: articles
tags: [ham-radio, adsb, rtl-sdr]
author: shaji
comments: true
share: true
image: adsb-igate-bg.jpg
---

## Building an APRS iGate and ADS-B plane tracker using Raspberry Pi and RTL SDR dongles

I have always been interested in SDR but never got a chance to really try it out. Luckily I was able to find a lot of resources on the Internet to get me started. A big thank you to all the open source contributors out there as I was able to build an APRS RX-only iGate and an ADS-B plane tracker very quickly.

Here are the items you would need:

1. RaspberryPi
2. 2 x RTL SDR dongles
3. USB powered hub
4. 4GB SD card
5. Wifi dongle (optional)

I started with a newly prepared SD card. You can download RASPBIAN (Debian Wheezy) OS image from http://www.raspberrypi.org/downloads/ and then follow the instructions at http://www.raspberrypi.org/documentation/installation/installing-images/README.md to write the image to the SD card.

### Prepare the Raspberry Pi
Login to the raspberry pi and make sure that you have Internet connectivity.

{% highlight js %}
sudo apt-get update
sudo apt-get upgrade
sudo raspi-config
{% endhighlight %}

A setup screen would be displayed when you run rasps-config. Using the setup screen expand the file system and configure your  time zone.

It seems that the newest kernel includes a DVB driver for the dongle as a TV receiver. We do not require this for our purposed so we will create a config file to blacklist it.

{% highlight js %}
cd /etc/modprobe.d
sudo nano blacklist-rtl.conf
{% endhighlight %}

Add the following lines to the new file:
{% highlight js %}
blacklist dvb_usb_rtl28xxu
blacklist rtl2832
blacklist rtl2830
{% endhighlight %}

### Install the dependencies
{% highlight js %}
sudo apt-get install git git-core cmake build-essential libusb-1.0-0-dev
sudo apt-get install qt4-qmake libpulse-dev libx11-dev patch pulseaudio
sudo apt-get install libtool autoconf automake libfftw3-dev
sudo apt-get install python-pkg-resources pkg-config
{% endhighlight %}
### Install RTL SDR driver
{% highlight js %}
cd ~
git clone git://git.osmocom.org/rtl-sdr.git
cd rtl-sdr
mkdir build
cd build
cmake ../ -DDETACH_KERNEL_DRIVER=ON -DINSTALL_UDEV_RULES=ON
make
sudo make install
sudo ldconfig
{% endhighlight %}
At this point, you should be able to run the rtl_test command. The command would find the devices, display supported gain values, and do some sampling. It should report everything is okay.

### Install multimonNG decoder
{% highlight js %}
cd ~
git clone https://github.com/EliasOenal/multimonNG.git
cd multimonNG
mkdir build
cd build
qmake ../multimon-ng.pro
make
sudo make install
{% endhighlight %}
### Install the kalibrate tool

Kalibrate, or kal, scans for GSM base stations in a given frequency band and uses GSM base stations to calculate the local oscillator frequency offset of the rtl-sdr devices.
{% highlight js %}
cd ~
git clone https://github.com/asdil12/kalibrate-rtl.git
cd kalibrate-rtl
git checkout arm_memory
./bootstrap
./configure
make
sudo make install
{% endhighlight %}
With kal it is a two step process. We find a strong channel first.
{% highlight js %}
kal -s GSM900
{% endhighlight %}
Once we have the strong channel number we can find the ppm.
{% highlight js %}
kal -c 34 -v
{% endhighlight %}
I was not able to use kal reliably to find the ppm offset in my area and used SDR# to manually find the ppm value.

### Install APRS iGate software
{% highlight js %}
cd ~
git clone https://github.com/asdil12/pymultimonaprs.git
cd pymultimonaprs
sudo ln -s /usr/bin/python2.7 /usr/bin/python2
sudo python2 setup.py install
{% endhighlight %}
Configure the startup script
{% highlight js %}
sudo cp pymultimonaprs.init /etc/init.d/pymultimonaprs
sudo chmod +x /etc/init.d/pymultimonaprs
sudo update-rc.d pymultimonaprs defaults
{% endhighlight %}
### Generate APRS-IS password
{% highlight js %}
cd ~/pymultimonaprs
./keygen.py CALLSIGN
Key for CALLSIGN: 12345
{% endhighlight %}
Update the pymultimonaprs configuration file with your callsign, passcode, gateway, freq, ppm, gain, lat, lng, and device index. Device index is important when multiple RTL SDR dongles are plugged in. Enter the index number of the dongle you would like to use with pymultimonaprs. You should be able to find the device index by using the rtl_test command.
{% highlight js %}
sudo nano /etc/pymultimonaprs.json
{% endhighlight %}

### Test and decode APRS packets
{% highlight js %}
rtl_fm -d 0 -f 144390000 -s 22050 -p -103 -g 42.0 - | multimon-ng -a AFSK1200 -A -t raw -
{% endhighlight %}
I was not able to pick up any stations in my apartment on the ground floor. However, I was able to decode packets from my APRS device.

### Start pymultimonaprs
{% highlight js %}
sudo /etc/init.d/pymultimonaprs start
{% endhighlight %}
### Install dump1090 to decode ADS-B transmissions from aircraft
Dump1090 is a simple Mode S decoder for RTLSDR devices
{% highlight js %}
cd ~
git clone git://github.com/MalcolmRobb/dump1090.git
cd dump1090
make
{% endhighlight %}
### Start dump1090
{% highlight js %}
cd ~
cd dump1090
./dump1090 --net --aggressive --ppm 43 --device-index 1
{% endhighlight %}
### References
1. [http://www.kubonweb.de/?p=130](http://www.kubonweb.de/?p=130)
2. [http://www.algissalys.com/index.php/amateur-radio/88-raspberry-pi-sdr-dongle-aprs-igate](http://www.algissalys.com/index.php/amateur-radio/88-raspberry-pi-sdr-dongle-aprs-igate)
3. [http://www.satsignal.eu/raspberry-pi/dump1090.html](http://www.satsignal.eu/raspberry-pi/dump1090.html)
