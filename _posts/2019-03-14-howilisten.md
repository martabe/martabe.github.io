---
title: 'How I listen to my favourite radio'
date: 2019-03-14
permalink: /posts/2019/03/howilisten/
tags:
  - Raspberry Pi
  - Volumio
  - HiFi Berry
  - Radio Popolare
---

I moved out of my country more than one year ago, and one of the things I miss the most is being able to easily listen to my favourite radio ([Radio Popolare](www.radiopopolare.it)). Since the radio has a web streaming, I figured out I could use my Raspberry Pi to play it. Quite a boring story up to now. The cool (at least for me) part of it, is that I managed to plug a rocker switch to the RPi to have an easy and quick ON/OFF button to play the radio. The RPi runs [Volumio](volumio.org), a very cool music player originally developed by Michelangelo Guarise. This means it can be used to play Spotify, music from a USB stick, AirPlay, and maybe other cool things I don't know about.
So here comes a report of what I did exactly, in case somebody wants to build something similar.


Disclaimer
======
I am not a programmer, nor an electronic engineer. I was a bit lazy in setting up the case. The final result doesn't work perfectly: sometimes when turning the switch to OFF, it actually sends the ON command. I suspect it might be a matter of loose connections.


Material
======
* [Raspberry Pi 3 Model B+](www.raspberrypi.org/products/raspberry-pi-3-model-b-plus)
* [HiFiBerry DAC+](www.hifiberry.com/products/dacplus)
* [Joy-it rp port-doubler](www.conrad.ch/fr/joy-it-rb-port-doubler-1-pcs-1720611.html) (Link to a Swiss reseller)
* 3 Jumper wires and 2 resistors (1 x 1k&#937;, 1 x 10k&#937;) from [this Arduino kit](wiki.seeedstudio.com/Sidekick_Basic_Kit_for_Arduino_V2)
* [the rocker switch](www.conrad.ch/fr/interrupteur-a-levier-1-x-offon-sci-701011-250-vac-15-a-a-accrochage-1-pcs-701011.html)
* a RCA stereo cable ending with your favourite connector (a small jack in my case, [like this](www.amazon.com/Adecco-LLC-Stereo-Female-Adapter/dp/B01ET3Y2SO/ref=sr_1_22?keywords=rca+to+jack&qid=1552585050&s=electronics&sr=1-22))
* a power supply for the Raspberry Pi
* a Micro SD card


Software
======
* Volumio version 2.452 [Volumio](volumio.org)
* a [python script](raw.githubusercontent.com/martabe/Pi-Fi/master/rocker.py) that deals with the switch


How to put things togheter
======


Software part
------


### The OS: Volumio
First thing, download and install Volumio on the Micro SD card. You can find the instructions on [Volumio's website](volumio.org/get-started/), just scroll down to find the guide.

### Interacting with Volumio, API requests
The system I use to turn on and off the radio is rather simple; I use one API request to ask Volumio to play a specific playlist and another to stop anything that might be playing. API requests to Volumio interface are [documented here](volumio.github.io/docs/API/API_Overview.html).

I use a playlist to be able to play the webradio from API. I had found this idea somewhere on the web, but cannot find the page any longer. Anyway the setup goes as follows:
1. Access Volumio interface
2. Go to Browse, then WebRadio, and find your WebRadio
3. On the far right, click on the three dots and click on "Clear and Play" (beware, this will clean your current queue)
4. Go to the Queue page, and save the queue as a playlist (top right). I called my playlist "rp_autoplay".
...This allows us to use the API request
...`http://127.0.01:3000/api/v1/commands/?cmd=playplaylist&name=rp_autoplay`
...to play the radio that is saved in the playlist "rp_autoplay".
...The request for stopping anything from playing is:
...`http://127.0.0.1:3000/api/v1/commands/?cmd=stop`
...Note that the IP address (`127.0.01`) can be replaced by `localhost`.

### Copy and enable rocker.py
To deal with the switch I wrote a python script that checks for changes in the state of the switch and subsequently sends the API requests to Volumio.

The script can be found [here](github.com/martabe/Pi-Fi/blob/master/rocker.py).

The module RPi.GPIO is needed and was not installed on my Volumio version so I had to install it manually:
1. Enable ssh on Volumio by following the [guide in this page](volumio.github.io/docs/User_Manual/SSH.html).
2. `ssh` to the Raspberry and install `gcc` and `RPi.GPIO` following [this guide](zasieczny.wordpress.com/2015/02/12/volumio-installing-gpio-python-module-to-control-amplifier/). Note that in my case `/etc/apt/sources.list` was already set up for jessie.
3. `wget` or `scp` the python script to the Raspberry
4. Enable the script to run at boot by editing `/etc/rc.local`, by adding the line `sudo python /home/volumio/rocker.py &`
... Note that the path has to be consistent with the location of the script, and the `&` is essential.
... Michelangelo Guarise [says that we should use a different system](volumio.org/forum/problem-starting-python-script-via-etc-local-t5971.html) to have a script run at boot, but I found that `rc.local` works as well.

