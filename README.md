# Motivation

I am running my self built [multiroom solution](https://github.com/ansible-multiroom/ansible-multiroom-audio) for
multiple years now.

While I am quite happy with it, times are changing and there are new players in town. This repo shall implement
the same feature set but with modern tools

* [musicassistant](https://www.music-assistant.io/) as core component to select the music that is played
* [sendspin](https://www.sendspin-audio.com/) for synchronous multiroom audio

## Old Implementation and Feature Set  

As **hardware**, I use **raspberrry pis** and **[hifiberry](https://www.hifiberry.com/) audio HATs** that are connected
either to classic HiFi-equipment with an amp and decent loudspeakers or directly to a pair of loudspeakers when the HAT 
has a built in amp.

As **software**, I used [mpd](https://www.musicpd.org/) to select the music to play (I have collection of about 100K MP3 and
FLAC files) and [snapcast](https://github.com/snapcast/snapcast) to distribute the music synchronously to my rooms. This
was glued together with some [ALSA](https://en.wikipedia.org/wiki/Advanced_Linux_Sound_Architecture) tricks.

In addition to play music from files, I also supported playing audio captured from the soundcard (e.g. when playing
a vinyl record) or delivered via bluetooth e.g. from a mobile using [bluez-alsa](https://github.com/arkq/bluez-alsa).

As most native mpd clients had one or the other bug or were not in active development any more, I added web frontends for
mpd ([ampd](https://github.com/rain0r/ampd), [rompr](https://fatg3erman.github.io/RompR/)) with mixed success. Especially rompr
was not able to deal with a collection of more than 100K audio files and playlists with 1000 songs or more.

## New Implementation (wip)

Too keep the memory footprint as small as possible (most of my Raspberry Pis have only 2GB of RAM), I did not use the 
pre made container for audio assistant but installed it from source. 

### Ansible Roles

Base Roles:

* `asound_conf`
* `python314` (required for musicassistant)
* `smb_mount`

Feature roles

* sendspin
* musicassistant

Planned, but roles not defined yet:

* support for audio capture (probably a special sendspin configuration
* support for bluetooth sources (probably a combination of bluez-alsa and sendspin)
