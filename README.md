# ns8-leantime

Module id: `leantime`. Images are pinned in `build-images.sh` and updated through reviewed changes.

## Install

```bash
add-module ghcr.io/geniusdynamics/leantime:latest 1
```

The command returns the instance id, for example `leantime1`. Use the id NS8 returns.

## Configure

```bash
api-cli run configure-module --agent module/leantime1 --data - <<'EOF'
{
  "host": "leantime.example.org",
  "http2https": true,
  "lets_encrypt": false
}
EOF
```

| Parameter | Required | Type | Description |
|---|---|---|---|
| `host` | yes | string | Host name for the application, like 'leantime.domain.org' |
| `lets_encrypt` | yes | boolean | Request a valid Let's Encrypt certificate. |
| `http2https` | yes | boolean | Redirect all the HTTP requests to HTTPS |

## Configuration output

`get-configuration` returns: `host`, `lets_encrypt`, `http2https`.

## Images

| Image | Version |
|---|---|
| `docker.io/mariadb` | `13.0.2` |
| `docker.io/leantime/leantime` | `3.1.4` |

## Network and access

- Reserves 1 TCP port(s) on the node. The web application is published through Traefik on the configured host name.
- Runs rootless.
- Declares these NS8 authorizations: `traefik@node:routeadm`.

## Backup

The following paths are included in the NS8 backup (`imageroot/etc/state-include.conf`):

- `state/leantime.sql`
- `state/leantime.env`
- `state/leantime-db.env`
- `volumes/leanuserfiles`
- `volumes/leanpublic_userfiles`
- `volumes/leanplugins`

## Services

- `leantime-app.service`
- `leantime.service`
- `mariadb-app.service`

## Get the configuration

```bash
api-cli run get-configuration --agent module/leantime1
```

## Update and uninstall

```bash
api-cli run update-module --data '{"module_url":"ghcr.io/geniusdynamics/leantime:latest","instances":["leantime1"],"force":true}'
remove-module --no-preserve leantime1
```

## Testing

Continuous integration:

- `Publish images` builds the module image on every push.
- `Test module` installs the image on a QEMU NS8 node and runs the Robot Framework suite in `tests/`. It runs after `Publish images` succeeds on `dev` and on `main`.
- `Build API JSON Schema docs` and the security scan run alongside.

The suite installs the module, configures a host name, checks that the web application answers through Traefik, takes UI screenshots and removes the module.

Run it locally with the NS8 standard infrastructure described in the [ns8-github-actions README](https://github.com/NethServer/ns8-github-actions/blob/v1/README.md#running-tests-locally).

## Open items

Found by the NS8 module standards check. These must be resolved before a stable release:

- The module description is still the template text: write a description of at least 60 characters in ui/public/metadata.json
- tests/leantime.robot generated from template: set real configure payload and URL check
- metadata: documentation_url is a placeholder
- metadata: categories empty (use standard categories)
- metadata: description.en needs >= 60 real chars (+ it translation)
- mariadb-app.service: uses %S/state, kickstart uses %E/state
- leantime.service: uses %S/state, kickstart uses %E/state
- leantime-app.service: uses %S/state, kickstart uses %E/state
- discover-smarthost still present: remove if the app does not use SMTP
- README still has the template boilerplate
- mariadb-app.service: %S/state changed to %E/state (review the bind mounts that remain)
- leantime.service: %S/state changed to %E/state (review the bind mounts that remain)
- leantime-app.service: %S/state changed to %E/state (review the bind mounts that remain)
- imageroot/etc/state-include.conf: smarthost lines removed, check the unit still starts
- imageroot/actions/restore-module/80start_services: smarthost lines removed, check the unit still starts
- imageroot/actions/configure-module/80start_services: smarthost lines removed, check the unit still starts
- imageroot/actions/configure-module/10configure_environment_vars: smarthost lines removed, check the unit still starts
- imageroot/systemd/user/leantime-app.service: smarthost lines removed, check the unit still starts
- LEANTIME_IMAGE is used by the module but no image in the label provides it: leftover from the template?

## Upstream

Application documentation: https://docs.leantime.com/

## License

GPL-3.0-or-later. See [LICENSE](LICENSE).
