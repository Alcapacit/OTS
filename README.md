# Oblivious Transport Security (OTS)

**Internet-Draft:** `draft-alcapacit-asots-ots-00, draft-alcapacit-asots-ots-01`  
**Status:** Experimental  
**Author:** Mohamed Daud Mohamed - ZatQasim ([Alcapacit Services](https://github.com/Alcapacit))

## Overview

**Oblivious Transport Security (OTS)** is a connection-oriented, encrypted, and authenticated transport protocol designed to eliminate network metadata leaks without relying on third-party proxies, relays, or peer-to-peer networks.

Unlike conventional transport security protocols (such as TLS 1.3), OTS prevents passive network observers from inspecting plaintext Server Name Indication (SNI) fields, fingerprinting client implementations, performing packet-size or timing analysis, or tracking endpoints across network-layer address changes.

## Key Features

* **Zero Handshake Metadata & Fingerprinting:** Encrypts SNI inside a 1-RTT handshake using pre-distributed server keys (e.g., via DNS SVCB/HTTPS records) and enforces a single mandatory cryptographic suite with zero feature or version negotiation.
  
* **Fixed Wire Format (1280 Octets):** Every datagram matches the IPv6 minimum link MTU (1280 octets), preventing fragmentation and eliminating packet-length side channels.
  
* **Isochronous Traffic Shaping:** Endpoints transmit at a deterministic **Tick Interval ($T$)**, injecting authenticated, encrypted **Cover Frames** when no application data is queued to maintain a constant bitrate.

* **Cryptographic Path Binding & Address Rotation:** Sessions are bound to a 64-bit **Connection ID (CID)** rather than IP addresses. At each **Rotation Epoch** (default: 256 datagrams or 10 seconds), initiators atomically rotate both their ephemeral source IP address and active CID.

## Cryptographic Primitives

OTS mandates a single fixed suite:
* **Key Exchange:** X25519 (RFC 7748)
* **Key Derivation:** HKDF-SHA256 (RFC 5869)
* **Authenticated Encryption:** ChaCha20-Poly1305 (RFC 8439)
* **Server Authentication:** Ed25519 (RFC 8032)

## Wire Layout (1280 Octets)

| Offset | Field | Size | Description |
| :--- | :--- | :--- | :--- |
| `0` | **Version** | 1 B | Protocol version (`0x01`) |
| `1` | **Type** | 1 B | `0x01` (Handshake) or `0x02` (Protected Data) |
| `2–3` | **Reserved** | 2 B | Must be `0x0000` |
| `4–11` | **Connection ID** | 8 B | 64-bit cryptographically random session identifier |
| `12–19` | **Packet Number** | 8 B | Monotonically increasing counter (96-bit AEAD nonce) |
| `20–1263`| **Encrypted Payload**| 1244 B| Ciphertext containing Frame Type (1 B), Length (2 B), Data/Padding (1241 B) |
| `1264–1279`| **Auth Tag** | 16 B | Poly1305 authentication tag |

## License

This specification is subject to the IETF Trust Legal Provisions (`trust200902`).
