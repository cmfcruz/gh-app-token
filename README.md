# gh-app-token

Mint a GitHub App installation token with a Bash script.

## Requirements

- Bash 3.2+
- `curl`
- `openssl`
- `jq`

## Install

Download the single script directly from GitHub:

```sh
mkdir -p "$HOME/.local/bin"
curl -fsSL https://raw.githubusercontent.com/cmfcruz/gh-app-token/main/gh-app-token -o "$HOME/.local/bin/gh-app-token"
chmod 755 "$HOME/.local/bin/gh-app-token"
```

Add `$HOME/.local/bin` to your `PATH` if it is not there already. In a Docker image, install the required commands and copy the same script into `/usr/local/bin` during the build. For example, on Alpine:

```dockerfile
RUN apk add --no-cache bash curl openssl jq \
 && curl -fsSL https://raw.githubusercontent.com/cmfcruz/gh-app-token/main/gh-app-token -o /usr/local/bin/gh-app-token \
 && chmod 755 /usr/local/bin/gh-app-token
```

Do not copy the private key into an image; mount it at runtime instead. For a reproducible or security-sensitive build, pin the download URL to a reviewed commit rather than `main`.

## Use

Create a GitHub App, install it on the repositories you need, and download its RSA private key. Give the app only the repositories and permissions needed for your work.

```sh
export GITHUB_APP_ID=123456
export GITHUB_APP_INSTALLATION_ID=789012
export GITHUB_APP_PRIVATE_KEY_FILE="/path/to/downloaded-key.pem"

GITHUB_TOKEN="$(gh-app-token)" gh pr view
```

`gh` is only an example consumer; `gh-app-token` prints the raw token and a newline, so it works with any tool that accepts a GitHub token.

## Troubleshooting

If GitHub CLI remains unauthenticated, run `gh-app-token` directly. It should print a token; otherwise, correct the reported configuration, signing, or GitHub API error.

## How it works

The script caches each installation token at `~/.cache/gh-app-token/<app-id>-<installation-id>`. It reuses a token only while the file's modification time is 0–1799 seconds old; otherwise it mints a new one. Its `umask 077` creates cache files as `0600`, and the cache directory is enforced as `0700`. Treat this cache as a secret: keep it out of backups, logs, and version control, along with the private key. In Docker, mount a writable home or cache directory if you want the cache to survive container replacement; do not bake the key or cache into an image.
