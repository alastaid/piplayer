# piplayer
My configuration for Raspberry Pi audio player

Looking on the web I know many have been playing around with different configurations, this is mine. I have a number of Pi's (mixture of Model B Plus Rev 1.2 & Pi 3's) in the house all with either a HiFiBerry DacPlus or HiFiBerry Amp.

Requirements are:

1 - Rock solid Spotify Connect to each player

2 - Ability to Spotify Connect to a collection of players (ie Downstairs)

3 - Play internet radio to any player

4 - On some of the players, pass-through the input from a USB soundcard on the pi to the HiFiBerry

The last one is so where I am using a soundbar for the amp/speakers I can have a "single" source, so dont have to keep faffing with finding the remote and changing the source of the soundbar from Pi to TV and vice-versa.  After many many hours and attempts, this appears to be the best I can get it, previously I used LMS and picoreplayer which was perfect except the spotty plugin just wasnt reliable enough, and I couldnt get raspotify working on picoreplayer.

I ended up with pulseaudio on the clients as the default sink to the hifiberry, raspotify for spotify connect, snapcast for collection of spotify connect players, squeezelite for radio from LMS with presets configured on tunein, and lastly configuring pulseaudio for pass-through on the players with soundbars.  If I didn't use pulseaudio, something would lock up or hog the card and it wasnt reliable enough.

As part of the setup, make sure that the avahi-daemon config is right on the network/players, check the /var/log/syslogs for machines looping with avahi, and to test, make sure that every machine that is to be used with raspotify can ssh pi@machine.local to each other, this proves the avahi is working.  I have only included the changes to the config files here, instructions on how to find and install the s/w are on the net.

##############################################################
## Disable the on-board sound card and enable the HiFiBerry
##############################################################

/boot/config.txt

#Enable audio (loads snd_bcm2835)

#dtparam=audio=on


dtoverlay=hifiberry-dacplus

#################################################################
## Make pulseaudio the default to alsa
#################################################################

/etc/asound.conf

pcm.!default {

  type pulse
  
}

ctl.!default {

  type pulse
  
}

################################################################
## Pulseaudio config
################################################################

Determine the devices you want as source and sink by starting pulseaudio, then running pacm, and use list-sources and list-sinks.  For info I use pulseaudio in "system" mode, as the pi's get powered on, play music, turned off, only logged into when something has gone wrong.

/etc/pulse/client.conf

default-sink = alsa_output.platform-soc_sound.stereo-fallback

default-source = alsa_input.usb-0d8c_USB_PnP_Sound_Device-00.analog-mono

autospawn = no

daemon-binary = /usr/bin/pulseaudio

/etc/pulse/system.pa

.ifexists module-esound-protocol-unix.so

load-module module-esound-protocol-unix auth-anonymous=1     (just add the auth-anonymous=1)

.endif

load-module module-native-protocol-unix auth-anonymous=1     (just add the auth-anonymous=1)

load-module module-loopback source=alsa_input.usb-0d8c_USB_PnP_Sound_Device-00.analog-mono sink=alsa_output.platform-soc_sound.stereo-fallback    (if using the passthrough from source to sink, otherwise ignore)

/etc/systemd/system/pulseaudio.service

[Unit]

Description=PulseAudio system server

After=avahi-daemon.service network.target


[Service]

Type=simple

ExecStart=/usr/bin/pulseaudio --system --realtime --disallow-exit --no-cpu-limit

[Install]

WantedBy=multi-user.target

#############################################################
## Snapclient config
#############################################################

/etc/systemd/system/multi-user.target.wants/snapclient.service

After=network-online.target time-sync.target sound.target avahi-daemon.service pulseaudio.service (just add pulseaudio at the end)

/etc/default/snapclient

SNAPCLIENT_OPTS="-h 192.168.x.x"

#############################################################
## Snapserver Config
#############################################################

/etc/snapserver.conf

stream = spotify:///librespot?name=Spotify&devicename=Downstairs&volume=50

As an aside, I really wanted some other options to librespot added with snapserver, but the options are not available in snapserver by default. I followed the instructions on the snapcast repo for building snapserver and if you follow them carefully its pretty straight forward to build, I edited the options I wanted in the server/streamreader/lirespot_stream.cpp file so when I ps -ef | grep libre on the snapserver machine I get the following:

/usr/bin/librespot --name Downstairs --bitrate 320 --backend pipe --initial-volume 50 --linear-volume --enable-volume-normalisation --autoplay --verbose


##############################################################
##  Raspotify Config
##############################################################

/etc/default/raspotify

OPTIONS="--device pulse"

#BACKEND_ARGS="--backend alsa"

/etc/systemd/system/multi-user.target.wants/raspotify.service

[Unit]

Description=Raspotify

After=network.target pulseaudio.service


[Service]

User=raspotify

Group=raspotify

Restart=always

RestartSec=10

PermissionsStartOnly=true

ExecStartPre=/bin/mkdir -m 0755 -p /var/cache/raspotify ; /bin/chown raspotify:raspotify /var/cache/raspotify

Environment="DEVICE_NAME=raspotify (%H)"

Environment="BITRATE=160"

Environment="CACHE_ARGS=--disable-audio-cache"

Environment="VOLUME_ARGS=--enable-volume-normalisation --linear-volume --initial-volume=30"

Environment="BACKEND_ARGS=--backend alsa"

EnvironmentFile=-/etc/default/raspotify

ExecStart=/usr/bin/librespot --name ${DEVICE_NAME} $BACKEND_ARGS --bitrate ${BITRATE} $CACHE_ARGS $VOLUME_ARGS $OPTIONS


[Install]

WantedBy=multi-user.target


################################################################
## Squeezelite config
################################################################

/etc/systemd/system/squeezelite.service

[Unit]

Description=Squeezelite service

After=network.target snapclient.service


[Service]

ExecStart=/usr/bin/squeezelite -o pulse -n playername -s 192.168.x.x


[Install]

WantedBy=multi-user.target

###################################################################
##  Running on Raspberry Pi Model B Plus Rev 1.2
###################################################################

Due to the slower processor on the Model B, I had to make a few changes to stop the squeezelite stuttering constantly.  

I overclocked the processor to the first level 800Mhz, not actually sure it still needs it.

/etc/default/raspotify

BITRATE="160"   It cant cope with 320

/etc/systemd/system/multi-user.target.wants/squeezelite.service

ExecStart=/usr/bin/squeezelite -o pulse -n "ARoom" -s 192.168.x.x -a 44100



