# piplayer
My configuration for Raspberry Pi audio player

Looking on the web I know many have been playing around with diiferent configurations, this is mine. I have a number of Pi's in the house all with either a HiFiBerry DacPlus or HiFiBerry Amp.

Requirements are:

1 - Rock solid Spotify Connect to each player

2 - Ability to Spotify Connect to a collection of players (ie Downstairs)

3 - Play internet radio to any player

4 - On some of the players, pass-through the input from a USB soundcard on the pi to the HiFiBerry

The last one is so where I am using a soundbar for the amp/speakers I can have a "single" source, so dont have to keep faffing with finding the remote and changing the source of the soundbar from Pi to TV and vice-versa.  So after many many hours and attempts, this appears to be the best I can get it, LMS and picoreplayer was perfect except the spotty plugin just wasnt reliable enough, and I couldnt get raspotify working on picoreplayer.

So ended up with pulseaudio on the clients as the default sink to the hifiberry, raspotify for spotify connect, snapcast for collection of spotify connect, squeezelite for radio from LMS with presets configured on tunein, and lastly configuring pulseaudio for pass-through on the players with soundbars.  If I didn't use pulseaudio, something would lock up or hog the card and it wasnt reliable enough.

As part of the setup make sure that the avahi-daemon config is right on the players, check the /var/log/syslogs for machines looping with avahi, and to test, make sure that every machine that is to be used with raspotify can ssh pi@machine.local to each other, this proves the avahi is working correctly.

##############################################################
## Disable the on-board sound card and enable the HiFiBerry
##############################################################

/boot/config.txt

#Enable audio (loads snd_bcm2835)
#dtparam=audio=on
dtoverlay=hifiberry-dacplus

#################################################################
## Make pulseaudio the default sink
#################################################################

/etc/asound.conf

pcm.!default {
  type pulse
}

ctl.!default {
  type pulse
}
