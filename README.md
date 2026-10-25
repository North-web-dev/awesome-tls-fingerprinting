# Awesome TLS / HTTP2 Fingerprinting [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> A curated list of resources for **TLS and HTTP/2 client fingerprinting** — the specs (JA3, JA4+, Akamai HTTP/2), the libraries that impersonate real browsers, the tools that measure what you actually send, and the research behind it all.

Fingerprinting looks at *how* a client speaks TLS and HTTP/2 — the ClientHello cipher/extension ordering, the HTTP/2 SETTINGS frame, header order — rather than *what* it claims to be in its `User-Agent`. A mismatch between the two is how most anti-bot systems flag automated traffic. This list covers both sides: generating a convincing fingerprint and detecting a forged one.

Every link below was checked to resolve (HTTP 200/301) at the time of writing. Entries are described honestly — language, approach, and what each thing actually is.

## Contents

- [Specs & concepts](#specs--concepts)
- [Libraries — client impersonation](#libraries--client-impersonation)
- [Tools — inspect & measure](#tools--inspect--measure)
- [Datasets](#datasets)
- [Defensive & detection](#defensive--detection)
- [Reflectors & test endpoints](#reflectors--test-endpoints)
- [Articles & research](#articles--research)
- [Related awesome lists](#related-awesome-lists)

## Specs & concepts

- [JA3](https://github.com/salesforce/ja3) — Salesforce's original TLS client fingerprint: an MD5 hash of five ClientHello fields (version, ciphers, extensions, elliptic curves, curve formats). The repo holds the reference spec and extractors for Zeek, Python, and Rust. See also the [Salesforce engineering write-up](https://engineering.salesforce.com/tls-fingerprinting-with-ja3-and-ja3s-247362855967/).
- JA3S — the server-side counterpart to JA3, hashing the ServerHello (version, chosen cipher, extensions). Documented and implemented in the same [salesforce/ja3](https://github.com/salesforce/ja3) repository; pairing JA3 with JA3S identifies a full client↔server session.
- [JA4+](https://github.com/FoxIO-LLC/ja4) — FoxIO's modern successor suite. Unlike JA3's single hash, JA4+ is a family of human-readable fingerprints: **JA4** (TLS client), **JA4S** (TLS server), **JA4H** (HTTP client), **JA4L** (light-distance/latency), **JA4T** (TCP), and **JA4X** (X.509 certificate). Full spec per fingerprint lives in the repo's `technical_details/` folder — e.g. [JA4.md](https://github.com/FoxIO-LLC/ja4/blob/main/technical_details/JA4.md) and [JA4H.md](https://github.com/FoxIO-LLC/ja4/blob/main/technical_details/JA4H.md) — with the rest covered in the [repo README](https://github.com/FoxIO-LLC/ja4#readme). Maintained by [FoxIO](https://foxio.io/). A clean-room Go implementation of the full suite: [ja4plus-go](https://github.com/North-web-dev/ja4plus-go).
- [Akamai HTTP/2 fingerprinting](https://www.blackhat.com/docs/eu-17/materials/eu-17-Shuster-Passive-Fingerprinting-Of-HTTP2-Clients-wp.pdf) — the Black Hat EU 2017 whitepaper by Elad Shuster (Akamai) that defines passive HTTP/2 client fingerprinting from the SETTINGS frame, WINDOW_UPDATE, stream priorities, and pseudo-header order. The basis for the widely used "Akamai h2 fingerprint" string.
- [peetprint](https://github.com/pagpeter/TrackMe) — a composite TLS fingerprint format popularised by the tls.peet.ws reflector (source: `pagpeter/TrackMe`). It captures ClientHello details in a readable, GREASE-tolerant form and is commonly reported alongside JA3/JA4 when validating browser impersonation.

## Libraries — client impersonation

- [utls](https://github.com/refraction-networking/utls) — **Go**. A fork of the standard `crypto/tls` that exposes and lets you forge the ClientHello, so handshakes match named browser profiles (Chrome, Firefox, Safari, iOS) byte-for-byte. The low-level building block most other Go impersonation tools sit on top of.
- [curl_cffi](https://github.com/lexiforest/curl_cffi) — **Python**. CFFI bindings to `curl-impersonate`, giving a `requests`-style API that reproduces real browser JA3/JA4 and HTTP/2 fingerprints. Actively maintained under the `lexiforest` org (formerly `yifeikong`).
- [curl-impersonate](https://github.com/lexiforest/curl-impersonate) — **C / curl**. A patched build of curl linked against browser TLS libraries (BoringSSL/NSS) with matching cipher, extension, and HTTP/2 settings, so the command-line `curl` impersonates Chrome/Firefox/Edge/Safari. Also see the original [lwthiker/curl-impersonate](https://github.com/lwthiker/curl-impersonate).
- [tls-client](https://github.com/bogdanfinn/tls-client) — **Go**. A higher-level HTTP client built on utls with a large catalogue of pre-built browser profiles and a simple request API; also shipped as shared libraries for use from other languages.
- [impersonate-http](https://github.com/North-web-dev/impersonate-http) — **Go**. An `http.Client` with a byte-exact browser TLS fingerprint (via uTLS), aimed at surviving JA3/JA4 blocking without spinning up a headless browser.
- [impersonate-rs](https://github.com/North-web-dev/impersonate-rs) — **Rust**. Byte-exact browser TLS (JA4) via BoringSSL — the Rust counterpart to impersonate-http. Chrome profile verified against tls.peet.ws.

## Tools — inspect & measure

- [fpkit](https://github.com/North-web-dev/fpkit) — a single Go binary (`fp`) covering both sides of the wire: `fp get` fetches as a byte-exact browser, `fp inspect` reflects a caller's JA3/JA4/JA4H/Akamai-h2 with a plain-English verdict, `fp check` is a CI drift guard, `fp guard` flags non-browser inbound traffic, `fp hook` is a webhook inspector that shows the caller's fingerprint, and `fp db ja4 <value>` reverse-looks-up a JA4 against bundled browser + known-client fingerprints.
- [fpcheck](https://github.com/North-web-dev/fpcheck) — a self-hostable TLS/HTTP2 fingerprint tester and differ: it reports your JA3/JA4/JA4H and Akamai HTTP/2 fingerprint and diffs them against real browser profiles so you can see exactly where an impersonation attempt leaks.
- [tls.peet.ws](https://tls.peet.ws/) — a hosted reflector that shows your JA3, JA4, peetprint, Akamai HTTP/2 fingerprint and raw ClientHello in the browser. The quickest way to see what a given client actually sends.
- [BrowserLeaks — TLS](https://browserleaks.com/tls) — a well-known browser-privacy test page that reports your TLS fingerprint (JA3/JA4), cipher suites, and extensions.
- [Scrapfly — JA3 fingerprint tool](https://scrapfly.io/web-scraping-tools/ja3-fingerprint) — a free hosted tool that computes and explains your JA3 fingerprint.
- [Scrapfly — HTTP/2 fingerprint tool](https://scrapfly.io/web-scraping-tools/http2-fingerprint) — companion tool that reports your Akamai-style HTTP/2 fingerprint.

- [fingerprint-check-action](https://github.com/North-web-dev/fingerprint-check-action) — a GitHub Action that runs your HTTP client against a reflector in CI and fails the build if its JA4/JA3/Akamai fingerprint drifts from the expected browser profile — a regression guard for impersonation setups.

## Datasets

- [fingerprint-db](https://github.com/North-web-dev/fingerprint-db) — measured browser TLS (JA3/JA4) and HTTP/2 (Akamai) fingerprints as JSON, with Go and Python loaders. Intended as a reference corpus for building and validating impersonation tooling.
- [cdn-ja4-policy-db](https://github.com/North-web-dev/cdn-ja4-policy-db) — open, sourced dataset of how CDN/WAF/anti-bot vendors (Cloudflare, Akamai, DataDome, …) treat TLS/JA4 client fingerprints; every entry carries provenance and a `basis` (capability vs observed).
- [JA4DB](https://ja4db.com/) — FoxIO's community database mapping JA4+ fingerprints to the applications and malware that produce them.
- [SSLBL JA3 fingerprints (abuse.ch)](https://sslbl.abuse.ch/ja3-fingerprints/) — abuse.ch's curated list of JA3 fingerprints associated with malware and botnet C2 traffic.
- [salesforce/ja3 — lists](https://github.com/salesforce/ja3/tree/master/lists) — reference JA3 fingerprint lists shipped alongside the original spec.

## Defensive & detection

- [ja4-guard](https://github.com/North-web-dev/ja4-guard) — **Go**. Zero-dependency library + reverse proxy that captures the raw inbound ClientHello, computes JA4/JA4H, and enforces allow/deny/known-browser policy at the edge — the defensive counterpart to client impersonation.
- [fingerproxy](https://github.com/0x4D31/fingerproxy) — **Go**. A reverse proxy that computes JA3/JA4 and Akamai HTTP/2 fingerprints for each incoming connection and forwards them as headers to your backend — a practical way to fingerprint clients at the edge.
- [FoxIO-LLC/ja4 — detector plugins](https://github.com/FoxIO-LLC/ja4/tree/main/rust) — the JA4+ repo ships generators/analyzers for Zeek, Wireshark, Suricata, and more, usable for passive detection of forged fingerprints.
- [dreadl0ck/ja3](https://github.com/dreadl0ck/ja3) — **Go**. A gopacket-based library and CLI for extracting JA3/JA3S fingerprints from live traffic or PCAPs — useful on the detection side.
- [trisul JA3 scripts](https://github.com/trisulnsm/trisul-scripts/tree/master/lua/frontend_scripts/reassembly/ja3) — Lua scripts that add JA3 extraction to the Trisul network monitoring platform.
- [Cloudflare — JA3/JA4 fingerprinting docs](https://developers.cloudflare.com/bots/concepts/ja3-ja4-fingerprint/) — vendor documentation on how a major WAF/anti-bot platform uses TLS fingerprints to score traffic. Useful for understanding what you're up against.

## Reflectors & test endpoints

- [tls.peet.ws/api/all](https://tls.peet.ws/api/all) — a JSON endpoint that echoes the full fingerprint of the requesting client (JA3, JA4, peetprint, HTTP/2). Ideal for scripted verification in CI.
- [check.ja3.zone](https://check.ja3.zone/) — a minimal endpoint that returns the caller's JA3 fingerprint.

## Articles & research

- [Cloudflare — JA3/JA4 fingerprint concepts](https://developers.cloudflare.com/bots/concepts/ja3-ja4-fingerprint/) — concise explainer of how TLS fingerprints feed bot detection.
- [lwthiker — TLS fingerprinting in the modern web](https://lwthiker.com/networks/2022/06/17/tls-fingerprinting.html) — the article that accompanied `curl-impersonate`, explaining how and why TLS fingerprinting works.
- [HTTP Toolkit — How to bypass Node.js TLS fingerprinting](https://httptoolkit.com/blog/tls-fingerprinting-node-js/) — a hands-on walk-through of detecting and altering a client's TLS fingerprint.
- [Scrapfly — How to avoid web scraping blocking: TLS](https://scrapfly.io/blog/how-to-avoid-web-scraping-blocking-tls/) — practical guide to TLS-based blocking and how impersonation defeats it.

## Related awesome lists

- [awesome](https://github.com/sindresorhus/awesome) — the root of the awesome-list universe.
- [awesome-osint](https://github.com/jivoi/awesome-osint) — open-source intelligence resources, including network reconnaissance.
- [awesome-ctf](https://github.com/apsdehal/awesome-ctf) — CTF tooling, much of it overlapping with network and TLS analysis.

## Contributing

Contributions are welcome. Please read [CONTRIBUTING.md](CONTRIBUTING.md) first — every entry must be real, have a resolving link, and carry an honest one-line description.

## License

[![CC0](https://licensebuttons.net/p/zero/1.0/88x31.png)](LICENSE)

To the extent possible under law, the contributors have waived all copyright and related or neighboring rights to this work. See [LICENSE](LICENSE) (CC0 1.0 Universal).

## Support

If this list is useful to you, a ⭐ star is the best thanks. Tips are welcome but never expected:

| Coin | Address |
|---|---|
| **BTC** | `bc1q0sfghv7yfl594gjurhyhlhdgm0dtwh4dkjha36` |
| **ETH / USDT-ERC20 / BNB-BEP20** | `0xB827b5a8B6c5111946c5D872933b2C23484E87dE` |
| **SOL** | `3ch5aHrPp3CierhiNXmqmtFH2dMHh6Gfv7a97KtLMMCh` |
| **USDT-TRC20 / TRX** | `TUAyz7TjgJdFVD2oFHWUUTNrzzAnBRgySG` |
| **TON** | `UQAJmYVuR3hh02etjaPKJF-h6jcpdSnq9ZIFvI64NEskS6tx` |
