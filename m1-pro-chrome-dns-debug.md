# M1 Pro Chrome DNS Issue -- Handoff for Deep Research

**Date:** 2026-04-12
**Machine:** M1 Pro (Maxs-Laptop.local) -- 100.85.64.20 via Tailscale
**SSH access:** `ssh -i ~/.ssh/id_ed25519 maxdekwaasteniet@100.85.64.20`

## Problem

Chrome on M1 Pro cannot load any websites. Pages hang indefinitely on loading. Safari works fine.

## What We Know

### Network is functional
- Ping to router (192.168.1.254): works, ~1ms
- Ping to 1.1.1.1: works, ~11ms
- DNS via Glide servers (130.43.187.40/41): works, resolves instantly
- DNS via Cloudflare (1.1.1.1): works
- Safari: loads pages fine
- curl: times out (HTTP 000 after 10s)
- Chrome: hangs on all pages

### DNS resolver ordering (the suspected cause)
The system has multiple DNS resolvers in this priority order:

```
resolver #1 (Tailscale - BROKEN)
  nameserver: 100.100.100.100
  nameserver: fd7a:115c:a1e0::53
  interface: utun4
  order: 111600
  flags: Supplemental

resolver #2 (Glide ISP - WORKING)
  nameserver: 130.43.187.40
  nameserver: 130.43.187.41
  interface: en17
  order: 200000

resolver #3 (Tailscale duplicate - BROKEN)
  nameserver: 100.100.100.100
  nameserver: fd7a:115c:a1e0::53
  interface: utun4
  order: 111601
```

Tailscale DNS (100.100.100.100) is the primary resolver and it **times out** on external queries like `dig @100.100.100.100 google.com`. Glide's DNS works fine but has lower priority (higher order number).

### Why Safari works but Chrome doesn't
Safari uses macOS's native `mDNSResponder` which appears to fall back from broken Tailscale DNS to Glide DNS quickly. Chrome (and curl) use different resolution paths that get stuck on the Tailscale timeout.

### What we already tried (none fixed Chrome)
1. **Killed OrbStack** -- freed ~1.9GB RAM (machine was memory-starved at 660MB free with 86M swapouts). Chrome still broken.
2. **Flushed DNS cache** -- `dscacheutil -flushcache` + `killall -HUP mDNSResponder`. No effect.
3. **Opened captive portal in Safari** -- Glide captive portal passed successfully. Safari works after this. Chrome still broken.
4. **Disabled Chrome's built-in DNS client** -- `defaults write com.google.Chrome BuiltInDnsClientEnabled -bool false`. Killed and relaunched Chrome. Still broken.
5. **Opened chrome://net-internals/#dns and chrome://flags** -- no manual action taken on those pages.

### System context
- macOS on M1 Pro, 16GB RAM
- Glide ISP (UK apartment building broadband, uses captive portal)
- Tailscale active (needed for SSH access from M4 Air)
- Surfshark VPN installed (running since March 26, ~113MB RAM)
- Ollama running (Gemma model loaded, ~380MB)
- WiFi config: DHCP, IP 192.168.1.64, router 192.168.1.254
- DNS set to 1.1.1.1 / 1.0.0.1 in network settings (but Tailscale overrides resolver order)
- Chrome was running with `--remote-debugging-port=9222` (two instances, one with `--user-data-dir=/tmp/chrome-cdp`)

## Research Directions

1. **Why does Tailscale DNS (100.100.100.100) fail to resolve external domains?** MagicDNS should forward non-Tailscale queries upstream. Is it misconfigured? Is the Glide captive portal blocking Tailscale's upstream forwarder?

2. **Why does disabling Chrome's built-in DNS client NOT help?** After `BuiltInDnsClientEnabled = false` and full Chrome restart, it should use the system resolver (same as Safari). But it still hangs. Is Chrome ignoring the pref? Is there another layer?

3. **Is Surfshark VPN interfering?** It's been running since March 26. Could it be adding routing rules or DNS interception that affects Chrome but not Safari? Check `utun` interfaces and routing table.

4. **Chrome profile corruption?** Two Chrome instances were running (one normal, one CDP with `/tmp/chrome-cdp` profile). Could socket/cache corruption from the swap-heavy session be causing issues even after restart?

5. **Is curl also broken for the same reason as Chrome?** curl timed out with HTTP 000. If the system resolver works for Safari but not curl, the issue might be deeper than just Chrome -- could be related to which network interface/route is used for outbound connections.

6. **Possible fix: lower Tailscale DNS priority without disabling it.** Can Tailscale's DNS be set to supplemental-only so it doesn't override the primary resolver? Check `tailscale set --accept-dns=false` vs MagicDNS admin toggle.

7. **Nuclear option:** Quit Surfshark, toggle Tailscale `accept-dns` off, flush DNS, restart Chrome. But be careful -- Tailscale must stay running for SSH access.

## Commands for quick verification

```bash
# Test if DNS works now
dig @130.43.187.40 google.com +short +time=3
dig @100.100.100.100 google.com +short +time=3
curl -s -o /dev/null -w 'HTTP %{http_code} in %{time_total}s\n' --connect-timeout 10 https://google.com

# Check resolver order
scutil --dns | head -40

# Check Chrome DNS pref
defaults read com.google.Chrome BuiltInDnsClientEnabled

# Check Tailscale DNS setting
tailscale status
tailscale dns status

# Check routing table
netstat -rn | head -30

# Check Surfshark interfaces
ifconfig | grep -A2 utun
```
