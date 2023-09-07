# The Tailscale universal Docker mod

This Docker mod lets you slipstream Tailscale into
[linuxserver.io](https://linuxserver.io) containers. This lets you
have applications join your tailnet.

## Configuration

The Docker mod exposes a bunch of environment variables that you can
use to configure it.

| Environment Variable      | Description                                                                                                                                                                                                                                                                                                   | Example                                  |
| :------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :--------------------------------------- |
| `DOCKER_MODS`             | The list of additional mods to layer on top of the running container, separated by pipes.                                                                                                                                                                                                                     | `ghcr.io/tailscale-dev/docker-mod:main`  |
| `TAILSCALE_STATE_DIR`     | The directory where the Tailscale state will be stored, this should be pointed to a Docker volume. If it is not, then the node will set itself as ephemeral, making the node disappear from your tailnet when the container exits.                                                                            | `/var/lib/tailscale`                     |
| `TAILSCALE_AUTHKEY`       | The authkey for your tailnet. You can create one in the [admin panel](https://login.tailscale.com/admin/settings/keys). See [here](https://tailscale.com/kb/1085/auth-keys/) for more information about authkeys and what you can do with them.                                                               | `tskey-auth-hunter2CNTRL-hunter2hunter2` |
| `TAILSCALE_HOSTNAME`      | The hostname that you want to set for the container. If you don't set this, the hostname of the node on your tailnet will be a bunch of random hexadecimal numbers, which many humans find hard to remember.                                                                                                  | `wiki`                                   |
| `TAILSCALE_USE_SSH`       | Set this to `1` to enable SSH access to the container.                                                                                                                                                                                                                                                        | `1`                                      |
| `TAILSCALE_SERVE_PORT`    | The port number that you want to expose on your tailnet. This will be the port of your DokuWiki, Transmission, or other container.                                                                                                                                                                            | `80`                                     |
| `TAILSCALE_SERVE_MODE`    | The mode you want to run Tailscale serving in. This should be `https` in most cases, but there may be times when you need to enable `tls-terminated-tcp` to deal with some weird edge cases like HTTP long-poll connections. See [here](https://tailscale.com/kb/1242/tailscale-serve/) for more information. | `https`                                  |
| `TAILSCALE_FUNNEL`        | Set this to `true` to enable [funnel](https://tailscale.com/kb/1243/funnel/). For more information about the accepted syntax, please read the [strconv.ParseBool documentation](https://pkg.go.dev/strconv#ParseBool) in the Go standard library.                                                             | `true`                                   |
| `TAILSCALE_BE_EXIT_NODE`  | Set this to `true` to advertise this container as an [exit host](https://tailscale.com/kb/1103/exit-nodes/).                                                                                                                                                                                                  | `true`                                   |
| `TAILSCALE_USE_EXIT_NODE` | Set this to an [exit host](https://tailscale.com/kb/1103/exit-nodes/) available in your tailnet. Either use the node's IP address or with [MagicDNS](https://tailscale.com/kb/1081/magicdns/) enabled, use the node's MagicDNS name.                                                                          | `se-sto-wg-001-mullvad.ts.net`           |

Something important to keep in mind is that you really should set up a
separate volume for Tailscale state. Here is how to do that with the
docker commandline:

```sh
docker volume create dokuwiki-tailscale
```

Then you can mount it into a container by using the volume name
instead of a host path:

```bash
docker run \
  ... 
  -v dokuwiki-tailscale:/var/lib/tailscale \
  ...
```

When using a docker composed file, you will need to declare the
volumes in the top level volumes section:

```yaml
volumes:
  dokuwiki-tailscale:
```

Then use it in a container:

```yaml
services:
  dokuwiki:
    volumes:
      - dokuwiki-tailscale:/var/lib/tailscale
```

If you don't do this, all Tailscale state will be lost when the
container reboots. This can range from inconvenience to annoying
because you can breach your Let's Encrypt rate limits with this.
