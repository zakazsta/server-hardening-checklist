# Server Hardening Checklist

A practical, no-nonsense checklist for hardening a Linux web server, with copy-paste
config snippets. Aimed at small VPS deployments (single host running web apps behind a
reverse proxy). Defensive only — protect the box, slow down brute-force and bot floods,
make recovery easy.

> Use what applies to your stack. Always back up a config before changing it, and keep a
> second session open so you don't lock yourself out of SSH.

## 1. SSH
- [ ] Disable password auth, use keys only (`sshd_hardening.conf`)
- [ ] Disable root login (`PermitRootLogin no`) or restrict to keys
- [ ] Keep SSH on a sane config; rely on fail2ban rather than port-hiding for real safety
- [ ] Limit auth attempts (`MaxAuthTries`, `LoginGraceTime`)

## 2. Firewall
- [ ] Default-deny inbound, allow only what you serve (80/443) and your admin path
- [ ] Don't expose database / app ports publicly — bind to `127.0.0.1`
- [ ] Document every open port and why it's open

## 3. fail2ban (brute-force / flood mitigation)
- [ ] Enable `sshd` jail
- [ ] Add a web jail for repeated 4xx/auth failures (`fail2ban/jail.local`)
- [ ] Set sane `bantime` / `findtime` / `maxretry`

## 4. Reverse proxy (Caddy / Nginx)
- [ ] Terminate TLS at the proxy, auto-renew certificates
- [ ] Security headers (HSTS, X-Content-Type-Options, Referrer-Policy)
- [ ] Rate limiting on auth endpoints and forms (`caddy/`, `nginx/`)
- [ ] Hide server version banners

## 5. Application
- [ ] Validate all input on the backend (never trust the client)
- [ ] Secrets in env / a secrets manager, never in code or git
- [ ] Parameterized queries (no string-built SQL)
- [ ] Lock admin routes behind a real auth/role check
- [ ] Friendly error pages — never leak stack traces to users

## 6. Updates & monitoring
- [ ] Unattended security updates
- [ ] Centralize and watch auth + web logs
- [ ] Alert on repeated bans / traffic spikes

## 7. Backups (the part everyone skips)
- [ ] Automated, scheduled, **off-host** backups
- [ ] Test a restore at least once — an untested backup is a guess
- [ ] Snapshot before any risky change

## Incident response (when a site is already under attack)
1. Stop the bleeding first: enable proxy/WAF "under attack" mode, rate-limit, block the
   offending IPs/subnets.
2. Back up current state before changing anything.
3. Then investigate root cause: read logs for the pattern, find and remove web shells /
   unauthorized accounts, patch the entry vector.
4. Restore from a known-good backup if integrity is in doubt.

See the snippet files in this repo for ready-to-adapt configs.

---
Maintained as part of independent IT services work (hosting, web development, defensive
infrastructure support). Contributions and corrections welcome.
