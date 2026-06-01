# DNS blocking and bypass with DNS over TLS

## What it is

This note is about a case where `reddit.com` would not resolve on Linux, even though other domains like `google.com` worked normally.

The error was:

```bash
ping: reddit.com: No address associated with hostname
```

That pointed to a DNS resolution problem, not a general internet problem.

## Why it matters

This is a useful example of how DNS failures can be selective. A site may exist and be reachable, but the configured DNS path can still make it look like the domain does not exist.

It is also a practical example of ISP-level DNS blocking and how encrypted DNS can bypass it.

## Key idea

The system was using `systemd-resolved`, which forwarded DNS queries through the normal resolver path. The working theory and observed behavior were:

- `google.com` resolved normally
- `reddit.com` failed only through the default resolver path
- `reddit.com` resolved successfully when queried against `8.8.8.8`

That means the domain itself was fine, but the default DNS path was likely being filtered or manipulated.

## Investigation

### Symptom

```bash
ping reddit.com
```

Result:

```bash
ping: reddit.com: No address associated with hostname
```

### Test with another resolver

```bash
nslookup reddit.com 8.8.8.8
```

This succeeded, which showed that:

- the domain existed
- the network connection was generally fine
- the problem was tied to the default DNS path

### Check local resolver setup

```bash
cat /etc/resolv.conf
```

This showed the system using the local stub resolver:

```txt
nameserver 127.0.0.53
```

That means applications were asking `systemd-resolved`, and that resolver was forwarding queries onward.

## Explanation

DNS blocking often works by intercepting queries for specific domains and returning a false answer such as:

- `NXDOMAIN`
- no answer
- a wrong IP

In this case, the likely behavior was selective DNS blocking for `reddit.com`.

## Fix

The fix was to use DNS over TLS with Google Public DNS through `systemd-resolved`.

Configuration file:

```ini
/etc/systemd/resolved.conf.d/dns_servers.conf
```

Content:

```ini
[Resolve]
DNS=8.8.8.8 8.8.4.4
DNSOverTLS=yes
```

Then restart the resolver:

```bash
sudo systemctl restart systemd-resolved
```

## Why this worked

Normal DNS is unencrypted, so an ISP can inspect and alter those queries.

DNS over TLS encrypts the DNS request and response, which means:

- the ISP can still see that traffic exists
- the ISP cannot easily read the queried domain name
- simple DNS filtering becomes much harder

## Verification

After the change:

```bash
ping reddit.com
```

This resolved successfully and returned an IP for `reddit.com`.

## Limits

DNS over TLS helps against DNS-level blocking, but it does not solve every kind of filtering.

It does not protect against:

- direct IP blocking
- deep packet inspection on other traffic layers
- traffic analysis
- destination IP visibility

## Alternatives

Other approaches that can solve similar problems:

- DNS over HTTPS
- VPN
- Tor
- manually changing DNS without encryption

DNS over TLS is a good middle ground because it is simple, fast, and usually enough when the issue is DNS manipulation.

## Commands

Check DNS status:

```bash
resolvectl status
cat /etc/resolv.conf
```

Test with a known resolver:

```bash
nslookup reddit.com 8.8.8.8
dig @8.8.8.8 reddit.com
```

Apply the fix:

```bash
sudo mkdir -p /etc/systemd/resolved.conf.d
printf '[Resolve]\nDNS=8.8.8.8 8.8.4.4\nDNSOverTLS=yes\n' | sudo tee /etc/systemd/resolved.conf.d/dns_servers.conf
sudo systemctl restart systemd-resolved
```

## Pitfalls

- Changing DNS to `8.8.8.8` without encryption may still leave DNS visible to the ISP.
- DNS over TLS can fail if port `853` is blocked.
- Using encrypted DNS shifts trust from the ISP to the DNS provider you choose.
- If the block happens at the IP or routing layer, DNS changes alone will not help.

## Related

- `systemd-resolved`
- DNS hijacking
- DNS over HTTPS
- VPN
