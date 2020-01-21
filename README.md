# SSHD

Minimal Alpine Linux Docker image with `sshd` exposed and `rsync` installed.

## Environment Options

Configure the container with the following environment variables or optionally mount a custom sshd config at `/etc/ssh/sshd_config`:

- `SSH_USERS` list of user accounts and uids/gids to create. eg `SSH_USERS=www:48:48,admin:1000:1000`
- `SSH_ENABLE_ROOT` if "true" unlock the root account
- `SSH_ENABLE_PASSWORD_AUTH` if "true" enable password authentication (disabled by default)
- `MOTD` change the login message
- `SFTP_MODE` if "true" sshd will only accept sftp connections
- `SFTP_CHROOT` if in sftp only mode sftp will be chrooted to this directory. Default "/data"
- `GATEWAY_PORTS` if "true" sshd will allow gateway ports
- `TCP_FORWARDING` if "true" sshd will allow TCP forwarding
- `IDLE_EXIT` if "true" script will check ssh connections, kill sshd when connection idle
- `IDLE_CHECK_INTERVAL` check alive script interval time. Default 5 minutes
- `IDLE_CHECK_MAX_COUNT` number of idle max count

## SSH Host Keys

SSH uses host keys to identify the server. To avoid receiving security warning the host keys should be mounted on an external volume.

By default this image will create new host keys in `/etc/ssh/keys` which should be mounted on an external volume. If you are using existing keys and they are mounted in `/etc/ssh` this image will use the default host key location making this image compatible with existing setups.

If you wish to configure SSH entirely with environment variables it is suggested that you externally mount `/etc/ssh/keys` instead of `/etc/ssh`.

## Authorized Keys

Mount your .ssh credentials (RSA public keys) at `/root/.ssh/` in order to
access the container via root and set `SSH_ENABLE_ROOT=true` or mount each user's key in
`/etc/authorized_keys/<username>` and set `SSH_USERS` environment config to create the user accounts.

Authorized keys must be either owned by root (uid/gid 0), or owned by the uid/gid that corresponds to the
uid/gid and user specified in `SSH_USERS`.

## SFTP mode

When in sftp only mode (activated by setting `SFTP_MODE=true` the container will only accept sftp connections. All sftp actions will be chrooted to the `SFTP_CHROOT` directory which defaults to "/data".

Please note that all components of the pathname in the ChrootDirectory directive must be root-owned directories that are not writable by any other user or group (see `man 5 sshd_config`).

## Custom Scripts

Executable shell scripts and binaries can be mounted or copied in to `/etc/entrypoint.d`. These will be run when the container is launched but before sshd is started. These can be used to customise the behaviour of the container.

## Usage Example

The example below will run interactively and bind to port `2222`. `/data` will be
bind mounted to the host. And the ssh host keys will be persisted in a `keys`
directory.

You can access with `ssh root@localhost -p 2222` using your private key.

```
docker run -ti -p 2222:22 \
  -v ${HOME}/.ssh/id_rsa.pub:/root/.ssh/authorized_keys:ro \
  -v $(pwd)/keys/:/etc/ssh/keys \
  -v $(pwd)/data/:/data/ \
  -e SSH_ENABLE_ROOT=true \
  docker.io/panubo/sshd:1.1.0
```

Create a `www` user with gid/uid 48. You can access with `ssh www@localhost -p 2222` using your private key.

```
docker run -ti -p 2222:22 \
  -v ${HOME}/.ssh/id_rsa.pub:/etc/authorized_keys/www:ro \
  -v $(pwd)/keys/:/etc/ssh/keys \
  -v $(pwd)/data/:/data/ \
  -e SSH_USERS="www:48:48" \
  docker.io/panubo/sshd:1.1.0
```

## Releases

For production usage, please use a versioned release rather than the floating 'latest' tag.

See the [releases](https://github.com/panubo/docker-sshd/releases) for tag usage
and release notes.

## Status

Production ready and stable.
