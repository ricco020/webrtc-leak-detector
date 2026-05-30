# webrtc-leak-detector

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Zero dependencies](https://img.shields.io/badge/dependencies-zero-brightgreen.svg)](#)
[![Browser based](https://img.shields.io/badge/runtime-browser-blue.svg)](#)

A **single HTML file** that detects WebRTC IP leaks in your browser —
even with a VPN active. No tracking, no analytics, no server roundtrip.
Just open the file (or host it anywhere) and run the test.

```text
WebRTC Leak Detector

Reference public IP (HTTP egress): 198.51.100.42
WebRTC candidates detected: 3

  • 192.168.1.42    (local)
  • a1b2c3.local    (mDNS-obfuscated)
  • 198.51.100.42   (public IP)

❌ Leak: your real public IP 198.51.100.42 is exposed via WebRTC.
```

## Why this tool exists

If you've ever wondered *"my VPN is on, my IP at ipify.org is masked — but
am I really safe?"*, this tool answers that one specific question: **does
your browser hand out your real IP to peers via WebRTC?**

A whole category of de-anonymization attacks relies on this: a malicious
ad, a compromised CDN, or a third-party script can silently open an
`RTCPeerConnection` and read your real IP via the ICE candidate
exchange — **without you noticing, without any user prompt**, and
without the VPN tunnel ever seeing the traffic.

Mainstream WebRTC leak testers (browserleaks.com, ipleak.net,
expressvpn.com/webrtc-leak-test) do detect this, but each of them ships
heavy analytics scripts. This file is **31 KB total, 0 third-party
calls except STUN servers and one IP-echo endpoint**, and is auditable
in 5 minutes by anyone reading the source.

For the full breakdown of how WebRTC leaks happen, what the mitigation
looks like per browser, and why mDNS obfuscation alone isn't always
enough, read the companion methodology:

**[DNS + WebRTC leak detection methodology — AnonymFlow](https://www.anonymflow.com/en/blog/dns-leak-test)**

## Run it

```bash
# Option 1: open the file locally
git clone https://github.com/<your-org>/webrtc-leak-detector
cd webrtc-leak-detector
open webrtc-detect.html         # macOS
xdg-open webrtc-detect.html     # Linux
start webrtc-detect.html        # Windows

# Option 2: host it on any static server
python3 -m http.server 8000
# then visit http://localhost:8000/webrtc-detect.html
```

Works in Chrome 75+, Firefox 60+, Safari 13+, Edge 79+.

## How it works

1. Creates an `RTCPeerConnection` with three free public STUN servers
   (Google's L-tier + Cloudflare).
2. Opens a data channel and calls `createOffer()` — this is what
   triggers the ICE candidate gathering.
3. Listens to `icecandidate` events for up to 4 seconds. Each candidate
   string is parsed to extract the IP.
4. Each IP is classified as **loopback** (`127.0.0.1`, `::1`), **local**
   (RFC 1918, fe80:, 169.254), **mDNS** (`*.local`), or **public**.
5. In parallel, an HTTP-egress lookup is done at `api.ipify.org` to
   compare with what WebRTC reveals.
6. **Verdict**:
   - No public IP in WebRTC candidates → no leak.
   - Public IP that matches your HTTP egress → ✅ no extra leak.
   - Public IP that matches your **real** IP behind the VPN → ❌ leak.
   - Public IPs that don't match anything obvious → ⚠️ investigate.

## Privacy

The only outbound calls during a test are:

| Endpoint                                | Purpose                |
|-----------------------------------------|------------------------|
| `stun.l.google.com:19302` (STUN)        | ICE candidate trigger  |
| `stun1.l.google.com:19302` (STUN)       | ICE candidate trigger  |
| `stun.cloudflare.com:3478` (STUN)       | ICE candidate trigger  |
| `api.ipify.org` (HTTPS)                 | Reference public IP    |

No fingerprinting, no cookies set, no requests to AnonymFlow servers.

## Limitations

- Some VPNs include their own WebRTC filtering inside their browser
  extension — if you use the **extension**, the test may show "no
  leak" simply because the extension intercepted the candidate before
  it reached your code. To test the *desktop app's tunneling alone*,
  disable the VPN browser extension first.
- Safari before iOS 14 returns mDNS candidates by default, which masks
  the real IP. This is correct behaviour, not a bug.
- The reference IP fetch (`api.ipify.org`) needs network access. If
  offline, the test still runs but you'll see "unknown" as the
  reference.

## Why a single file?

Because a privacy tool you can't read in 10 minutes is a privacy tool
you have to trust on faith. The whole detector — HTML, CSS, JS — fits
under 350 lines. Read it, understand it, host it on your own server
if you don't want to depend on someone else's.

## Companion resources

- [DNS leak CLI](https://github.com/your-org/dns-leak-detector-cli) — sister tool, OS-level DNS leak probing
- [AnonymFlow VPN security audit (9 tests)](https://www.anonymflow.com/en/blog/complete-vpn-security-audit) — broader checklist
- [AnonymFlow web-based DNS + WebRTC checker](https://www.anonymflow.com/en/tools/test-fuite-dns) — same logic, hosted
- [AnonymFlow public WiFi risks 2026](https://www.anonymflow.com/en/blog/public-wifi-risks-2026) — context when this leak matters most

## License

[MIT](LICENSE) — do whatever you want, including commercial use.

## Acknowledgements

- [AnonymFlow leak detection methodology](https://www.anonymflow.com/en/blog/dns-leak-test)
- [WebRTC IP leak issue thread (Chromium)](https://bugs.chromium.org/p/chromium/issues/detail?id=333752)
- [mDNS in WebRTC (RFC 6762)](https://datatracker.ietf.org/doc/html/rfc6762)

---

*Built and maintained by the team behind [AnonymFlow](https://www.anonymflow.com) — independent VPN / privacy tooling and research. Methodology and write-ups at [anonymflow.com/methodology](https://www.anonymflow.com/en/methodology).*
