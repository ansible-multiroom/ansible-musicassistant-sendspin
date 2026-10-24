# plumbing

The role cares about the configuration of physical audio sinks (for playback) and sources (for capture). 

Without installing this role, musicassistant will only be able to play to the built in Web-Player.

## Basics

**Playing audio** is outputting audio data to an **audio sink** like e.g. a loudspeaker.

**Capturing audio** (or *recording audio*) is inputting audio data from an **audio source** like e.g. a microphone or a playlist player etc.

For all audio, we only use **alsa** devices and all audio devices use the same audio format (currently `S16_LE/2/44100`). For *playing*, we always use the `dmix` alsa endpoint, for *capturing*, the `dsnoop` alsa endpoint. In contrast to the `hw` endpoint, these allow multiple simultaneous processes to play/capture audio - as long all processes use the same audio format.

## Components

The following base components are available

* alsa to alsa (aka **a2a**): *capture* audio from an alsa source and *play* it to an alsa sink.
* alsa to http (aka **a2h**): *capture* audio from an alsa source to a http server that can be consumed like an Internet Radio.
* sendspin sink (aka **s2a**): get audio from a sendspin server (musicassistant) and *play* it to an alsa sink. Sendspin sinks can be synchronized by the server (musicassistant) to achieve seamless multiroom experience.
* sendspin source (aka **a2s**): *capture* audio from an alsa source and send it to a sendspin server (musicassistant).

The **a2a** role also supports the configuration of a `bluealsa` sound card (for **capture only**) so that you can stream audio from e.g. your mobile via bluetooth to musicassistant.

## Variables

The components are configured via the `plumbings` variable which is a list of connection dicts. This variable must be set explicitly for each target host in order to **play** or **capture** audio. From an ansible point of view, you can combine the base components in any combination, but there are some rules you should follow, leaving only a few combinations that really make sense. 

### Rules

* the `bluealsa` soundcard is ephemeral and is really usable only as long as there is an actual bluetooth connection. On the other hand, both **a2h** and **a2s** are only working properly (i.e. start their http/sendspin service) when there is actually audio data available. Therefore **a2h** and **a2s** must not directly **capture** from `bluealsa`
* **s2a** is used as part of the multiroom experience and should play to an alsa sink that is directly connected to physical audio sink (e.g. loudspeaker). Otherwise you would introduce varying delays that would ruin the synchronous multiroom experience.
* While *sendspin* usually relies on autodetection via mDNS, the implementation here fully relies on the sendspin clients to actively contact audioassistant.
* Each plumbing has a `name` key. The value should consist of lower case letters, numbers, and underscores only. It is used for file and directory names etc. and must be unique for that target host.

### Example 1: Sendspin Loudspeaker

~~~
plumbings:
  - name: loudspeaker_from_musitest
    s2a:
      sendspin_own_port: 8930
      sendspin_ma_url: "ws://<your sendspin server name or ip address>:8927/sendspin"
      sendspin_ma_name: "{{ ansible_facts['hostname'] }}"
      sink: "dmix:CARD=sndrpihifiberry,DEV=0"
~~~

In the `s2a` dict, the keys have the following meanings:

* `sendspin_own_port`: the port that the sendspin process is reachable. Must be specified, unique for that target host. But is not used in the current setup.
* `sendspin_ma_url`: The URL to reach the sendspin server of musicassistant. This is usually on port `8972` and the endpoint `sendspin` as shown in the example above
* `sendspin_ma_name`: This is the name of this sendspin client that will be shown in the musicassistant GUI (e.g. to control the volume or to group together with others). Should be unique within the musicassistant server.
* `sink`: The alsa audio sink to *play* the audio to. Should be a real sound card endpoint with the loudspeakers directly connected to it (at least as long as you want the multiroom experience).

### Example 2: Line In as musicassistant sendspin audio source

~~~
plumbings:
  - name: linein_to_musitest
    a2s:
      source: "dsnoop:CARD=sndrpihifiberry,DEV=0"
      sendspin_own_port: 8940
      sendspin_ma_url: "ws://<your sendspin server name or ip address>:8927/sendspin"
      sendspin_ma_name: "linein_{{ ansible_facts['hostname'] }}"
~~~

