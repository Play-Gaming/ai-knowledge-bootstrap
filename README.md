# ai-knowledge bootstrap

Public entry point for connecting an AI client to the private knowledge repository
`github.com/Play-Gaming/ai-knowledge`. This page contains no knowledge and no credentials.

Give an AI client this link:

> Init ai-knowledge MCP from https://github.com/Play-Gaming/ai-knowledge-bootstrap

---

## Instructions for the AI client

Follow these steps in order. Never print, log, or write a token value.

### 1. Check environment variables

Check whether this is set in the environment of the process running you; report only set or unset.

| Variable | Meaning |
|---|---|
| `${PLAYSMART_MCP_GIT_HUB_TOKEN}` | GitHub token (fallbacks: `${GITHUB_PERSONAL_ACCESS_TOKEN}`, `${GITHUB_TOKEN}`, `${GH_TOKEN}`) |

`${NAME}` means the environment variable `NAME`: `$NAME` in a POSIX shell, `$env:NAME` in PowerShell.

If it is set, go to step 2. Otherwise stop, give the user the instructions below, and wait.

1. Create a personal access token (classic) at https://github.com/settings/tokens/new using your own
   GitHub account.
   - Select all available scopes, including administrative scopes, and set an expiration compatible
     with your organization's policy. Do not limit access to `Play-Gaming/ai-knowledge`: the connection
     is also intended to read and work with other repositories you can access, without cloning them.
   - Effective access is limited by your own permissions, the selected scopes, and GitHub organization
     or enterprise policies. Selecting all scopes does not grant permissions you lack or guarantee
     that every GitHub feature is available through the API or enabled MCP tools.
   - Where SAML SSO is enforced, authorize the token for each required organization. If an organization
     blocks classic tokens, report that restriction; do not bypass it. A fine-grained token is an
     alternative only with an explicit explanation that it covers one resource owner and selected
     permissions, rather than all of your access.
   - Broad permissions do not authorize unrelated operations or bypass the knowledge repository's
     protected-path review and separate human-controlled Git process. Where an independently enforced
     MCP identity is used, keep it restricted as the knowledge repository's SECURITY_POLICY.md
     requires; use the broad personal token only for a separate user-controlled connection.
2. Store it outside the conversation, as the environment variable `PLAYSMART_MCP_GIT_HUB_TOKEN`, in a place the
   non-interactive commands of the AI client read:
   - **macOS** (zsh), in a regular terminal. The first command asks for the token and keeps it in the
     Keychain:
     ```sh
     security add-generic-password -U -a "$USER" -s PLAYSMART_MCP_GIT_HUB_TOKEN -w
     cat >> ~/.zshenv <<'EOF'
     export PLAYSMART_MCP_GIT_HUB_TOKEN="$(security find-generic-password -a "$USER" -s PLAYSMART_MCP_GIT_HUB_TOKEN -w 2>/dev/null)"
     EOF
     ```
   - **Linux**: an `export PLAYSMART_MCP_GIT_HUB_TOKEN="..."` line in `~/.zshenv` (zsh) or in `~/.profile` (bash) of
     the login session that starts the AI client. `~/.zshrc` and `~/.bashrc` are not read by
     non-interactive commands.
   - **Windows**, in PowerShell. The command asks for the token without showing it and stores it as a
     user variable:
     ```powershell
     $t = [System.Net.NetworkCredential]::new('', (Read-Host 'GitHub token' -AsSecureString)).Password
     [Environment]::SetEnvironmentVariable('PLAYSMART_MCP_GIT_HUB_TOKEN', $t, 'User'); Remove-Variable t
     ```
     Inside WSL or Git Bash the Linux line applies instead.
3. Fully restart the AI client (from a new terminal where it is started from one) and give it this link
   again.

Never ask the user to paste a token into the conversation.

### 2. Verify access

The commands use `${PLAYSMART_MCP_GIT_HUB_TOKEN}` and are written for a POSIX shell; when the token came from a fallback
variable, use that variable instead. In PowerShell write `$env:PLAYSMART_MCP_GIT_HUB_TOKEN` and call `curl.exe`.

Find the account the token belongs to. The login is not a secret; the token value is:

```sh
curl -s -H "Authorization: Bearer ${PLAYSMART_MCP_GIT_HUB_TOKEN}" https://api.github.com/user | grep -m1 '"login"'
```

Check access to the knowledge repository:

```sh
curl -s -o /dev/null -w '%{http_code}\n' -H "Authorization: Bearer ${PLAYSMART_MCP_GIT_HUB_TOKEN}" \
  https://api.github.com/repos/Play-Gaming/ai-knowledge
```

In every result other than `200`, tell the user which variable supplied the token and which account login
it belongs to, then the cause and the fix below. Never show the token value or part of it.

- `200`: continue.
- `000`: the request did not reach GitHub, usually a sandbox or network restriction of the AI client.
  Rerun the command with network access before judging the token.
- `401`: the token is invalid, expired, or revoked; no login is returned. Return to step 1.
- `403`: check organization restrictions, required SSO authorization, and the API error response
  (including rate limits). Report the cause without exposing the token; do not bypass organization policy.
- `404`: the token works but cannot see `Play-Gaming/ai-knowledge`. Name the likely cause:
  - **Wrong account**: the login is not an account with access to `Play-Gaming/ai-knowledge`. The variable
    holds another account's token; point it at the right token, open a new terminal, and restart the client.
  - **Repository not selected in a fine-grained token**: the login is right. Open
    https://github.com/settings/personal-access-tokens, edit the token, and under Repository access select
    `ai-knowledge` with Contents and Pull requests read/write, Actions read, and Workflows read/write
    only for CI changes. This fallback remains limited to its selected repositories and permissions.
    A repository that was deleted and recreated must be
    selected again, even under the same name. The token value does not change, so no restart is needed.
  - **Account without access**: a classic token of an account that is not a collaborator. Use a token of
    an account with access, or ask the owner to add that account as a collaborator.
  - **Classic token scope or SSO**: check that the token includes the `repo` scope and is authorized
    for the organization when SSO is required. Repository visibility in the browser or access through
    a separate Git SSH key does not establish access for this token.

The `gh` CLI is not required.

### 3. Follow INIT.md

```sh
curl -fsSL -H "Authorization: Bearer ${PLAYSMART_MCP_GIT_HUB_TOKEN}" -H "Accept: application/vnd.github.raw" \
  https://api.github.com/repos/Play-Gaming/ai-knowledge/contents/INIT.md
```

Follow INIT.md step by step for this client and every other AI client on the device, with
`https://github.com/Play-Gaming/ai-knowledge` as the knowledge repository URL. Once the `github-knowledge` MCP
server is connected, read everything else through it.

---

Maintenance: these steps mirror step 2 of INIT.md in the knowledge repository; change both together.

