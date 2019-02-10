# Home assistant configuration

Uses:

- Chromecast
- Yeelights
- MQTT to get data from Zigbee protocol equipment (https://github.com/Koenkk/zigbee2mqtt)
- ZWave through 
- Broadlink RM Pro to control Blinds and LG TV.

Aside these I have installed

- Let's encrypt
- SSH Server
- MQTT Zigbeer addon https://github.com/danielwelch/hassio-zigbee2mqtt
- Pi Hole 
- 17 Track
- Recorder with MySQL connection and Grafana
- Node-RED


## Things I have found out

Here I'm putting all things I found on a way

### 1. How to get IR / RF codes 

#### 1.1. Learning codes
Broadlink RM Pro works very well with Homeassistant but the learning part starts failing after some minutes of starting Homeassistant and after couple of learning calls so it needs restarting. Also it need a mobile application to [ihc](https://play.google.com/store/apps/details?id=cn.com.broadlink.econtrol.plus&hl=en) enable the learning. The whole process can be found from here: https://community.home-assistant.io/t/guide-how-to-learn-broadlink-rf-codes/62119 .

#### 1.2 Modify codes

For TV I wanted to have separate on and off call but my remote only has power toggle button so I ended up finding the codes from internet. I did following steps:

1. After learning I had a long base64 string I tried to cut it as short as possible by trying it out through services. Then I
converted String to Hex with [Base 64 to Hex converter](https://cryptii.com/pipes/base64-to-hex) to start analysing. The Hex code looks like this:

> 2600 8801 0001 2b94 1312 1311 1436 1411 1411 1411 1411 1411 1436 1436 1411 1336 1436 1436 1436 1436 1411 1411 1436 1411 1312 
> 1312 1436 1436 1436 1436 1312 1436 1436 1435 1312 1312 1400 0528 0001 294a 1400 0c5c 0001 2a49 1400 0c5c 0001 2a49 1400 0c5c 
> 0001 2b48

2. From one forum which I forgot already, I found that follwoing guidance to read this:

*2600* is RM Pro's way to tell it's learned code and *8801 0001 2b94* is a starting part for LG television. Then we have three 16bit string. 

> 1. 1312 1311 1436 1411 1411 1411 1411 1411 1436 1436 1411 1336 1436 1436 1436 1436

> 2. 1411 1411 1411 1436 1312 1312 1312 1311 1436 1436 1436 1411 1436 1435 1436 1436

> 3. 1400 0528 0001 294a 1400 0c5c 0001 2a49 1400 0c5c 0001 2a49 1400 0c5c 0001 2b48

If we think that 1312 and close to that is 0 and 1436 is 1 then from the first line we get 0010 0000 1101 1111 Binary which to turning back to Hex is 20DF. From https://gist.github.com/francis2110/8f69843dd57ae07dce80 or https://gitlab.com/snippets/1690600
I can see that On/Off Power is 20DF10EF so first 16bit's matches. Then second part translated to 0001 0000 1110 1111.

Now I need to translate this to 20DFA35C (ON) and 20DF23DC (OFF). For ON I need to convert last 4 characters to binary and then use the same translation process get it back to hex which looks like this:

> 1411 1411 1436 1411 1312 1312 1436 1436 1436 1436 1312 1436 1436 1435 1312 1312

I will replace the second row with this and combine everything and then turn it back to Base64 and that's it.


### 2. Getting Eurotronic Z-Wave Plus SPIRIT Thermostat working with Homeassistant

At the time of buying these Eurotronic Spirit thermostats, Homeassistant didn't use the version of OpenZwave (which Homeassistant uses to control zwave devices) which included the configuration files for these thermostats. This made them work less reliably than I hoped.

Luckily people had figured this out before me (https://community.home-assistant.io/t/spirit-z-wave-plus/36685/13) so here were the steps I did. 

Since I had already paired these before noticing the issue, I needed to remove and add them after these changes so I would suggest either to do this before adding thermostats or removing and adding them after these changes.

#### 2.0 Check whether Spirit has configuration files already

Search for eurotronic configuration files

> find . | grep eurotronic

It should find files with following type of directory structure: /share/open-zwave/config/eurotronic/eur_spiritz.xml . Take a note where the directory of config is and if there already is a eur_spiritz.xml file or not. It may happen that this has been included to the current version of Homeassistant.

#### 2.1 Copy configuration files from OpenZwave repository

Copy the latest version of configuration files from OpenZWave repository https://github.com/OpenZWave/open-zwave/tree/Dev/config.
If you already had the spirit file, check whether the repository file is newer or not.

Replace current config directory from Homeassistant with the one you downloaded.

#### 2.2 Setup the thermostats

Once configuration has been set up, add thermostats from homeassistant. It should be able to identify them as __EUROtronic EUR_SPIRITZ Wall Radiator Thermostat__ instead of __Unknown thermostat__ and they should work much more reliably.