In the `a2s` dict, the keys have the following meanings:

* `source`: The alsa audio source to *capture* the audio from. Should be a permanent alsa soundcard where you always can capture audio from.
* `sendspin_own_port`/`sendspin_ma_url`: See in first example.
* `sendspin_ma_name`: This is the name of this sendspin client that will be shown in the musicassistant GUI. As there can be multiple sources from the same host, you should qualify the name with the type of the source (like `linein` in the example).
* `sink`: The alsa audio sink to *play* the audio to. Should be a real sound card endpoint with the loudspeakers directly connected to it (at least as long as you want the multiroom experience).

### Example 3: Bluetooth as musicassistant sendspin audio source

~~~
plumbings:
  - name: bluetooth_to_musitest
    a2a:
      source: bluealsa
      sink: dmix:CARD=Loopback,DEV=0,SUBDEV=0
    a2s:
      source: dsnoop:CARD=Loopback,DEV=1,SUBDEV=0
      sendspin_own_port: 8941
      sendspin_ma_url: "ws://<your sendspin server name or ip address>:8927/sendspin"
      sendspin_ma_name: "bluetooth_{{ ansible_facts['hostname'] }}"
~~~

Here we have **a2a** and **a2s** used for one plumbing, connected via a Loopback alsa audio card. Routing via **a2a** first is required for bluetooth connections (see above).  The Loopback card works as follows:

* There are 8 subdevices (SUBDEV=0 to SUBDEV=7).
* Whatever you *play* to `Looback,DEV=0` comes out of `Loopback,DEV=1` on the same subdevice and vice versa. So you have up to 16 virtual alsa connections.

The semantics of the **a2a** keys should be obvious:

* `source`: An alsa source. If it is called `bluealsa`, the ansible role configures `bluetoothd` with an agent and also `bluealsa`.
* `sink`: An alsa sink.

### Example 4: Line In to http (can be used as musicassistant source)

~~~
plumbings:
  - name: line2http8950
    a2h:
      source: dsnoop:CARD=sndrpihifiberry,DEV=0
      http_port: 8950
~~~

The **a2h** keys have the following meaning:
* `source`: An alsa source. You should **not** use bluealsa here (although it is possible from a syntax point of view)
* `http_port`: The port you will be able to get the audio stream from. You could use this to add a "radio station" to musicassistant, but this will result in much more latency than the Example 2 above with a sendspin source.

### Example 5: Local loops (bypassing musicassistant)

Sometimes, you want to use a certain device as direct bluetooth or linein loudspeaker without feeding the audio to musicassistant

~~~
plumbings:
  - name: local_bluetooth
    a2a:
      source: bluealsa
      sink: "dmix:CARD=sndrpihifiberry,DEV=0"
  - name: local_loop
    a2a:
      source: dsnoop:CARD=sndrpihifiberry,DEV=0
      sink: "dmix:CARD=sndrpihifiberry,DEV=0"
~~~

For a local loopback, you don't need to use the loopback device, as the consuming device does not care if "nothing" is delivered - it will output just silence.

The upper is an example for "from bluetooth to local dmix", the lower for "from local dsnoop to to local dmix".

## Other variables

* `test_cards`: This is used to parametrize the `/usr/local/bin/testalsa` script that allows to quickly check what audio format is currently used on the alsa endpoints
* `sendspin_version`: Currently defaulting to 0.4.0, the first version of sendspin-cpp-cli that supports sendspin acting as audio source.
* `sendspin_config_dir`, `sendspin_state_dir`, `sendspin_socket_dir`: Used in the sendspin config files and required because we run multiple sendspin processes on the same host and therefore cannot use the default values.

See also the [defaults](defaults/main.yml).

## Implementation hints

* The role is fully idempotent
* On each host with `plumbings != []", the following checks are applied to the elements of `plumbings`:
   * The `sendspin-cli` binary is installed when there is at least one `a2s` or `s2a` key in one of the connections
   * The `snd_aloop` module which creates the `Loopback` audio card is installed when there is at least one connection with more than 2 keys. The first is the `name` and the others must be components. So with 3 keys or more you will need the `Loopback` audio card
   * `bluealsa` is configured when there is at least one `a2a.source == 'bluealsa'`.
