<!--
N.B.: This README was manually written for this unofficial package.
-->

# imapfilter Web UI for YunoHost

[![Install imapfilter Web UI with YunoHost](https://install-app.yunohost.org/install-with-yunohost.svg)](https://install-app.yunohost.org/?app=imapfilter_webui)

> *This package allows you to install imapfilter and its web frontend quickly and simply on a YunoHost server.
If you don't have YunoHost, please consult [the guide](https://doc.yunohost.org/admin/get_started/install_on/) to learn how to install it.*

## Overview

[imapfilter](https://github.com/lefcha/imapfilter) is a mail filtering utility that connects to IMAP accounts and moves, copies, deletes or flags messages based on rules written in Lua. This package ships it together with [imapfilter-web-ui](https://github.com/sebastianludwig/imapfilter-web-ui), a lightweight browser-based frontend that lets you edit the Lua configuration and start/stop imapfilter without needing SSH access.

### Features

- Edit your imapfilter Lua configuration directly in the browser
- Start and stop imapfilter from the web UI
- Enter IMAP passwords securely via the web UI (no plaintext credentials in config files required)
- Real-time log output streamed to the browser via Server-Sent Events
- Auto-restart imapfilter on connection drops (configurable)
- Protected by YunoHost SSOwat (admins-only by default)
- Runs as a systemd service with automatic restart on failure

**Shipped version:** 1.0~ynh1

**Upstream projects:**
- imapfilter: <https://github.com/lefcha/imapfilter>
- imapfilter-web-ui: <https://github.com/sebastianludwig/imapfilter-web-ui>

## Screenshots

![imapfilter Web UI main screen](https://raw.githubusercontent.com/sebastianludwig/imapfilter-web-ui/main/screenshots/main.png)

## Configuration

After installation the web UI is available at `https://yourdomain/imapfilter`. Click **Config** to edit the Lua filter rules.

### Example configuration

```lua
options.timeout   = 120
options.subscribe = true
options.create    = true

-- Password is entered interactively via the web UI input field
io.write("Type password for user@example.com in the input field and press Enter:\n")
io.flush()
local password = io.read()

account = IMAP {
    server   = "imap.example.com",
    port     = 993,
    username = "user@example.com",
    password = password,
    ssl      = "ssl23",
}

function filter_inbox()
    -- Move cron/system mail to a subfolder
    results = account.INBOX:contain_subject("Cron <") +
              account.INBOX:contain_from("root@") +
              account.INBOX:contain_field("X-Cron-Env", "SHELL")
    results:move_messages(account["INBOX/Cron"])
end

filter_inbox()

while true do
    account.INBOX:enter_idle()
    filter_inbox()
end
```

**Important notes for the configuration:**

- Use `contain_subject` / `contain_from` / `contain_field` (server-side IMAP SEARCH) rather than `match_*` (client-side regex) when combining results with `+` before calling `move_messages`.
- Always call `mark_seen()` **before** `move_messages()` — both operate on the same UIDs in the original mailbox.
- Use `io.write` + `io.read()` instead of `get_password()` — the web UI runs imapfilter via pipes (no TTY), so `get_password()` fails with an ioctl error.
- The prompt string passed to `io.write` must contain `Enter password for <user>@` if you want the web UI to automatically redirect to its dedicated password entry page instead of using the main input field.

## Disclaimers / important information

- **No LDAP / SSO integration for imapfilter itself** — IMAP credentials are entered manually at runtime via the web UI. The web UI interface is protected by YunoHost SSOwat.
- **Single instance only** — the package does not support multiple instances.
- **Sub-path deployment** — the app is served under a path (e.g. `/imapfilter`) on an existing domain, not a dedicated subdomain.
- **nginx sub_filter** — the upstream app uses absolute asset paths internally. The nginx config rewrites these via `sub_filter`. Gzip compression from the backend is disabled (`Accept-Encoding: ""`) to ensure sub_filter always applies.
- **imapfilter does not auto-start** — after a server reboot the systemd service starts, but imapfilter itself only begins filtering once you click **Start** in the web UI and enter your IMAP password.

## Documentation and resources

- Upstream imapfilter repository: <https://github.com/lefcha/imapfilter>
- Upstream web UI repository: <https://github.com/sebastianludwig/imapfilter-web-ui>
- imapfilter sample configs: <https://github.com/lefcha/imapfilter/tree/master/samples>
- YunoHost documentation: <https://doc.yunohost.org>
- Report a bug: <https://github.com/caspargross/imapfilter_webui_ynh/issues>

## Developer info

To install from this repository:

```bash
sudo yunohost app install https://github.com/caspargross/imapfilter_webui_ynh \
    --args "domain=yourdomain.tld&path=/imapfilter&init_main_permission=admins"
```

To upgrade an existing installation:

```bash
sudo yunohost app upgrade imapfilter_webui \
    -u https://github.com/caspargross/imapfilter_webui_ynh
```

**More info regarding app packaging:** <https://doc.yunohost.org/dev/packaging/>
