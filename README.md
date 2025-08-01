About This Fork: n8n Home Assistant Add-on Hotfix

This fork of the n8n Home Assistant add-on exists to fix a critical bug affecting all users of the official add-on after n8n version 1.104.1. The bug causes connection errors and UI failures due to incorrect subpath handling for `config.js` and related assets when served via Home Assistant Ingress.

## Why this fork?

**Upstream Issue:**
After updating to n8n 1.104.1, users consistently see "Error connecting to n8n" and requests to URLs like `/undefinedrest/settings`, breaking the UI and API. This is tracked upstream:

- [n8n-io/n8n Issue #17817](https://github.com/n8n-io/n8n/issues/17817)
- [Rbillon59/hass-n8n Issue #359](https://github.com/Rbillon59/hass-n8n/issues/359)

**Root Cause:**
The n8n frontend fails to detect the correct subpath for config.js when running behind Home Assistant Ingress, resulting in broken asset URLs and connection errors.

**Hotfix:**
This fork uses the patched Docker image: `ghcr.io/n8n-io/n8n:branch-account-for-subpath-when-serving-configjs`, which includes a fix for subpath handling. This resolves the connection and UI issues for Home Assistant users.

# Installation

Follow these steps to get the add-on installed on your system:

**Important:** Make sure you've added this addon repository to your Home Assistant addon library: https://github.com/HampusAndersson01/hass-n8n-hotfix

1. Navigate in your Home Assistant frontend to **Supervisor** -> **Add-on Store**.
2. Find the "hass-n8n" add-on and click it.
3. Click on the "INSTALL" button.

# Configuration

```yaml
timezone: Europe/Berlin
env_vars_list: []
cmd_line_args: ""
```

## Option: `env_vars_list` (required)

List of the n8n environment variables. You can add as many environment variables as you want to the list through the UI. The format is the following:

`SOME_ENVIRONMENT_VARIABLE: some-value` (the regular expression is `^[A-Z_0-9]+: .*$` )

All the available environment variables are available here : <https://docs.n8n.io/hosting/environment-variables/environment-variables/>

## Option: `cmd_line_args` (optional)

The command line to start n8n. If you want to use a custom command line, you can use this variable.

## Installing external packages

In n8n you can add external node modules by setting the `NODE_FUNCTION_ALLOW_EXTERNAL` environment variable with the list of npm packages you need.

For example, to install the `lodash` and the `moment` packages, in the UI, set the `env_vars_list` variable to:

```txt
NODE_FUNCTION_ALLOW_EXTERNAL: lodash,moment
```

## Setting up external access

The addon is presented via a [Home Assistant Ingress](https://www.home-assistant.io/blog/2019/04/15/hassio-ingress/) for additional security.

This means that in addition to n8n's credentials, n8n can't be accessed without being authenticated with Home Assistant as well, even when accessing it remotely.

### Nabu Casa remote URL

If you use Nabu Casa remote URL, the addon can't fetch your Nabu Casa remote URL programmatically, in that case, you need to set the `EXTERNAL_URL` environment variable to your Nabu Casa remote URL for things like OAuth2 credentials to work properly (setting the right redirect URLs).

### Manual external URL

If you are using a manual external URL, Ingress traffic is not relayed via this in a safe manner. In that case, you need to expose port `5678` of the container to a certain port (for instance, `5678`) on Home Assistant, under the "Network" section of the configuration tab of the addon.

### Webhooks, triggers and n8n API

Webhooks, triggers and the n8n API can't go via an Ingress, since having Home Assistant authentication on top of it would break the webhook (they need to be publically accessible and unauthenticated).

For this reason, the addon exposes all webhook or API traffic on port `8081`.

For the n8n API, webhooks and some webhook-based triggers to work properly, you need to open up a tunnel for port `8081` to the internet via the [Cloudflared addon](https://github.com/brenner-tobias/addon-cloudflared) or something similar.

When done, set the `WEBHOOK_URL` environment variable to the URL of the tunnel.

### Bypassing Home Assistant Ingress entirely (not recommended)

If you want to, you can expose n8n without requiring to go through the Ingress. This is done in the "ports" section of the "configuration" tab of the addon, for port `5678`. Note that you need to enable "hidden port bindings" for that port to show up. It is not exposed by default.

# How to use it?

Just start the addon and head to the addon's web UI.

## Useful ressources

### n8n documentation

<https://docs.n8n.io>
<https://docs.n8n.io/getting-started/tutorials.html>

### Community public workflows

<https://n8n.io/workflows>

### Available integrations

<https://n8n.io/integrations>

## License

This addon is published under the Apache 2 license. Original author of the addon's bundled software is n8n.

# Troubleshooting

Got questions?

You can open an issue on GitHub.

Repository: <https://github.com/Rbillon59/hass-n8n>

## `401: Unauthorized` in popup-window when trying to set up OAuth-based credential

This can happen depending on which browser you are using. To work around it, copy the URL of the popup window and paste it into a new tab in the main window. Then the authorization will complete.
