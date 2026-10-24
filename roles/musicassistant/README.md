# musicassistant

The role downloads musicassistant from git to `/usr/src/musicassistant` and installs it from there into a venv at `/opt/musicassistant`.

As musicassistant requires python 3.14 but debian trixie delivers 3.13 only, it expects `/opt/python314/bin/python3.14` to exist (see the [python314 role](../python314/README.md)).

The Web-GUI is reachable via `http://<target hostname or ip address>:8095/`

The files are owned by the created user `musicassistant` so that plugins can be downloaded and installed via the Web GUI.

Idempotency is implemented by checking whether the git download did any changes.

## Variables

none yet.
