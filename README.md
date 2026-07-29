# SSL-TLS-Remediation
Before/after TLS/SSL audit reports generated with testssl.sh to document remediation of an SSL/TLS misconfiguration finding

# TLS/SSL Hardening – Remediation Evidence (testssl.sh)

TLS/SSL audit reports generated with `testssl.sh` to document the remediation of
vulnerability **KR-NTS-26-14-07 – SSL/TLS Misconfiguration** (severity: Low),
identified in the penetration test and tracked in the
*Vulnerability Remediation Tracker*.

Each endpoint has a **baseline (before)** report and an **`-after`** report,
providing evidence that the protocol/cipher configuration was fixed.

> ⚠️ **Security notice:** All hostnames, FQDNs, IP addresses, and rDNS entries in
> these reports have been replaced with reserved documentation placeholders
> (`192.0.2.0/24` – RFC 5737 TEST-NET-1, and `.invalid` – RFC 2606). No real
> address is exposed. This does not affect the methodology or the validity of
> the before/after comparison.

---

## 1. Why it was used

The pentest flagged "Weak SSL/TLS protocols or cipher suites are enabled"
(layer: Infrastructure / Middleware – Web Server / Reverse Proxy / Load Balancer).

We needed **objective, reproducible, local** evidence that would:

- list exactly which protocols (SSLv3 / TLS 1.0 / 1.1 / 1.2 / 1.3) and cipher
  suites each endpoint accepts;
- verify certificate validity and chain;
- test for known vulnerabilities (Heartbleed, POODLE, ROBOT, etc.);
- produce one artifact **before** the change and one **after**, to attach to
  the tracker as proof of remediation.

`testssl.sh` was chosen because it is a self-contained Bash script (it ships its
own OpenSSL to cover legacy ciphers), runs **on-premise** without sending data
to any external service, and produces HTML output ready for archiving — an
important requirement in a regulated environment.

---

## 2. Tool / version used

- testssl.sh **3.3dev** (commit `611b1b1f`, 2026-07-12)
- OpenSSL bundled with the tool itself (~179 ciphers covered)
- License: GPLv2
- Official site: https://testssl.sh

---

## 3. Installation

No compilation or system packages required; just clone the repository.

```bash
# via HTTPS
git clone --depth 1 https://github.com/testssl/testssl.sh.git
cd testssl.sh

# check version
./testssl.sh --version
```

Dependencies: only `bash` and common utilities (`sed`, `awk`, `hexdump`).
The correct OpenSSL binary already ships with the project under `./bin/`, so
there is **no need** to install OpenSSL separately.

---

## 4. How it was used

Each endpoint was tested on port **443**, producing an HTML report.

### 4.1 Baseline (before remediation)

```bash
./testssl.sh --html --logfile ./report-before/<TARGET>.html <FQDN_OR_IP>
```

### 4.2 Verification (after remediation)

Same command, pointing to the same endpoint, with an `-after` suffix on the file:

```bash
./testssl.sh --html --logfile ./report-before/<TARGET>-after.html <FQDN_OR_IP>
```

### Useful flags (reference)

```bash
-U                 # focus on known vulnerabilities
-p                 # protocols only
-E / -e            # ciphers only
--fast             # faster run (fewer handshakes)
--jsonfile x.json  # JSON output (for automation/parsing)
--sneaky           # less "aggressive" user-agent
```

---

## 5. Generated files

Structure under `report-before/`. Site names, IPs, and reverse DNS shown below
are **fictional placeholders** (see security notice above).

| Target (role)       | Before / After                     | Site (placeholder)  | IP (placeholder) | rDNS (placeholder)     |
|---------------------|------------------------------------|---------------------|------------------|------------------------|
| Application node #1 | `acc1.html` / `acc1-after.html`     | `acc1.example`      | `192.0.2.101`    | `acc1.example.invalid.`   |
| Application node #2 | `acc2.html` / `acc2-after.html`     | `acc2.example`      | `192.0.2.102`    | `acc2.example.invalid.`   |
| SSRS (PRD)          | `ssrs.html` / `ssrs-after.html`     | `ssrs.example`      | `192.0.2.103`    | `ssrs.example.invalid.`   |
| SSRS (QA)           | `ssrsqa.html` / `ssrsqa-after.html` | `ssrsqa.example`    | `192.0.2.104`    | `ssrsqa.example.invalid.` |
| Public site (www)   | `www.html` / `www-after.html`       | `www.example`       | `192.0.2.105`    | `www.example.invalid.`    |

Each `.html` is the full colorized dump of the test: protocols, cipher
categories, server preferences, certificate, HSTS/headers, and vulnerability
tests. Green = OK; red/orange shades = points of attention.

---

## 6. How to interpret / compare

1. Open the pair `<TARGET>.html` (before) and `<TARGET>-after.html` (after)
   side by side.
2. Check the **Testing protocols** and **Testing cipher categories** sections:
   - legacy protocols (SSLv3 / TLS 1.0 / 1.1) should show as `not offered (OK)`;
   - weak ciphers (NULL, EXPORT, RC4, 3DES, obsolete CBC) should not be
     `offered`.
3. Review the certificate (validity, key size, signature algorithm) and the
   vulnerability section.
4. The `-after` report is the evidence that the configuration was corrected.

---

## 7. How to reproduce

```bash
git clone --depth 1 https://github.com/testssl/testssl.sh.git
cd testssl.sh
mkdir -p report-before

for target in acc1 acc2 ssrs ssrsqa www; do
  ./testssl.sh --html --logfile ./report-before/${target}.html <FQDN_OF_${target}>
done
```

After applying the TLS hardening, repeat, changing the filename to
`${target}-after.html`.

---

## 8. Notes

- All IP addresses, FQDNs, and rDNS records in the reports were replaced with
  reserved documentation ranges (RFC 5737 `192.0.2.0/24` and RFC 2606
  `.invalid`), so no real infrastructure detail is disclosed.
- A full scan performs **many handshakes** per target — avoid running it against
  production during peak hours, and only run it against authorized hosts.
- The tool is distributed under **GPLv2**, with no warranty ("use at your own
  risk").
- Scope of this evidence: item **KR-NTS-26-14-07** of the remediation tracker.
