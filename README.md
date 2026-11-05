# Motivation

I have been running my self built [multiroom solution](https://github.com/ansible-multiroom/ansible-multiroom-audio) for multiple years now. You can find a short description of my old implementation at the bottom.

While I was quite happy with it, times are changing and there are new players in town. This repo implements the same feature set but with modern tools:

* [musicassistant](https://www.music-assistant.io/) as core component to select the music that is played
* [sendspin](https://www.sendspin-audio.com/) for synchronous multiroom audio

## Hardware

I use **raspberry pis** with **[hifiberry](https://www.hifiberry.com/) audio HATs** that are connected
either to classic HiFi-equipment with an amp and decent loudspeakers or directly to a pair of loudspeakers when the HAT has a built in amp. Any other audio HATs will work too, but you will have use the proper ALSA device names in the configuration. Don't use the builtin audio output if you love your ears.

For the Music Assistant host, the bare minimum is a Pi 4 with 2G of RAM (more is better), for audio satellites (pure audio sources and sinks), even a Pi Zero 2 is sufficient.

## Software

### Ansible Roles

For the central Music Assistant node:
* [`smb_mount`](roles/smb_mount/README.md): mount remote audio library read only, useful for musicassistant
* [`musicassistant`](roles/musicassistant/README.md): Downloads and install the current version of music assistant server from github
* [`maclient`](roles/maclient/README.md): Installs a venv with the music assistant client library and a demo tool to migrate mpd playlists to musicassistant

For plumbing audio sinks and sources on all nodes to Music Assistant:
* [`asound_conf`](roles/asound_conf/README.md): Configures a default data rate for `dsnoop` and `dmix` devices
* [`plumbing`](roles/plumbing/README.md): Builds up all audio sources and sinks in your house to build the multiroom system

Network Helper for selected hosts:
* [`vwbridge`](roles/vwbridge/README.md): Configures a wired Pi as WLAN extender.

For more details, please check the linked README files in the respective role directories

### Ansible Tags and Host Groups

This describes the settings and configuration used for the [testamsel](environments/testamsel) environment. You should be able to use the [playbook](environments/testamsel/playbook.yml) and the [group_vars](environments/testamsel/group_vars/all.yml) unchanged and adjust the [inventory](environments/testamsel/inventory) only.

Tags:
* `musicassistant`: Roles needed to run the music assistant server
* `maclient`: Role to install the music assistant client library
* `plumbing`: Roles needed to make plumbing work
* `vwbridge`: Role to make a wired Pi a WLAN extender

Host groups:
* `musicassistants`: Host(s) where the music assistant server and a music assistant client are to be installed
* `plumbed`: Host(s) where any physical audio source or sink is required
* `vwbridge`: Wired host(s) to be used as WLAN extender

## Install it

### Preconditions Ansible host

Install the dependencies on your ansible host:

~~~
ansible-galaxy collection install -r ./requirements.yml
~~~


### Preconditions Raspberry Pis

All your target pis are running at least debian trixie and are reachable in the network, preferrably via a hostname (and not only via an IP address).

You can ssh into them without password (either as user `pi` and `pi` can become root without password, or directly as `root`).

You have verified that the audio hats work and the internal sound card is disabled:

* use `aplay -L` to show the available audio cards. You should see e.g. `hw:CARD=sndrpihifiberry,DEV=0` for hifiberry cards
* adjust `/boot/firmware/config.txt`:
   * for older hifiberry cards, you might need
      * `dtoverlay=hifiberry-dac` for the miniamp hat
      * `dtoverlay=hifiberry-dacplusadc` for the dac+adc hat
   * `#dtparam=audio=on` to disable the internal audio card
   * `dtoverlay=disable-wifi` if you don't need wifi

### Create a new environment (only the first time)

Create an environment directory:

~~~
cd environments
mkdir myown
cd myown
~~~

Populate it e.g. from the `testamsel` environment

~~~
cp -a ../testamsel/inventory ../testamsel/playbook.yml .
cp -a ../testamsel/group_vars .
mkdir host_vars
~~~

### Configure it

Adjust the `inventory`, put (at least) one host into the `[musicassistants]` group, put at least one host into the [`plumbed`] group. Usually, your server host is in both groups.

Configure the `plumbing` variable for each host according to your needs in the `host_vars` directory.

### Test it

Run the playbook first in check mode to verify that your inventory and vars make sense:

~~~
cd environments/testamsel
ANSIBLE_CONFIG=../../ansible.cfg ansible-playbook -CD -i inventory playbook.yml -u pi
~~~

You can use `--tags` to select only some roles and/or `--limit` to select only some target hosts.

### Do it

Finally, remove the "C" from the `ansible-playbook` command to really apply the changes.

## Old Implementation

I used [mpd](https://www.musicpd.org/) to select the music to play (I have collection of about 100K MP3 and FLAC files) and [snapcast](https://github.com/snapcast/snapcast) to distribute the music synchronously to my rooms. This was glued together with some [ALSA](https://en.wikipedia.org/wiki/Advanced_Linux_Sound_Architecture) tricks.

In addition to play music from files, I also supported playing audio captured from the soundcard (e.g. when playing a vinyl record) or delivered via bluetooth e.g. from a mobile using [bluez-alsa](https://github.com/arkq/bluez-alsa).

As most native mpd clients had one or the other bug or were not in active development any more, I added web frontends for mpd ([ampd](https://github.com/rain0r/ampd), [rompr](https://fatg3erman.github.io/RompR/)) with mixed success. Especially rompr was not able to deal with a collection of more than 100K audio files and playlists with 1000 songs or more.
