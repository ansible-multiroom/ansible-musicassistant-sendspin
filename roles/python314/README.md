# python314

This role downloads the source of python 3.14 from `https://www.python.org`, compiles it and installs it to `/opt/python314`.

It takes a bit less than 30 minutes on a raspi 4. 

Idempotency is implemented by explicitly checking the existence of `/opt/python314/bin/python3.14.

This is prerequisite for musicassistant.

## Variables

none, the version is currently hardcoded to `3.14.8`

