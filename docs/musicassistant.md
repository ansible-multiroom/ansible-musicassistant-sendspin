# Configuring Music Assistant

This document describes the settings I applied to Music Assistant after installation.

## Admin: Settings / User management

The very first user you create immediately after installation is an administrator account. If you are not the only user in your household, I strongly use this first account for configuring the system only, but not for every day use.

For every user in your household, create an account. As long as you don't expose your instance to the internet, you can use simple users and passwords, but do create accounts.

### User: Settings / User Interface

These settings have to be made when logged in as the user the setting should affect.

* Decide about the **language** you would like to use. The default is to inherit the language from the end user device, but you might want to override this.

* The **volume controls** are much easier to handle on a mobile if you are using the  *relative (drag to adjust)* setting, especially near the lowest volume.

* Decide if you want to use an audio player on your controlling device or not. If you **Enable built-in (Sendspin) Web Player**, you can listen to your music also on your mobile or laptop (the controlling device), but the user interface will show an additional audio sink and their naming is not always obvious.

## User: other settings

You can configure the left side **menu** and the content of the **Discover** view by using the respective *edit* options.

I removed the **Party**, **Genres**, **Podcasts** and **Audiobooks** from the menu.

I only kept **recently played** and **recently added tracks** on the Discover page.

## Admin: Settings / Music sources / Filesystem (local disk)

Configure here the path where you mounted the local audio collection. This is usually [/media/music](/roles/musicassistant/templates/etc/systemd/system/musicassistant.service.j2#L21) where it is mapped to in the container from the os mount path [/srv/music](/roles/smb_mount/defaults/main.yml).

If your collection misses the *albumartist* tag in many places, check the options for the **Action when a track is missing the Albumartist ID3 tag**. I have set it to *Use Track artist(s)*, but YMMV.

## Admin: Settings / Music sources / Music Assistant

Here you can configure the auto generated playlists you actually want to use. I kept only **All favorited tracks**, **Recently played tracks** and **Recently added tracks**.

## Admin: Settings / Music sources / Radio Paradise

I am a big fan of the [Radio Paradise](https://radioparadise.com/) streams (no talk, no ads, just music), so I activated that music source. If you want one or more of those streams appear in the *Radio* menu, just star the ones you like.

## Admin: Settings / Music sources / Spotify

Although Musicassistant delivers a builtin application id for spotify that works out of the box, I strongly suggest to create and use your own spotify application id: Although it misses some functions (i.e. you cannot play songs from playlists that are not yours), you don't suffer from the massive rate limiting the globally shared application id does.

I also strongly suggest to use the Soloist integration (where you will need an extra Soloist API key in addition).

One of the final steps of the spotify music source configuration is to link it your spotify app (and account). Therefore, you should map one spotify integration to one music assistant account, i.e. if you have multiple users, you should create multiple spotify integrations, name them accordingly and give each music assistant user access to its own spotify integration only. This mapping probably applies to any streaming provider.

However, I finally removed the Spotify Music Source as I want to primarily rely on my local files and use spotify only sometimes as add on. The spotify integration is much too integrated, i.e. I often had hits from spotify before the ones of my local collection, in some use cases you don't even see if you are accessing a spotify or local resource etc.

## Admin: Settings / Plugins / Sendspin Source

This must be enabled so that you can make use of the Sendspin sources you might have configured in the [plumbing role](/roles/plumbing).

## Admin: Settings / Plugins / Spotify connect

After ditching Spotify as Music Source, I tried the Spotify connect plugin. Spotify connect makes Music Assistant appear as Audio Sink on the Spotify App. But only for the audio sinks you configure in **Connected Players**.

As above, I strongly suggest to use the spotify *Soloist* option.

## Admin Settings / Players

Here you configure multiroom sync groups to enable actual multiroom. You should not create multiple groups using the same sinks, I created just one called *everywhere* containing all sinks.

It's also the place to accept all sendspin **sources** so that you can actually receive the audio stream from them. As the ansible [plumbing role](/roles/plumbing) only builds either sendspin sinks or sources, you should **Hide this player in the user interface** for each sendspin *source* in order to minimize clutter in the Music Assistant user interface. I also chose to set specific icons, but they never showed up again...

## Admin Settings / System / Player Queues

As I like to accumulate longer queues, I changed the **Default enqueue option** to *Play (and keep queue)* for all items.

## Admin Settings / System / Players

I played with the **Volume step size**, but learned that the default of *0* is exactly what I want: It uses smaller steps near the bottom and top of the volume range - provided you *use the scroll wheel of your mouse*.

## Admin Settings / System / Background tasks (violet)

I disabled the *Audio analysis* schedule as I feared the cpu and memory footprint: My music assistant is running on a 2 GB RAM Pi 4.

## Menu / Radio

Provided you use an *admin* or *user* account, you can add Radio stations to the **Radio** menu. Note that you need the stream URL itself and not e.g. a m3u link that contains the stream URL.
