# musicassistant

The role uses the official Music Assistant container and runs it from a systemd service using rootless `podman` as the local user `musicassistant`.

The Web-GUI is reachable via `http://<target hostname or ip address>:8095/`

Idempotency is implemented by checking whether the git download did any changes.

## Variables

Usually, you don't need to set these variables and can just use the [defaults](defaults/main.yml)

* `musicassistant_user`: The name of the user and group to run podman
* `musicassistant_data_dir`: The directory where the persistent data for the container is stored
* `musicassistant_subid_count`, `musicassistant_subuid_start`, `musicassistant_subgid_start`: The values required for the podman id mapping
* `musicassistant_image`: The source of the Music Container image 
* `musicassistant_container_name`: The name of the running container
