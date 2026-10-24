# maclient

This role installs a virtenv at `/opt/maclient` using python 3.14 from `/opt/python314/` with the `music_assistant_client` module.

As example script, a tool to import m3u playlists from mpd into musicassistant is provided (`import_m3u_playlist.py`). The m3u playlist contains filenames, the import script expects to find the files in the m3u at the same place (including subdirectory structure) as MusicAssistant sees them. Of course, the beginning of the path will be different in both systems, but this is taken into account. You might need to adjust the `MPD_PATH_PREFIX_TO_REMOVE`. Both found and not found files are listed during the import.

Note that `import_m3u_playlist.py` expects a file called `ma_access_token` to be in the same directory as the script itself containing a "long lived token". Use the Music Assistant Web GUI to create such a token.
