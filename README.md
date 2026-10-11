# APT Repository

[![Publish](https://github.com/HansTanzi/apt/actions/workflows/publish.yml/badge.svg)](https://github.com/HansTanzi/apt/actions/workflows/publish.yml)

Personal APT repository for Debian/Ubuntu, served from GitHub Pages.

## Apps

| App | Package | Description |
| --- | --- | --- |
| [RestGuard](https://github.com/HansTanzi/RestGuard) | `rest-guard` | A cross-platform break reminder that actually makes you rest |

## How do I install these packages?

Add the repository once:

```sh
sudo install -d /etc/apt/keyrings
curl -fsSL https://hanstanzi.github.io/apt/hanstanzi.gpg | sudo tee /etc/apt/keyrings/hanstanzi.gpg > /dev/null
echo "deb [signed-by=/etc/apt/keyrings/hanstanzi.gpg] https://hanstanzi.github.io/apt stable main" | sudo tee /etc/apt/sources.list.d/hanstanzi.list
sudo apt update
```

Then install a package, e.g. `sudo apt install rest-guard`. Updates arrive with `sudo apt update && sudo apt upgrade`.

## How does it work?

[`publish.yml`](.github/workflows/publish.yml) downloads the `.deb` files from the latest release of every repository listed in [`apps.txt`](apps.txt), builds and signs the index, and deploys it to GitHub Pages. Only the latest version of each app is served; no `.deb` is stored in git.

It runs when an app's release workflow sends a `release` [repository dispatch](https://docs.github.com/en/rest/repos/repos#create-a-repository-dispatch-event), when `apps.txt` changes, or by hand.

### Setup

- **Pages**: Settings → Pages → Source: **GitHub Actions**.
- **`APT_SIGNING_KEY` secret**: an ASCII-armored GPG private key without a passphrase:

  ```sh
  gpg --batch --passphrase '' --quick-gen-key "HansTanzi APT" ed25519 sign never
  gpg --armor --export-secret-keys "HansTanzi APT"
  ```

  Back it up somewhere safe and never replace it: users trust this key when they add the repository.
- **Dispatch token in each app repository**: a fine-grained PAT limited to this repository with **Contents: Read and write**, stored as `APT_DISPATCH_TOKEN`.

### Adding an app

Add a line `<owner>/<repo> <asset pattern>` to `apps.txt`, and have the app's release workflow trigger this repository after the `.deb` is uploaded:

```sh
gh api repos/HansTanzi/apt/dispatches -f event_type=release
```
