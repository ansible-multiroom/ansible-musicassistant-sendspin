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

## New Implementation

Too keep the memory footprint as small as possible (most of my Raspberry Pis have only 2GB of RAM), I don't use the
pre made container for audio assistant but install it from source.

### Ansible Roles

* [`asound_conf`](roles/asound_conf/README.md): Configures a default data rate for `dsnoop` and `dmix` devices
* [`python314`](roles/python314/README.md): required for musicassistant
* [`smb_mount`](roles/smb_mount/README.md): mount remote audio library read only, useful for musicassistant
* [`musicassistant`](roles/musicassistant/README.md): Downloads and install the current version of music assistant server from github
* [`plumbing`](roles/plumbing/README.md): Builds up all audio sources and sinks in your house to build the multiroom system
* [`maclient`](roles/maclient/README.md): Installs a venv with the music assistant client library and a demo tool to migrate mpd playlists to musicassistant

For more details, please check the README files in the respective role directories

### Ansible Tags and Host Groups

This describes the settings and configuration used for the [testamsel](environments/testamsel) environment. You should be able to use the [playbook](environments/testamsel/playbook.yml) and the [group_vars](environments/testamsel/group_vars/all.yml) unchanged and adjust the [inventory](environments/testamsel/inventory) only.

Tags:
* `musicassistant`: Roles needed to run the music assistant server
* `plumbing`: Roles needed to make plumbing work
* `maclient`: Roles to install the music assistant client library

Host groups:
* `musicassistants`: Host(s) where the music assistant server (and, if wanted, music assistant client) should be installed
* `plumbed`: Hosts(s) where any physical audio source or sink is required

## Install it

### Preconditions

All your target pis are running at least debian trixie and are reachable in the network, preferrably via a hostname (and not only via an IP address).

You can ssh into them without password (either as user `pi` and `pi` can become root without password, or directly as `root`).

### Configure it

Populate your `inventory`, put (at least) one host into the `[musicassistants]` group, put at least one host into the [`plumbed`] group. Usually, your server host is in both groups.

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


