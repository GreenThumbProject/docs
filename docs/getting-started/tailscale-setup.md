# Tailscale Setup (retired)

!!! warning "This page is kept only so existing links do not break"
    GreenThumb no longer uses Tailscale. It was decommissioned on the Raspberry Pi node in **July 2026**
    and replaced by a self-hosted **WireGuard** hub-and-spoke network.

    **See [Remote Access (WireGuard)](vpn-setup.md) instead.**

Nothing in the current stack depends on Tailscale. The `TAILSCALE_AUTHKEY` environment variable and the
`tailscale_ip` database column are both gone: the column is now `device_ip`, and node enrolment is a
deliberate exchange of WireGuard public keys rather than an auth key baked into a boot file.
