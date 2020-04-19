# piplayer
My configuration for Raspberry Pi audio player

Looking on the web I know many have been playing around with diiferent configurations, this is mine. I have a number of Pi's in the house all with either a HiFiBerry DacPlus or HiFiBerry Amp.

Requirements are:

1 - Rock solid Spotify Connect to each player
2 - Ability to Spotify Connect to a collection of players
3 - Play internet radio to any player
4 - On some of the players, pass-through the input from a USB soundcard to the HiFiBerry

The last one is so where I am using a soundbar for the amp/speakers I can have a "single" source, so dont have to keep faffing with finding the remote and changing the source of the soundbar from Pi to TV and vice-versa

##############################################################
## Disable the on-board sound card and enable the HiFiBerry
##############################################################

sudo nano /boot/config.txt

# Enable audio (loads snd_bcm2835)
#dtparam=audio=on
dtoverlay=hifiberry-dacplus
