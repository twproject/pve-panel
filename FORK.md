# PVE Cloud Panel

This is an independent MIT-licensed fork of [PVE Panel](https://github.com/sebastianflint/pve-panel), based on upstream v1.16.0. The original copyright and license remain in [LICENSE](LICENSE). The panel name is configurable with `PANEL_NAME`.

## Included additions

- **LXC access VLANs:** administrators can set an optional tag from 1 to 4094 on each offered container image. Creation and reinstall preserve the configured bridge, VLAN and disk storage. Customer SDN networking takes precedence when enabled. See the container section in [README](README.md).
- **Customer root-password recovery:** an opt-in, owner-checked action for running, unlocked, unprivileged containers. A separate restricted helper on the Proxmox node independently checks the allowed pool and uses a dedicated forced-command SSH key. See [setup and restrictions](docs/ct-password-reset.md).
- **Generate during password recovery:** reuses the initial deployment form's password generator, fills both confirmation fields and still requires confirmation before applying a change.
- **Native LXC terminals:** xterm.js uses Proxmox termproxy, with selection/copy, Unicode paste, automatic resizing and reconnect. Terminal tickets remain server-side and panel sessions are owner-bound and single-use. Virtual machines retain their noVNC graphical console. See [browser consoles](docs/browser-consoles.md).

All added features are configurable. VLAN tags default to untagged and root-password recovery defaults to disabled. Existing guest network settings are not rewritten when an image is edited. Root recovery preserves guest files and SSH keys and does not change the guest's SSH login policy.

## Build and configure

Use Node.js 22.13 or newer, or build the Docker image from this repository:

```sh
docker build -t pve-cloud-panel:local .
```

Use the upstream deployment instructions and Compose examples with `pve-cloud-panel:local` as the image. This source fork does not require images from an upstream release to contain its additions. Keep configuration and secrets outside the source tree; `example.env` contains placeholders only.

The optional password-reset helper needs Python 3 on the Proxmox node, a root-owned configuration file and a dedicated restricted SSH key. The panel image includes the SSH client. Follow the helper guide before enabling the feature; a regular management key is not suitable.

## Verification

```sh
npm ci
npm test
python3 -m unittest discover -s test/helper
npx playwright install --with-deps chromium
npm run test:e2e
```

The unit/API suite uses simulated Proxmox. Browser tests additionally cover the CT terminal, clipboard, resizing, reconnect, graphical VM fallback and password-generation confirmation. The helper has independent Python tests. Verify isolation and allowed actions on a disposable or explicitly authorized guest before using a new deployment.

## Maintain the fork

Keep the original repository as an upstream remote, fetch its changes and review/merge them with the fork's tests before updating an installation. Preserve the VLAN and recovery features when replacing an image. Do not force-push over unrelated work.

The inherited Docker workflow derives the image name from the current GitHub repository. Fork workflows may need to be enabled in GitHub Actions before builds can run. The inherited documentation website belongs to the original project; this fork's specific setup is documented here and in `docs/`.
