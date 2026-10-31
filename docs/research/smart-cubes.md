# Smart cubes for Rouxtine: which cube to buy, which protocols to build, and how

Researched 2026-10-28 (shop listings, Android docs and parts of the code re-read 2026-10-29) to choose the first test cube for Rouxtine and the order in which to implement smart-cube Bluetooth (BLE) protocols. `[n]` refers to the numbered sources at the bottom. A line marked UNVERIFIED or "inference" was not confirmed from a primary source or on hardware.

Code permalinks are shallow clones, read only, never built. A reference like `SWB:src/x.ts#L1-L9` means that base URL plus the path and line anchor.
- **SWB** = `https://github.com/poliva/smartcube-web-bluetooth/blob/2077416d6c828ec926dee424cf1bbfe4c0f25a47/` (smartcube-web-bluetooth v4.1.0, MIT)
- **GWB** = `https://github.com/afedotov/gan-web-bluetooth/blob/65173e2cdd0fa38ef384f237f6da8d76c177ed1e/` (gan-web-bluetooth v3.0.2, MIT)
- **CST** = `https://github.com/cs0x7f/cstimer/blob/5f8e46cb78da6a487fc01ba913a6ce6fd101e28d/` (csTimer, GPL-3.0)
- **CJS** = `https://github.com/cubing/cubing.js/blob/615719165e79c85453fcf5a69dee885a3124d57a/` (cubing.js, MPL-2.0 OR GPL-3.0-or-later)
- **OG** = `https://github.com/turnerglenn-cmyk/open-gan/blob/27520a574f53270716f11d0386e2597b43a553a6/` (open-gan, GPL-3.0-or-later, Kotlin)
- **QYS** = `https://codeberg.org/Flying-Toast/qiyi_smartcube_protocol/src/commit/0c1d02eaac7097d246709fb211da58478b351863/` (QiYi protocol spec, MIT)
- **GCP** = `https://github.com/oddpetersson/gocube-protocol/blob/696e25e6ccb621493cb987c28cbb0877656fe425/` (GoCube protocol spec, MIT)
- **HKP** = `https://github.com/heykube/heykube_python/blob/6fab8c0bd03a6ca1ee817d4ae0cb7361d83c02ad/` (HEYKUBE Python library, MIT)
- **KBL** = `https://github.com/JuulLabs/kable/blob/ad88767b67ae60990b63cd36897b38e7d69aef61/` (Kable, Apache-2.0)
- **AOSP** = `https://android.googlesource.com/platform/packages/modules/Bluetooth/+/refs/heads/main/android/app/` (Android Bluetooth stack, main branch)

Capture files are `SWB:captures/fixture_<name>.json`. Shop counts and prices are from 2026-10-28 unless a line says otherwise. Prices move often; treat them as a snapshot.

## Summary

- **Buy a QiYi AI 3x3 first** (about EUR 20–23 or $23).
  - It is the cheapest current smart 3x3: EUR 19.90 at atoutcubes (black, in stock), EUR 22.99 at cuboss (in stock), $22.95–22.99 in the US [33][32][30][31].
  - Its protocol is the best documented: an MIT byte-level spec plus SWB (MIT), csTimer (GPL) and cubing.js drivers, and two SWB replay captures [QYS][SWB][CST][CJS].
  - Every move packet carries cube-clock timestamps and an 11-move history with the full cube state, so splits can use the cube clock and lost moves are recovered without a request (`SWB:src/smartcube/protocols/qiyi.ts#L14-L106`).
  - It exercises the Android parts most likely to break: 48- and 96-byte notifications (bigger than the default MTU), ACK writes, and a documented "solved state never arrives during fast slice moves" glitch that Roux LSE will trigger [QYS:README.md#L137-L145].
  - The owner has no smart cube yet and planned to buy a cheap one for testing [79].
- **Second cube: a GAN i4 (MagLev, EUR 64.99–69.99 in the EU)**, after GAN Gen4 passes SWB's replay captures. It is GAN's current protocol, confirmed by a capture [10][11][SWB:captures]. The cheaper **i4 Leap (EUR 53.90–54.99)** probably speaks the same protocol, but no capture or driver names it.
- **If only one cube will ever be bought, buy the GAN i4 instead.** GAN makes about half of the smart cubes on US shelves (16 of 29 at SpeedCubeShop, 18 of 36 at TheCubicle), so GAN support reaches the most users [1][2].
- **Protocol implementation order:**
  1. **Protocol-agnostic core + QiYi** (QiYi AI, X-Man Tornado V4 AI), tested on the real cube.
  2. **GAN Gen4** (GAN16 ui MAX, i4, 12 ui Maglev), then **Gen2** (older GAN12 ui, probably i Carry E) with **MoYu AI 2023** as a Gen2 key variant. **Gen3** (i Carry 2 only) last: buggy firmware and no capture.
  3. **MoYu32** (WeiLong V10 AI confirmed; V11 AI, Super AoLong AI and AoLong V5 AI probable).
  4. **GoCube / Rubik's Connected**: no encryption, but no cube timestamps, so splits are less precise.
  5. **On demand only:** Giiker/Mi, GAN Gen1, MoYu MHC, HEYKUBE. TokTok, MoYu Super WeiLong V2 AI and unconfirmed GAN models once someone sends a BLE capture.
- **Android makes connection easier than the web.** The GAN, MoYu32 and QiYi keys or handshakes need the cube's MAC address. Web apps often have to ask the user to type it. Android reads it from the scan (`BluetoothDevice.getAddress()`, Kable `AndroidPeripheral.address`) [68][KBL:README.md#L276][60].
- **Roux-specific work no library does for us:**
  - No protocol reports slice moves; an M turn arrives as an L + R' pair. Rouxtine must merge opposite-face pairs into M/E/S and track centres itself [QYS:README.md#L172-L188][51].
  - GAN Gen4 can pack two turns into one notification, "common on slice moves". open-gan's Kotlin driver still reads only the first one [SWB:src/gan-cube-protocol.ts#L976-L1010][OG:core/src/main/kotlin/app/opengan/core/gan/GanProtocolDrivers.kt#L308-L314].
  - The QiYi "treat needs-ACK as solved" workaround is in the spec, but neither SWB nor csTimer implements it [QYS][SWB:src/smartcube/protocols/qiyi.ts#L277-L281][CST:src/js/hardware/qiyicube.js#L179-L181].
- **Timing:** use cube timestamps plus a linear fit of cube clock to host clock (csTimer's method). On SWB's captures, host-only intervals are off by a median of 6–26 ms and up to 81 ms per move [SWB:captures]. **QiYi ticks are 0.625 ms**: divide by 1.6. They are not 1.6 ms units, despite the spec's wording [QYS:README.md#L42][SWB:src/smartcube/protocols/qiyi.ts#L305].
- **Code to reuse:** port **SWB** (MIT, TypeScript, active, covers every known protocol in this note except HEYKUBE) to Kotlin, and replay its 17 MIT capture files in JVM unit tests [SWB]. Use **Kable 0.45.0** (Apache-2.0, minSdk 21) for BLE [66]. All candidate sources are GPL-3.0-compatible, which matters because Rouxtine is GPL-3.0-only [77].

## 1. Recommendation

1. **Buy one QiYi AI 3x3** (Magnetic, not UV): atoutcubes EUR 19.90 or cuboss EUR 22.99 in the EU, SpeedCubeShop or TheCubicle about $23 in the US [33][32][30][31]. Treat it as the test device, not the main target: it has no gyroscope (§3.1).
2. **Build the protocol-agnostic core first** (§7), then the QiYi driver from the MIT spec and SWB. Test it on the real cube on the owner's Galaxy S25 (Android 16, One UI 8) [79].
3. **Port GAN Gen4, then Gen2,** from SWB. Check them against SWB's five GAN replay captures before buying any GAN [SWB:captures].
4. **Buy a GAN i4 (MagLev)** as the second cube, to test Gen4 and the gyroscope on hardware. Pick the i4 Leap only if saving EUR 10–15 matters more than certainty about its protocol [10][11].
5. **Add MoYu32 and GoCube from captures**, then wait for user demand or captures for the rest.
6. **Ship a "record BLE capture" export** in SWB's fixture format, so users with untested cubes (TokTok, MoYu Super WeiLong V2 AI, GAN i Carry 4, GAN12 ui SP) can send one in (§7).
7. **Use Kable 0.45.0,** request MTU 517 and high connection priority during timed solves, and timestamp moves with the cube clock (§4, §5).

**Why not GAN first?** GAN Gen4 is the bigger market and can be developed against two captures without hardware. But a GAN i4 costs about three times as much as a QiYi AI. The QiYi AI also tests the Android risks that captures cannot test: large notifications, writes and ACKs on a live link, and the slice glitch. GAN Gen4 shares its crypto with Gen2 and Gen3, so once the core works, GAN is mostly porting [SWB:src/gan-cube-definitions.ts#L23-L71].

## 2. Market: who sells what

**Shelf share (US, 2026-10-28; listings, not sales).**

| Shop | Smart puzzles listed | GAN | MoYu | QiYi protocol (QiYi AI, X-Man Tornado V4 AI) | Other |
|---|---|---|---|---|---|
| SpeedCubeShop [1] | 29 | 16 | 9 | 4 | – |
| TheCubicle [2] | 36 | 18 | 12 | 5 | 1 TokTok |

The counts include a few 2x2s and several editions of the same cube. GAN is 50–55% of listings and QiYi-protocol cubes about 14% at both shops [1][2].

**EU shops.** cuboss.com and atoutcubes.com (FR) both list smart cubes with prices and stock. atoutcubes has the broader range, including the GAN16 ui MAX, the i4 and i4 Leap, the i Carry 4, the GAN12 ui SP and ui MagLev, the Tornado V4 AI and the QiYi AI [11][32]. kubekings.com (Cloudflare 403), speedcubes.be (proxy CONNECT 502) and speedcube.de (522) could not be read.

**Discontinued or legacy.**
- Giiker / Xiaomi: GiiKER's store marks the SUPERCUBE i3S ($39.99) unavailable, and TheCubicle tags the Xiaomi Giiker cube discontinued. Neither SpeedCubeShop nor cuboss stocks them [37][38].
- GoCube and Rubik's Connected: Particula still sells Rubik's Connected ($49.95), the GoCube Edge Full Pack ($79.95) and the GoCube 2x2 ($49.95) on particula-tech.com (getgocube.com redirects there). The two big US shops no longer stock them [36].
- HEYKUBE: TheCubicle says "discontinued indefinitely" ($99.99), but the maker's site shows $69.95 "IN STOCK" with a © 2022 footer [39][40].
- GAN12 ui FreePlay and Mini ui FreePlay: TheCubicle keeps sold-out pages. The GAN14 ui FreePlay was not found at SpeedCubeShop, TheCubicle or gancube.com [20][3].
- Monster Go AI (GAN's budget line): atoutcubes files it under "no longer distributed" [43].

**2x2s share the shelves and some drivers.** The GAN 251 UI 2x2 ($71.96–79.99, EUR 79.99 at cuboss) and the MoYu WeiPo V5 AI 2x2 ($75.95) connect through the Gen4 and MoYu32 drivers [1][44][SWB:README.md#L34-L48]. A Roux-only 3x3 app must detect and reject them. SWB and csTimer tell them apart by name: `GAN251ui`/`GANic251` and `WCU_MY22` [SWB:src/gan-cube-definitions.ts#L53-L56][CST:src/js/hardware/gancube.js#L362][SWB:src/smartcube/attachment/mac-candidates.ts#L58-L67]. Note that `GANic` alone is not enough, because the GANicXXX 3x3 capture uses the same prefix (§3.1).

## 3. Model × protocol × capability × implementation × price

"Protocol evidence" says how the model-to-protocol link is known. "Captures" counts SWB replay files that contain that model's traffic. US prices span GAN, SpeedCubeShop and TheCubicle; EU prices are cuboss.com and atoutcubes.com.

### 3.1 Current 3x3 models

| Model (listed since) | Protocol | Protocol evidence | Gyro | Open drivers | Captures | US price | EU price | Notes |
|---|---|---|---|---|---|---|---|---|
| **GAN16 ui Maglev MAX** (Jun–Jul 2026) | GAN Gen4 | Capture `GAN16ui_FF89` decodes as Gen4; HARDWARE reports `GAN16ui`, productDate 2026-01-09, gyro [49][SWB:captures] | Yes | SWB; OG (GPL) | 1 (8 moves) | $125.96 (sale) – $139.99 [4][5][6] | EUR 139.90, Minions edition [7]; standard edition: none found | GAN's "current top-tier flagship smart cube" [4]. Battery "45h (15h standalone + 30h via GanPod)" [4][5] |
| **GAN i4** (Feb–Mar 2026) | GAN Gen4 | Capture `GANi4_A26E`, HARDWARE `GANi4`, productDate 2025-12-11 [SWB:captures] | Yes | SWB; OG | 1 (30 moves) | $62.96–69.99 [8][1][2] | EUR 64.99 (MagLev Frosted) / 69.99 (UV) [10]; EUR 69.90 [11] | "30-hr Burst. Gyroscope on." and "200-hr Marathon. Gyroscope off. Recording disabled" [8] |
| **GAN i4 Leap** (Mar 2026) | Probably Gen4 | UNVERIFIED: "Leap" appears in no driver, capture or commit | Not checked | – | 0 | $49.46–54.99 [9][1] | EUR 54.99 [10]; EUR 53.90 [11] | cuboss sells it as the "Leap Frosted" variant of its i4 product [10] |
| **GAN12 ui Maglev** (2024; "PowerPod, 2026 Edition" at TheCubicle) | Gen4 | gan-web-bluetooth's 2024 list [GWB:README.md#L91-L104]; no capture names a Maglev | Not checked | SWB; OG | 0 | $105.99–138.99 [3][1][2] | EUR 125.90 (PowerPod V2), 139.90 (10th-anniversary box) [11][19] | Whether the 2026 edition changed protocol is UNVERIFIED |
| **GAN12 ui SP** (2025) | UNVERIFIED | No driver or capture names it | Not checked | – | 0 | $89.95–99.99 [17][2] | EUR 89.90 / 99.90 [11] | "6 angle sensors ... help reduce missed move detection" [17] |
| **GAN356 i3 V2** | UNVERIFIED | No driver or capture names it | Yes, per shop copy [18] | – | 0 | $59.99, TheCubicle only [18] | – | Rechargeable ("15 hours"), "Smart Core 3.0" [18] |
| **GAN i Carry 4** (Jan 2026) | UNVERIFIED | No driver or capture names it | No [12] | – | 0 | $31.46–39.99 [13][1][2] | EUR 31.90 [11] | Button cell, about 150 h, not rechargeable [12] |
| **GAN356 i Carry E / E V2** | UNVERIFIED; Gen2 likely | Unlabelled Gen2 capture `GANicXXX` reports hardware name `GANicEqV`, no gyro: fits the i Carry E, but is not labelled [SWB:captures] | No [12][14] | SWB would connect if it exposes a Gen2–4 service [SWB:src/smartcube/protocols/gan.ts#L224-L311] | 1? | $25.99–29.95 [3][14][2] | EUR 25.99 [15] | Button cell, about 150 h [12] |
| **GAN356 i Carry 2** | GAN Gen3 (the only Gen3 model) | [GWB:README.md#L100-L101] | No [12]; the Gen3 driver has no gyro branch [SWB:src/gan-cube-protocol.ts#L773-L792] | SWB; OG | 0 | $35.99–48.99 (GAN), $44.99 (TheCubicle, 10th-anniversary UV); sold out at SpeedCubeShop [3][2][16] | – | "iCarry2 has very buggy firmware ... skipping moves so bad" (Andy Fedotov) [45]. Button cell, about 700 h [12] |
| **MoYu WeiLong V10 AI** (2024) | MoYu32 | csTimer commit for "bound WeiLong V10 Ai" in the MoYu32 path [46]; csTimer issue #518 shows a V10 AI as `WCU_MY32_431D` [47]; captures `WCU_MY32_A388` | Yes (captures contain gyro); the cheap no-gyro unit needs an init burst [48] | SWB; CST | 2 | $53.06–80.96; V10 AI Lite $32.99 [1][21] | EUR 57.99 [22]; EUR 54.90 / 70.99 (on order) [11] | Name-to-model link rests on [46][47] |
| **MoYu WeiLong V11 AI** (Jul 2025) | Probably MoYu32 | UNVERIFIED. Drivers try MoYu32 for any `WCU_` name [SWB:src/smartcube/protocols/moyu32.ts#L533-L540] | Gyro on the MagLev Gyroscope edition [27] | (SWB, CST by prefix) | 0 | $114.95–125.99 [1][23] | EUR 115.90, last items [27] | Pairs with the WCU CUBE app [23]. A "Guided Recovery Edition" was added 2026-09-01 [1] |
| **MoYu Super AoLong AI** (Nov 2025) | Probably MoYu32 | UNVERIFIED, as above | "Gyroscope magnetic core" [24] | (by prefix) | 0 | $121.46–159.99 [24][2] | EUR 134.90 / 149.90, out of stock [11] | "up to 30 hours" [24] |
| **MoYu AoLong V5 AI** (2026-10-23) | Probably MoYu32 | UNVERIFIED, as above | MagLev variant only [25] | (by prefix) | 0 | $42.99–80.95 [26][25] | None found; cuboss lists only the non-AI V5 [29] | Brand new |
| **MoYu Super WeiLong V2 AI** | Unknown | No driver names it | Not checked | – | 0 | Not at SpeedCubeShop or TheCubicle [1][2] | EUR 139.90; 169.90 (18th Anniversary), in stock [28] | See open questions |
| *(unidentified MoYu 3x3 `WCU_MY33`)* | MoYu32 | Capture decodes with the MoYu32 key; no gyro packets [SWB:captures] | No packets seen | SWB | 1 | – | – | Product unknown |
| **QiYi AI 3x3** ("QiYi Smart Cube") | QiYi | Capture `QY-QYSC-S-A0E6`; advertises `QY-QYSC-S-XXXX` [SWB:captures] | No [SWB:src/smartcube/protocols/qiyi.ts#L183-L205] | SWB; CST; CJS | 1 | $22.95–22.99; UV $26.95–26.99 [30][31] | EUR 19.90 (black, in stock) [33]; EUR 22.99 (in stock) [32]; EUR 22.90 UV, out of stock [33] | Cheapest current smart 3x3 |
| **X-Man Tornado V4 AI** | QiYi | Capture `XMD-TornadoV4-i-034C` with 214 gyro packets [SWB:captures] | Yes (0xCC 0x10 quaternions) [SWB:src/smartcube/protocols/qiyi.ts#L183-L215] | SWB; CST (no gyro decoding); not CJS [CST:src/js/hardware/qiyicube.js#L16-L22][CJS:src/cubing/bluetooth/smart-puzzle/qiyi.ts#L443-L446] | 1 | $59.99–99.99 [34][2] | EUR 69.90 [11]; EUR 79.99 [35] | SpeedCubeShop lists Roux among its app tutorials [34] |
| **TokTok Cube 3x3** (Smart Cube Labs, eX-Mars app) | Unknown | No driver in SWB, csTimer, cubing.js or open-gan | Not checked | – | 0 | $74.99 (black, sold out), $98.99 (white) [41] | – | Its app's last version is from 2024-02-19 [42] |

### 3.2 Legacy and niche models

| Model | Protocol | Gyro | Open drivers | Captures | Availability |
|---|---|---|---|---|---|
| GAN12 ui, GAN12 ui FreePlay, Mini ui FreePlay, i Carry S, i Carry, i 3, Monster Go 3Ai | GAN Gen2 [GWB:README.md#L91-L99] | Per HARDWARE bit; both GAN12 ui captures report gyro | SWB; OG | `GAN12ui--B3C`, `GAN12uiFp-753` | Sold out or second-hand [20] |
| GAN14 ui FreePlay | GAN Gen4 [GWB:README.md#L102-L104] | Yes | SWB; OG | 0 | Not found in shops |
| MoYu AI 2023 (`AiCube`) | GAN Gen2 framing, own key | No (turned off by name) [SWB:src/smartcube/protocols/gan.ts#L115-L123] | SWB; OG | 0 | Not checked |
| Older MoYu AI (`MHC`) | MoYu MHC [SWB:src/smartcube/protocols/moyu-mhc.ts#L15-L35] | With characteristic 0x1004 | SWB; CST | 0 | Not checked |
| GAN356 i (v1) | GAN Gen1 [SWB:src/gan-cube-definitions.ts#L1-L21] | Yes | SWB; CJS | 0 | Second-hand |
| GoCube (classic, Edge), GoCube X, Rubik's Connected | GoCube | GoCube and Edge yes; GoCube X and Rubik's Connected no [SWB:src/smartcube/protocols/gocube.ts#L19-L53] | SWB; CST; CJS | `GoCube`, `Rubiks_Connected` | Particula store only [36] |
| Giiker i3 / i3S, Xiaomi Mi Smart | Giiker [SWB:src/smartcube/protocols/giiker.ts#L11-L68] | No | SWB; CST; CJS | `Giiker_i3SE` | Second-hand [37][38] |
| HEYKUBE | HEYKUBE (service `b46a791a-8273-4fc1-9e67-94d3dc2aac1c`) [HKP:heykube/heykube_btle.py#L22-L57] | Not checked | CJS; HKP (Python) | 0 | Unclear [39][40] |

Edge orientation in SWB depends only on the advertised name starting with `GoCube` (but not `GoCubeX`). It rests on one commit marked "experimental" [SWB:src/smartcube/protocols/gocube.ts#L19-L53].

### 3.3 Protocol capabilities

| Protocol | GATT service | Crypto | Needs MAC? | Cube timestamp | Lost-move recovery | Gyro | App must... | Open drivers | Spec |
|---|---|---|---|---|---|---|---|---|---|
| **QiYi** | `fff0`, almost all traffic on `fff6` | AES-128-ECB, fixed key | Yes, reversed in the App Hello | 32-bit, **0.625 ms ticks** (÷ 1.6) | 11 history slots plus full state in every State Change | Tornado V4 only | ACK most messages; ACK State Change when byte 91 is set | SWB, CST, CJS | MIT spec [QYS] |
| **GAN Gen4** | `00000010-0000-fff7-fff6-fff5fff4fff0` | AES-128-CBC on first and last 16-byte block; key and IV salted by the reversed MAC | Yes | 32-bit | `MOVE_HISTORY` request (0xD1 0x04) | Yes (0xEC) | Request facelets first; read every MOVE chunk | SWB, OG | Code only |
| **GAN Gen3** | `8653000a-43e6-47b7-9cb0-5fc21d4ae340` | Same as Gen4 | Yes | 32-bit | `MOVE_HISTORY` (0x68 0x03) | No | Work around i Carry 2 firmware bugs | SWB, OG | Code only |
| **GAN Gen2** | `6e400001-b5a3-f393-e0a9-e50e24dc4179` | Same as Gen4 (MoYu AI 2023 has its own key) | Yes | 16-bit per-move deltas, summed by the driver | Last 7 moves per packet; no history request | Yes (event 0x01; support bit in HARDWARE) | Request facelets first | SWB, OG | Code only |
| **GAN Gen1** | `fff0` + Device Information | AES, key from LZ-compressed blobs, salted with System ID 0x2A23 | No (System ID) | 16-bit raw stamps | Polled every 30 ms | Yes | Poll | SWB, CJS | cubing.js doc [CJS:src/cubing/bluetooth/docs/gan356i.md] |
| **MoYu32** | `0783b03e-7735-b5a0-1760-a305d2795cb0` | AES-128, IV XOR over first and last 16 bytes, key salted by reversed MAC | Yes; company ID and 4th octet vary | 16-bit offsets, accumulated; re-anchored to host if drift > 2 s | Last 5 moves (3x3) | Yes (opcode 171, after enable 172) | Request facelets (163) at init; poll battery; init burst on cheap units | SWB, CST | Code only |
| **MoYu MHC** | `0x1000` | Not checked | Not checked | 32-bit, 1/65536 s | Not checked | With 0x1004 | Not checked | SWB, CST | Code only |
| **GoCube** | `6e400001-b5a3-f393-e0a9-e50e24dcca9e` | None; sum checksum | No | **None** | None | GoCube, Edge | Enable orientation (0x38) | SWB, CST, CJS | MIT spec [GCP] |
| **Giiker** | `0xaadb` / notify `0xaadc` | Static 36-byte table when byte 18 = 0xA7 | No | **None** | None | No | Poll battery (0xb5 on `aaac`) | SWB, CST, CJS | Code only |

Sources for the table: QiYi [QYS:README.md#L9-L48][QYS:README.md#L84-L188][SWB:src/smartcube/protocols/qiyi.ts#L14-L120]; GAN Gen2–4 [SWB:src/gan-cube-definitions.ts#L23-L71][SWB:src/gan-mac-salt.ts#L1-L17][SWB:src/gan-cube-encrypter.ts#L22-L76][SWB:src/gan-cube-protocol.ts#L383-L443][SWB:src/gan-cube-protocol.ts#L561-L660][SWB:src/gan-cube-protocol.ts#L773-L792][SWB:src/gan-cube-protocol.ts#L875-L1010]; Gen1 [SWB:src/gan-gen1.ts#L31-L50][SWB:src/gan-gen1.ts#L260-L323]; MoYu32 [SWB:src/smartcube/protocols/moyu32.ts#L16-L117][SWB:src/smartcube/protocols/moyu32.ts#L280-L360][SWB:src/smartcube/protocols/moyu32.ts#L420-L436][SWB:src/smartcube/protocols/moyu32-move-history.ts#L8-L24]; MHC [SWB:src/smartcube/protocols/moyu-mhc.ts#L15-L20][SWB:src/smartcube/protocols/moyu-mhc.ts#L123-L181][SWB:src/smartcube/protocols/moyu-mhc.ts#L222-L262]; GoCube [GCP:README.md][SWB:src/smartcube/protocols/gocube.ts#L11-L53][SWB:src/smartcube/protocols/gocube.ts#L127-L143]; Giiker [SWB:src/smartcube/protocols/giiker.ts#L11-L68][SWB:src/smartcube/protocols/giiker.ts#L105-L115][SWB:src/smartcube/protocols/giiker.ts#L239-L262].

### 3.4 Protocol details that affect a port

**GAN Gen2–4 (shared crypto).**
- Each cube's key and IV are a fixed base pair. The first 6 bytes are salted with the cube's MAC in reverse order: `key[i] = (base[i] + mac_rev[i]) % 255`. The modulus really is 255 [SWB:src/gan-cube-encrypter.ts#L29-L35].
- There are three base pairs: GAN Gen2–4, MoYu AI 2023, and a separate pair for the GAN 251 UI 2x2 on Gen4 framing [SWB:src/gan-cube-definitions.ts#L44-L71].
- The generation is picked from the GATT service, after a name match on `GAN`, `gan`, `MG` or `AiCube`. SWB tries Gen2, then Gen3, then Gen4 [SWB:src/smartcube/protocols/gan.ts#L224-L311]. A new model with a new base key, as on the GAN 251 UI, would still fail.
- Without the right MAC nothing decrypts. A verifier decrypted all GAN captures with the salted key; with the unsalted key, only 4–5 of 161–325 Gen4 frames look plausible.
- The drivers accept moves only after the first FACELETS event, so a session must request facelets first. Gen3 and Gen4 also send FACELETS on their own, so the request mainly avoids a startup delay [SWB:src/gan-cube-protocol.ts#L412-L416].

**GAN Gen2.** A move packet carries an 8-bit serial and the last 7 moves, each with a 16-bit elapsed time in ms. The driver sums these deltas into its own "cube timestamp". A delta of 0 means 16-bit overflow and is replaced with host elapsed time. More than 7 moves between packets are lost silently, because Gen2 has no history request [SWB:src/gan-cube-protocol.ts#L412-L443]. The gyroscope arrives as its own event type 0x01; only the gyro-support bit comes from the HARDWARE (0x05) response [SWB:src/gan-cube-protocol.ts#L383-L411][SWB:src/gan-cube-protocol.ts#L486-L507].

**GAN Gen3 and Gen4.**
- Moves carry a 32-bit cube timestamp and a 16-bit serial. The driver does gap arithmetic mod 256, so in practice the serial wraps at 8 bits [SWB:src/gan-cube-protocol.ts#L876-L963].
- A serial gap, or a FACELETS event whose serial is ahead of the last move (checked after 500 ms idle), triggers a MOVE_HISTORY request. Recovered moves have no cube or host timestamp; the linear fit later places them 50 ms from their neighbours [SWB:src/gan-cube-protocol.ts#L561-L660][SWB:src/gan-cube-protocol.ts#L696-L736][SWB:src/utils.ts#L48-L78].
- **Gen4 batches moves.** One notification may hold several 72-bit MOVE chunks. SWB's comment says reading only the first one "drops later face turns (common on slice moves) until MOVE_HISTORY catches up". SWB fixed this in March 2026 [SWB:src/gan-cube-protocol.ts#L976-L1010][50]. Notifications are 20 bytes, so at most two chunks fit. Neither Gen4 capture contains a batched notification, so this path has no replay test.
- **Gen3 (i Carry 2) quirks.** SWB caps the history window at the serial 255→0 edge, because moves past it are "spoofed with D". It also skips serial 0 because of a "facelets state event at 255 move counter" bug [SWB:src/gan-cube-protocol.ts#L561-L650]. Gen3 has no gyro decoding, and its HARDWARE event hard-codes `gyroSupported: false` [SWB:src/gan-cube-protocol.ts#L773-L792].
- On Gen4, `gyroSupported` in HARDWARE means "a 0xEC packet has been seen" [SWB:src/gan-cube-protocol.ts#L822-L871].

**GAN Gen1.** Polled every 30 ms, with facelets every 50 ticks. The key comes from one of six LZ blobs chosen by firmware version, salted with the Device Information System ID (0x2A23), not the MAC. It has no hardware-info or reset command [SWB:src/gan-gen1.ts#L31-L50][SWB:src/gan-gen1.ts#L260-L323][SWB:src/smartcube/protocols/gan.ts#L26-L40].

**MoYu32.**
- Read characteristic `...cb1`, write `...cb2`. Opcode 165 move packets carry the last 5 moves (3x3), 16-bit time offsets and an 8-bit counter. If more were missed, SWB only logs "lost move events". Other opcodes: 161 hardware, 163 facelets, 164 battery (SWB polls every 60 s), 171 gyro after enable payload 172 [SWB:src/smartcube/protocols/moyu32.ts#L16-L117][SWB:src/smartcube/protocols/moyu32.ts#L280-L360][SWB:src/smartcube/protocols/moyu32.ts#L420-L436].
- The driver drops moves until a 163 facelets reply sets its counter, so it requests 163 during init [SWB:src/smartcube/protocols/moyu32.ts#L283-L287].
- SWB drops a whole 165 packet if any of its five move codes is 12 or more (`invalidMove`). If newer WCU firmware adds codes, a straight port would lose moves silently [SWB:src/smartcube/protocols/moyu32.ts#L296-L302].
- **MAC discovery.** The MAC is the 6 bytes after the company ID in the manufacturer data, reversed. The company ID is the high bytes of the WCU account ID when the cube is bound in the WCU Cube app, and 0x0000 when unbound [CST:src/js/hardware/moyu32cube.js#L140-L198]. csTimer derives `CF:30:16:00:XX:XX` from a `WCU_MY32_XXXX` name. SWB also tries 00, 01 and 02 in the fourth octet, in a different order per name (MY32 00/01/02, MY33 02/01/00, MY22 01/00/02), and checks each guess by decrypting a probe [SWB:src/smartcube/attachment/mac-candidates.ts#L46-L90][SWB:src/smartcube/protocols/moyu32.ts#L474-L530].

**MoYu MHC.** Write 0x1001, read 0x1002, turns 0x1003, gyro 0x1004. Turn records carry face-angle steps with a 32-bit timestamp in 1/65536 s. A move is emitted only when the 9-step angle crosses the 4/5 boundary. Battery and facelets need the v1 characteristics [SWB:src/smartcube/protocols/moyu-mhc.ts#L123-L181][SWB:src/smartcube/protocols/moyu-mhc.ts#L222-L262].

**QiYi.**
- Encryption: AES-128-ECB with a fixed key, zero padding to 16 bytes, and a CRC16-MODBUS checksum. The App Hello must be sent first and must contain the MAC in reverse order [QYS:README.md#L30-L83].
- The cube→app header carries a 4-byte big-endian timestamp. The spec says "units of 1.6 milliseconds! Divide it by 1.6 to convert to milliseconds!" [QYS:README.md#L42]. The two sentences conflict. The code settles it: the spec's own Rust example, SWB and csTimer all divide by 1.6, so one tick is 0.625 ms [QYS:example_app/src/messages.rs#L219][SWB:src/smartcube/protocols/qiyi.ts#L305][CST:src/js/hardware/qiyicube.js#L208]. My fit of host time against SWB's converted cube times gives slopes of 1.009 and 1.000 on the two QiYi captures; 1.6 ms ticks would give about 2.56.
- State Change messages are 94 bytes: full state, the last move, battery, previous moves with timestamps in bytes 36–90 (11 slots in SWB), and a needs-ACK byte at 91 [QYS:README.md#L137-L188][SWB:src/smartcube/protocols/qiyi.ts#L14-L106]. csTimer reads only 9 history moves [CST:src/js/hardware/qiyicube.js#L179-L210].
- ACKs echo bytes 3–7 of the message [QYS:README.md#L84-L106]. A State Change needs one only when byte 91 is set, which normally means "solved". SWB ACKs only then; csTimer ACKs every State Change [SWB:src/smartcube/protocols/qiyi.ts#L277-L281][CST:src/js/hardware/qiyicube.js#L179-L181].
- **The solved glitch.** The spec says that if the cube is solved during fast slice moves (e.g. an H-perm), the solved State Change never arrives. Instead a State Change with needs-ACK set arrives whose state is not solved. The spec's fix: treat any needs-ACK State Change as solved [QYS:README.md#L137-L145]. No open driver does this.
- The spec also documents **Sync State** (set the cube's remembered state), **Sync Confirmation** (opcode 4), and **Request State / Current State** (opcode 5) [QYS:README.md#L190-L296]. SWB handles an incoming opcode 4, but sends neither command: its `reset` capability is false and REQUEST_FACELETS re-sends the Hello [SWB:src/smartcube/protocols/qiyi.ts#L114-L120][SWB:src/smartcube/protocols/qiyi.ts#L327-L375]. Rouxtine can build "reset to solved" and state polling from the spec.
- **Slice report.** SWB issue #2 (Tornado V4, opened 2026-05-04) reports that the second face of a slice move sometimes arrives only after another face is turned. The reporter closed it on 2026-05-12 with no maintainer reply and no fix described [51].

**GoCube / Rubik's Connected.** No encryption, a sum-mod-256 checksum, facelets and battery, no cube timestamps [SWB:src/smartcube/protocols/gocube.ts#L11-L53][SWB:src/smartcube/protocols/gocube.ts#L127-L143]. A community MIT spec documents commands (0x32 battery, 0x33 state, 0x35 reset to solved, 0x37/0x38 orientation off/on), the checksum and orientation messages [GCP:README.md]. Firmware may send a truncated move frame for the second face of an opposite-face pair; SWB maps it to the opposite face with inverted direction [SWB:src/smartcube/protocols/gocube.ts#L223-L240]. That pair is exactly a slice move.

**Giiker.** Each notification has the full state and the latest move, with no timestamps and no gyro. Packets whose byte 18 is 0xA7 are obfuscated with a static 36-byte table. Giiker can report a half turn as one move [SWB:src/smartcube/protocols/giiker.ts#L11-L68][SWB:src/smartcube/protocols/giiker.ts#L105-L115].

### 3.5 What no protocol does: slices, wide moves, centres

- Every 3x3 protocol encodes only the six outer faces (URFDLB). An M turn arrives as an L + R' pair [QYS:README.md#L172-L188][SWB:src/gan-cube-protocol.ts#L419-L424]. The log in SWB issue #2 shows "M (L+R')" with a 15 ms cube-timestamp gap; that line comes from the reporter's own app [51].
- So Rouxtine must merge opposite-face pairs that fall within a short window into M/E/S.
- Inference from the same fact: sensors measure faces against the core, so a wide move such as r should arrive as a single L turn plus an implied rotation. Stage detection should therefore compare states modulo whole-cube rotation, as csTimer's Roux split does with 24 orientations [78 §3]. Test this on hardware.
- SWB's unified `MOVE.direction` is not consistent across drivers: GAN, QiYi, GoCube and MHC use 0 = CW and 1 = CCW, MoYu32 uses 0 and 2, and Giiker uses 2 for a half turn. A port should normalise from the move string [SWB:src/smartcube/protocols/moyu32.ts#L319-L333][SWB:src/smartcube/protocols/giiker.ts#L29-L68].

### 3.6 Notification sizes (from SWB's captures)

My count over all 17 capture files [SWB:captures]:
- GAN Gen2/Gen4, MoYu32 and Giiker notifications are always 20 bytes or less.
- QiYi sends 48-byte Cube Hello and 96-byte State Change notifications, and writes a 32-byte App Hello. The Tornado also sends 16-byte gyro packets.
- GoCube and Rubik's Connected send up to 66 bytes.

QiYi and GoCube therefore exceed the 20-byte payload of the default ATT MTU of 23. Inference, not tested on hardware: on Android, Rouxtine should call `requestMtu(517)` after connecting and before the QiYi App Hello or the GoCube subscription, or those frames may be cut. open-gan never calls `requestMtu` [OG:app/src/main/kotlin/app/opengan/ble/GattConnection.kt].

## 4. Android BLE specifics

**MAC address.**
- The MAC used to salt GAN's key is "the Bluetooth device hardware address", per Andy Fedotov's FAQ. It notes that platform-native apps can read it automatically, while Web Bluetooth cannot [59].
- Android's `BluetoothDevice.getAddress()` "Returns the hardware address of this BluetoothDevice" [68]. Kable exposes it as `AndroidPeripheral.address` [KBL:README.md#L276][KBL:kable-core/src/androidMain/kotlin/AndroidPeripheral.kt#L121-L126].
- csTimer on the web asks the user to type the MAC unless `watchAdvertisements` works, which needs `chrome://flags/#enable-experimental-web-platform-features`. On Linux and ChromeOS it always falls back to typing [CST:src/js/hardware/bluetooth.js#L39-L51][CST:src/lang/en-us.js#L616][60].
- **Recommended order:** read the MAC from the advertised manufacturer data (as SWB, csTimer and open-gan do), fall back to `getAddress()`, then to name-derived candidates, and confirm by decrypting a packet [SWB:src/gan-cube-definitions.ts#L58-L59][CST:src/js/hardware/moyu32cube.js#L186-L190][OG:app/src/main/kotlin/app/opengan/ble/DeviceManager.kt#L85-L95]. open-gan's comment: "Android exposes the real address, unlike Web Bluetooth; prefer the advertised one if present". SWB already validates GAN packets after decryption and probes MoYu32 and QiYi candidates; open-gan does no decrypt check.
- Android's `ScanRecord` returns every manufacturer-data entry. Web Bluetooth must list up to 256 GAN company IDs in advance [SWB:src/gan-cube-definitions.ts#L58-L59]. **Kable's `Advertisement.manufacturerData` property returns only the first entry;** use `manufacturerData(companyId)` or the raw `bytes` [KBL:kable-core/src/androidMain/kotlin/ScanResultAndroidAdvertisement.kt#L58-L89].
- Address-type APIs are recent: `getAddressType()` arrived in API 35, and `getIdentityAddressWithType()` (API 36) also needs `BLUETOOTH_PRIVILEGED`, so normal apps cannot use it [68].
- Capture MACs: MoYu32 `CF:30:16:0X:…`, QiYi `CC:A3`/`CC:A6:00:00:…`, and the last two bytes match the name suffix. GAN Gen4 captures (`0C:3D:5E:…`, `54:6C:50:…`) also end in the name suffix [SWB:captures]. cubing.js documents the same QiYi name-to-MAC rule [CJS:src/cubing/bluetooth/smart-puzzle/qiyi.ts#L189-L205]. Whether these addresses are stable across power cycles is an open question.

**Permissions** (targeting Android 12+, read 2026-10-28) [67]:
- Declare `BLUETOOTH_SCAN` with `android:usesPermissionFlags="neverForLocation"` (the app never derives location) and `BLUETOOTH_CONNECT`. Both are runtime permissions, shown to the user as "Nearby devices".
- Set `android:maxSdkVersion="30"` on the legacy `BLUETOOTH` and `BLUETOOTH_ADMIN` permissions, and on `ACCESS_FINE_LOCATION`, which scanning needs on Android 11 and below.
- The docs warn that `neverForLocation` filters "some BLE beacons" from scan results. That cubes are not affected is an inference.
- On Android 8.0+, `CompanionDeviceManager` can scan for a device without the location permission.
- open-gan's manifest already follows this pattern. Kable declares no permissions, but its optional `com.juul.kable:kable-default-permissions` artifact adds the same set and marks `bluetooth_le` as not required. open-gan marks it required [OG:app/src/main/AndroidManifest.xml][KBL:kable-default-permissions/src/main/AndroidManifest.xml][KBL:README.md#L606-L613].

**Scanning limits.** AOSP's defaults allow 5 scan starts per 30-second window. After that, a non-privileged app gets `SCAN_FAILED_SCANNING_TOO_FREQUENTLY`. A scan longer than 10 minutes counts as "too long". OEMs can change these values through DeviceConfig [AOSP:src/com/android/bluetooth/btservice/AdapterService.java][AOSP:src/com/android/bluetooth/le_scan/ScanController.java][AOSP:src/com/android/bluetooth/le_scan/AppScanStats.java]. The connect screen should run one filtered scan and never restart it in a loop.

**Connection parameters.**
- In AOSP's config, `CONNECTION_PRIORITY_HIGH` asks for a 9–12 unit interval (11.25–15 ms), and the default BALANCED asks for 24–40 units (30–50 ms), at 1.25 ms per unit. The same file has separate `_primary`/`_secondary` values for companion devices (for example HIGH 6–8 units with latency 45), so the interval actually used can differ [AOSP:res/values/config.xml].
- The API docs say to request HIGH "only ... to transfer large amounts of data over LE quickly" and then go back to BALANCED [69]. Holding HIGH for the length of a timed solve, then dropping it, is our judgement, not Google's guidance.
- The interval bounds how precise host receive timestamps can be (§5).
- From Android 14, the stack requests an ATT MTU of 517 when the first GATT client calls `requestMtu`, and ignores later requests [69].

**Android 16 and 17.** Android 16's Bluetooth changes for all apps are bond-loss handling and the Companion Device Manager discovery timeout. Apps targeting 16 also get bond-loss intents and a CDM `removeBond` API. Android 17 adds autonomous re-pairing after bond loss [70][71][72]. All of these concern bonded devices. The open drivers never bond with a cube (open-gan has no `createBond` call; SWB uses Web Bluetooth, which has no bonding API), so nothing changes for the owner's Galaxy S25 on Android 16 [79]. No One UI 8 BLE notification regression was found.

**Library choice.**

| Option | Version and licence | Pros | Cons |
|---|---|---|---|
| **Kable** (recommended) | 0.45.0, git tag 2026-09-14, on Maven Central 2026-09-15; Apache-2.0; AAR minSdk 21; coroutines 1.11.0, Kotlin 2.4.10 [66][KBL:LICENSE.txt][KBL:gradle/libs.versions.toml#L3-L5] | Coroutine Flows; GATT operations serialised behind a `Mutex` [KBL:kable-core/src/androidMain/kotlin/Connection.kt#L102]; exposes the address, `requestConnectionPriority`, `requestMtu`, per-company manufacturer data and raw scan bytes [KBL:kable-core/src/androidMain/kotlin/AndroidPeripheral.kt#L110-L171]; already planned in the stack note [80] | Pre-1.0 API. Of 182 commits in 2026, 140 are by a bot, 32 by Travis Wyatt and 10 by others: active, but one main human maintainer. `manufacturerData` property returns only the first entry |
| Platform `android.bluetooth` | – | No dependency; exact control of the callback thread for host timestamps | Must write the CCCD (0x2902) by hand, handle the pre- and post-API-33 `onCharacteristicChanged`/`writeDescriptor` signatures, and queue GATT operations one at a time, as open-gan's `GattConnection` does [OG:app/src/main/kotlin/app/opengan/ble/GattConnection.kt] |
| Nordic Kotlin-BLE-Library | 2.0.0-beta09 (tag 2026-10-28); BSD-3-Clause [65] | Backed by a BLE chip vendor | Still beta |

Kable's minSdk (21) does not constrain Rouxtine's minSdk.

## 5. Timing quality

**Which protocols carry a cube clock.**
- With cube timestamps: GAN Gen2 (16-bit deltas, summed by the driver, host fallback on overflow), GAN Gen3/Gen4 (32-bit), MoYu32 (16-bit offsets, accumulated, re-anchored to the host when drift exceeds 2 s), MoYu MHC (32-bit, 1/65536 s), QiYi (32-bit, 0.625 ms ticks) and GAN Gen1 (raw 16-bit stamps, polled every 30 ms) [SWB:src/gan-cube-protocol.ts#L412-L443][SWB:src/gan-cube-protocol.ts#L976-L1005][SWB:src/smartcube/protocols/moyu32.ts#L300-L345][SWB:src/smartcube/protocols/moyu-mhc.ts#L123-L170][SWB:src/smartcube/protocols/qiyi.ts#L290-L306][SWB:src/gan-gen1.ts#L285-L310].
- Without: GoCube, Rubik's Connected and Giiker/Mi. Their `cubeTimestamp` is null in every capture move, so splits rely on host receive time only [SWB:src/smartcube/protocols/gocube.ts#L127-L143][SWB:src/smartcube/protocols/giiker.ts#L150-L165][SWB:captures].

**Measured jitter.** My own calculation over SWB's captures (Web Bluetooth hosts, 6–31 moves per file, recovered moves without a host time excluded), comparing the interval between consecutive moves on the host clock with the same interval on the cube clock [SWB:captures]:

| | Median error | 95th percentile | Max | Host/cube slope |
|---|---|---|---|---|
| GAN Gen2 (3 files) | 9–12 ms | 19–56 ms | 60 ms | 0.998–1.008 |
| GAN Gen4 (2 files) | 17–18 ms | 32–49 ms | 81 ms | 0.993–1.003 |
| MoYu32 (4 files, one a 2x2) | 6–10 ms | 11–22 ms | 28 ms | 1.000–1.003 |
| QiYi (2 files) | 7–26 ms | 24–53 ms | 72 ms | 1.000–1.009 |

A verifier reproduced these ranges; with interpolated percentiles the 95th is 12.6–66 ms. Two caveats. MoYu32's cube time is partly host-anchored, which flatters it. And with so few moves per file, part of the slope spread is jitter, not clock error.

**What this means for Roux.** Errors of tens of ms are small next to a whole step of 0.5 s or more. They matter for per-move pauses, recognition gaps between steps (for example the pause before CMLL or before EOLR) and TPS. A clock skew near 1% would add up to about 0.1 s over a 10 s solve. SWB's README says the internal clock of most GAN cubes "is not ideally calibrated" [SWB:README.md#L216-L223].

**Best practice (csTimer, SWB).**
- Record both the host and the cube clock for every move, fit cube→host time by linear regression over the solve, and compute splits from the fitted cube times. Chen Shuang introduced this in csTimer [SWB:README.md#L216-L223][GWB:README.md#L127-L134].
- SWB's `cubeTimestampLinearFit` fills moves with no cube time at ±50 ms from their neighbours and returns times relative to the first move. `cubeTimestampCalcSkew` reports skew in percent [SWB:src/utils.ts#L48-L90].
- SWB's host clock is `Math.floor(performance.now())`, whole ms [SWB:src/utils.ts#L9-L13]. On Android, take `SystemClock.elapsedRealtimeNanos()` as early as possible in the notification path (recommendation, not measured).
- For host-only protocols (GoCube, Giiker), the connection interval dominates: HIGH priority (11.25–15 ms) instead of BALANCED (30–50 ms) should cut the error roughly in proportion (inference from §4).
- QiYi needs ACK writes, and GAN and MoYu32 need occasional requests. Keep those writes off the timestamp path and queue them, as SWB does [SWB:src/gatt-characteristic-write.ts].
- GAN i4 users should time solves in the 30-hour Performance mode. GAN says the 200-hour mode has "Recording disabled" [8]; what the cube then sends is untested.

## 6. Reusable code and licences

Rouxtine is GPL-3.0-only [77][79]. Every source below except CubeDesk can be used in it. MIT, BSD and Apache-2.0 code needs its notices kept. GPL-3.0-or-later code can be used under v3. cubing.js can be taken under its GPL option.

| Source | Licence | Language | Covers | State | Use for |
|---|---|---|---|---|---|
| **smartcube-web-bluetooth** (SWB) | MIT (Pau Oliva, Andy Fedotov) [SWB:LICENSE.txt] | TypeScript + RxJS; deps aes-js, lz-string, rxjs | GAN Gen1–4 (including GAN 251 UI 2x2), Giiker/Mi/Hi-, GoCube/Rubik's Connected, MoYu AI 2023, MHC, MoYu32 (incl. WeiPo V5 AI 2x2), QiYi incl. Tornado V4; GAN Smart Timer and Halo [SWB:README.md#L34-L48][SWB:README.md#L148-L152][SWB:package.json#L41-L52] | v4.1.0, HEAD `2077416` (2026-10-20), unchanged on 2026-10-29. 37 commits by Oliva and 1 by Bryan Lundberg since 2026-03-01 | **Main port source** |
| SWB capture fixtures | MIT | JSON | 17 files: raw encrypted traffic, MAC, expected events. GAN12ui, GAN12uiFp, GANicXXX (Gen2); GANi4, GAN16ui (Gen4); Giiker i3SE; GoCube; Rubik's Connected; QiYi QY-QYSC-S; Tornado V4; WCU_MY32 (2); WCU_MY33; WCU_MY22 (4) [SWB:captures][SWB:src/smartcube/protocols/gan.replay.test.ts][SWB:src/smartcube/protocols/qiyi.replay.test.ts] | **No capture for GAN Gen1, Gen3, MoYu MHC, MoYu AI 2023 or the no-gyro V10 AI, and no Gen4 batched notification.** One undated WCU_MY22 file has no expected events | JVM replay tests of each decoder that has a capture |
| gan-web-bluetooth | MIT (licence file `LICENSE.txt`) | TypeScript | GAN Gen2–4, GAN timers | v3.0.2, no commit since 2024-08-22 [58]. SWB continues its git history and keeps its GAN API [GWB:README.md][SWB:README.md#L29-L32] | Superseded by SWB |
| **QiYi protocol spec** | MIT, added 2026-01-16 [QYS:LICENSE] | Markdown + Rust example app + Wireshark dissector | QiYi | Moved from GitHub to Codeberg in Jan 2026 [54][55] | QiYi driver, Sync/Request State |
| **GoCube protocol spec** | MIT (2020, last edited 2025-01-11) [GCP:LICENSE] | Markdown | GoCube | Community spec [56] | GoCube driver |
| csTimer drivers | GPL-3.0 [CST:LICENSE] | JavaScript | gancube, giikercube, gocube, moyucube (MHC), moyu32cube, qiyicube; also GAN and QiYi timers and Stackmat | Maintained: qiyicube changed 2026-10-25, gancube 2026-09-07 (GAN 251 UI), moyu32cube 2026-08-08 (unbound company ID). giikercube and gocube unchanged since 2024-08-22. HEAD moved to `cb4f3fa` on 2026-10-29 without touching the drivers [53] | Cross-check; Roux split ideas [78] |
| cubing.js bluetooth | MPL-2.0 OR GPL-3.0-or-later [CJS:package.json] | TypeScript | GAN Gen1 only (no current GAN), GoCube, HEYKUBE, QiYi (`QY-QYSC` names only), GiiKER; GAN Robot; GAN timer. No MoYu [CJS:src/cubing/bluetooth/smart-puzzle/connect.ts#L6-L18][CJS:src/cubing/bluetooth/smart-puzzle/gan.ts#L151-L156] | Bluetooth code last changed 2026-10-08 [61] | HEYKUBE; GAN Robot docs |
| heykube_python | MIT (22nd Solutions, 2021) [HKP:LICENSE] | Python (bleak) | HEYKUBE | Beta v0.5.0; the maker links it as its "open API" [57] | HEYKUBE, if ever needed |
| open-gan | GPL-3.0-or-later [OG:README.md#L51-L58] | **Kotlin**: pure-Kotlin core + Compose BLE app; minSdk 26, targetSdk 35 [OG:app/build.gradle.kts#L9-L14] | GAN Gen2–4, MoYu AI 2023, timestamp fit, GAN Robot. No Gen1, no GAN 251 UI | 8 commits between 2026-10-22 and 2026-10-27, all by "Claude". Its README says the app "has not been tested on real hardware yet" [OG:README.md#L6-L7]. Gen4 reads only the first MOVE chunk; no `requestMtu`/`requestConnectionPriority`; unfiltered scan [OG:core/src/main/kotlin/app/opengan/core/gan/GanProtocolDrivers.kt#L277-L314] | Kotlin reference only; fix the chunk bug if any code is reused |
| CubicTimer | GPL-3.0 | Java (Android) | GAN Smart Timer, GAN Halo, QiYi timer over BLE | Last release 2026-05 [64] | Smart timers (see [76]) |
| Cubedex | MIT | Web PWA | Built on SWB [62] | – | UX reference |
| CubeDesk | Ambiguous: `LICENSE.md` is GPL-3.0, `package.json` says "All Rights Reserved" | Web | PR #257 adds QiYi, GAN Gen1 and MoYu AI [63] | – | Avoid copying |

No native Kotlin/Android library covers several cube brands. The only Kotlin cube code found is open-gan, and it is GAN only [OG]. Other projects are web or desktop: Cubedex, CubeDesk, and Rust or Python terminal tools [62][63].

**Options for the driver code.**

| Option | Pros | Cons |
|---|---|---|
| **Port SWB to Kotlin** (recommended) | MIT; every protocol except HEYKUBE; active; replay captures to test against; already handles Gen4 batching, MoYu32 MAC probing and QiYi history | TypeScript + RxJS idioms to translate; inconsistent `direction`; no QiYi solved-glitch workaround; no captures for Gen1, Gen3, MHC, AiCube |
| Reuse open-gan's Kotlin core | Already Kotlin; GPL is fine for Rouxtine | GAN only; untested on hardware; Gen4 single-chunk bug; brand new, single author |
| Port csTimer | Maintained; the reference for the linear fit | GPL-3.0 (fine here, but blocks a later permissive split); no Tornado gyro; reads 9 QiYi history moves; tied to csTimer globals |

open-gan and csTimer: ideas only (HANDOFF "Don't").

The licensing note prefers MIT sources for new code to keep options open [77]. It also asks to confirm that SWB's linear fit is not copied csTimer code before relying on its MIT status; under GPL-3.0-only this does not matter [77].

**Other hardware the same code could reach.** GAN Smart Timer and Halo (SWB, csTimer, CubicTimer) and the QiYi timer (csTimer, CubicTimer) are covered in [76]. The GAN Robot, a cube-turning scrambler, is driven by cubing.js and open-gan, and cubing.js documents its protocol [CJS:src/cubing/bluetooth/docs/gan-robot.md][OG:README.md#L1-L20]. It could scramble automatically for drills (not evaluated).

## 7. Implementation order and core design

**Order** (reasons in §1 and §3):

| Step | Protocol | Models | Test without hardware | Test with hardware |
|---|---|---|---|---|
| 1 | Core + **QiYi** | QiYi AI, X-Man Tornado V4 AI | 2 captures | QiYi AI (buy first) |
| 2a | **GAN Gen4** | GAN16 ui MAX, i4, 12 ui Maglev, 14 ui; probably i4 Leap | 2 captures; write a synthetic two-chunk test | GAN i4 (buy second) |
| 2b | **GAN Gen2** + MoYu AI 2023 key | Older GAN12 ui, FreePlays, i Carry/i 3, probably i Carry E | 3 captures (none for AiCube) | – |
| 2c | **GAN Gen3** | i Carry 2 | None | User reports |
| 3 | **MoYu32** | WeiLong V10 AI; probably V11 AI, Super AoLong AI, AoLong V5 AI | 3 3x3 captures (none of the no-gyro unit) | User reports |
| 4 | **GoCube** | GoCube, Edge, Rubik's Connected | 2 captures + MIT spec | – |
| 5 | On demand | Giiker/Mi (1 capture), GAN Gen1, MoYu MHC, HEYKUBE (no captures) | – | – |
| 6 | Unknown | TokTok, Super WeiLong V2 AI, GAN i Carry 4, 12 ui SP, i3 V2 | Need a user capture | – |

**Core, built once for all protocols.**
1. **Connect:** one filtered scan (names and service UUIDs); reject 2x2 names; read the MAC from manufacturer data, then `getAddress()`, then name-derived candidates; confirm by a decrypt probe.
2. **Link setup:** `requestMtu(517)`, then subscribe, then protocol init (QiYi Hello, GAN/MoYu32 facelets request). Serialise every GATT operation through one queue.
3. **Events:** a unified `Move(face, turn, cubeTime?, hostTimeNanos?, serial?)`, normalised from the move string, not from driver-specific direction codes.
4. **Roux layer:** merge opposite-face pairs inside a short window into M/E/S; track centres and whole-cube orientation; detect stages modulo rotation.
5. **Timing:** host and cube clocks per move; linear fit per solve; HIGH connection priority from inspection to the end of the solve, BALANCED otherwise.
6. **Recovery:** request history (Gen3/4) or facelets (others) after a gap; for QiYi, use the spec's Request State and Sync State; apply the QiYi needs-ACK-as-solved rule.
7. **Diagnostics:** a "record BLE capture" export in SWB's fixture format (raw traffic, MAC, decoded events). It turns every unsupported cube into a test case.

## Sources

[1]: https://speedcubeshop.com/collections/smart-cubes/products.json?limit=250 (fetched 2026-10-28 and 2026-10-29)
[2]: https://www.thecubicle.com/collections/smart-cubes-and-bluetooth/products.json?limit=250
[3]: https://www.gancube.com/collections/all/products.json
[4]: https://www.gancube.com/products/gan16-ui-maglev-max
[5]: https://speedcubeshop.com/products/16-ui-maglev-max-uv
[6]: https://www.thecubicle.com/products/gan16-ui-maglev-max-uv-3x3-smart-cube
[7]: https://www.atoutcubes.com/en/smart-cubes-robots/70807-gan16-ui-smart-cube-maglev-max-uv-minions-4000000001263.html
[8]: https://www.gancube.com/products/gan-i4-smart-cube
[9]: https://www.gancube.com/products/gan-i4-leap
[10]: https://cuboss.com/wp-json/wc/store/v1/products/78354 (Leap Frosted) ; https://cuboss.com/wp-json/wc/store/v1/products/80916 (MagLev Frosted) ; https://cuboss.com/wp-json/wc/store/v1/products/78355 (MagLev UV) ; https://cuboss.com/shop/gan-i4-3x3-smart-cube/
[11]: https://www.atoutcubes.com/en/2081-smart-cubes-robots
[12]: https://www.gancube.com/products/gancube-gan356-i-carry-e-smart-cube (i Series comparison table)
[13]: https://www.gancube.com/products/gan-i-carry-4-smart-cube
[14]: https://speedcubeshop.com/products/gan-356-i-carry-e-3x3-magnetic-bluetooth-smart-cube
[15]: https://cuboss.com/shop/gan-356-i-carry-e-3x3-smart-cube/
[16]: https://speedcubeshop.com/products/gan-356-i-carry-2-3x3-bluetooth-smart-cube-magnetic-uv-coated
[17]: https://speedcubeshop.com/products/gan-12-ui-sp-3x3-magnetic-uv-coated-bluetooth-smart-cube
[18]: https://www.thecubicle.com/products/gan356-i3-v2-3x3
[19]: https://www.atoutcubes.com/en/smart-cubes-robots/67349-gan-12-ui-maglev-uv-10th-anniversary-box-set-4000000000921.html
[20]: https://www.thecubicle.com/search/suggest.json?q=freeplay
[21]: https://www.thecubicle.com/products/moyu-weilong-v10-ai-3x3-lite
[22]: https://cuboss.com/shop/moyu-weilong-v10-ai-3x3/
[23]: https://www.thecubicle.com/products/moyu-weilong-v11-ai-3x3-maglev-uv
[24]: https://speedcubeshop.com/products/moyu-super-aolong-ai-3x3-magnetic-maglev-20-magnet-ball-core-uv-coated-bluetooth-smart-cube
[25]: https://speedcubeshop.com/products/aolong-ai-3x3-v5-magnetic-maglev-ball-core-uv-coated-bluetooth-smart-cube
[26]: https://www.thecubicle.com/products/moyu-aolong-v5-ai-3x3-smart-cube-magnetic
[27]: https://www.atoutcubes.com/en/smart-cubes-robots/67656-weilong-v11-ai-3x3-maglev-gyroscope-uv-4000000000907.html
[28]: https://www.atoutcubes.com/en/3x3-cubes/67548-super-weilong-v2-ai-18th-anniversary-4000000020264.html
[29]: https://cuboss.com/wp-json/wc/store/v1/products?search=aolong
[30]: https://speedcubeshop.com/products/qiyi-ai-3x3-magnetic-bluetooth-smart-cube
[31]: https://www.thecubicle.com/products/qiyi-smart-cube-3x3
[32]: https://cuboss.com/shop/qiyi-smart-cube-3x3/
[33]: https://www.atoutcubes.com/en/smart-cubes-robots/67110-66544-qiyi-smart-cube-3x3-4000000007487.html (black, EUR 19.90, in stock 2026-10-29) ; https://www.atoutcubes.com/en/smart-cubes-robots/67327-qiyi-smart-cube-3x3-bluetooth-uv-4000000007487.html (UV, EUR 22.90, out of stock)
[34]: https://speedcubeshop.com/products/x-man-tornado-v4-ai-3x3-magnetic-uv-coated-charging-case-bluetooth-smart-cube
[35]: https://cuboss.com/shop/x-man-tornado-v4-ai-3x3-m-uv/
[36]: https://particula-tech.com/products.json
[37]: https://giiker.com/products.json
[38]: https://www.thecubicle.com/products/xiaomi-giiker-cube
[39]: https://www.thecubicle.com/products/heykube-3x3
[40]: https://www.heykube.com/
[41]: https://www.thecubicle.com/products/toktok-cube-3x3
[42]: https://apps.apple.com/np/app/ex-mars-connect/id6451303338
[43]: https://www.atoutcubes.com/en/plus-distribue/49170-49173-monster-go-ai-bluetooth-3x3.html
[44]: https://cuboss.com/shop/gan-251-m-ui-2x2-smart-cube-ball-core-uv/
[45]: https://github.com/cs0x7f/cstimer/issues/407
[46]: https://github.com/cs0x7f/cstimer/commit/7c13949489d7789d3be6593bd729578b4b6284b5
[47]: https://github.com/cs0x7f/cstimer/issues/518
[48]: https://github.com/poliva/smartcube-web-bluetooth/commit/1e44c2c8b9c4ba44f9157673e9443e4d4b44ac62 ; https://github.com/poliva/smartcube-web-bluetooth/commit/7bbd5e969d6c27990d6fecf89ecccc1fe65521d8
[49]: https://github.com/poliva/smartcube-web-bluetooth/commit/436e88a76abbf656bb460a40dae3d9d5a6d5fd51
[50]: https://github.com/poliva/smartcube-web-bluetooth/commit/d517e49ca1460761b92595d05a11b1fecef42f33
[51]: https://github.com/poliva/smartcube-web-bluetooth/issues/2
[52]: https://github.com/cs0x7f/cstimer/issues/488
[53]: https://github.com/cs0x7f/cstimer/commit/78ab4b0b9ddbba03bd365ab7c2dc2d8fa4444514 ; https://github.com/cs0x7f/cstimer/commit/dbeb68353b06ac7f3a79ece0ee113fe46aca628b ; https://github.com/cs0x7f/cstimer/commit/13babe21f46ac3e4a8eb9dc3486f867bb92c955a ; https://github.com/cs0x7f/cstimer/commit/cb4f3faac4d699c170a9c860f272eec157a3bcc5
[54]: https://codeberg.org/Flying-Toast/qiyi_smartcube_protocol
[55]: https://github.com/Flying-Toast/qiyi_smartcube_protocol
[56]: https://github.com/oddpetersson/gocube-protocol
[57]: https://github.com/heykube/heykube_python ; https://heykube-python.readthedocs.io/en/latest/ ; https://www.heykube.com/pages/pythonlibrary
[58]: https://github.com/afedotov/gan-web-bluetooth
[59]: https://gist.github.com/afedotov/52057533a8b27a0277598160c384ae71
[60]: https://github.com/cs0x7f/cstimer/wiki/Use-csTimer-to-connect-to-smart-cubes---%E4%BD%BF%E7%94%A8csTimer%E8%BF%9E%E6%8E%A5%E6%99%BA%E8%83%BD%E9%AD%94%E6%96%B9
[61]: https://js.cubing.net/cubing/bluetooth/
[62]: https://github.com/poliva/cubedex
[63]: https://github.com/kash/cubedesk/pull/257
[64]: https://github.com/hato-ya/CubicTimer
[65]: https://github.com/NordicSemiconductor/Kotlin-BLE-Library
[66]: https://repo1.maven.org/maven2/com/juul/kable/kable-core-android/0.45.0/kable-core-android-0.45.0.aar ; https://repo1.maven.org/maven2/com/juul/kable/kable-core-android/0.45.0/kable-core-android-0.45.0.pom ; https://github.com/JuulLabs/kable/releases/tag/0.45.0
[67]: https://developer.android.com/develop/connectivity/bluetooth/bt-permissions
[68]: https://developer.android.com/reference/android/bluetooth/BluetoothDevice
[69]: https://developer.android.com/reference/android/bluetooth/BluetoothGatt
[70]: https://developer.android.com/about/versions/16/behavior-changes-all
[71]: https://developer.android.com/about/versions/16/behavior-changes-16
[72]: https://developer.android.com/about/versions/17/behavior-changes-all
[76]: timer-integration.md (this folder), "Timing solves in our own app"
[77]: licensing-monetization.md (this folder), Summary and §6
[78]: roux-training-features.md (this folder), §3 and §4
[79]: https://github.com/nauroth/rouxtine/blob/main/docs/HANDOFF.md ("Licence", "minSdk", "Smart cubes")
[80]: android-ui-stack.md (this folder), Summary item 5

## Open questions / unverified

- **Unconfirmed GAN protocols.** Which protocol do the GAN i Carry 4, i Carry E V2, GAN12 ui SP, GAN356 i3 V2 and i4 Leap speak? Does the 2026 "PowerPod" GAN12 ui Maglev still speak Gen4? No open driver, capture or issue names them. The unlabelled `GANicXXX` capture (hardware name `GANicEqV`, Gen2, no gyro) may be an i Carry E. A short nRF Connect scan of each (name, services) would settle it.
- **MoYu WCU cubes.** Do the WeiLong V11 AI, Super AoLong AI and AoLong V5 AI use MoYu32 with the same move codes? Which product advertises `WCU_MY33`?
- **MoYu Super WeiLong V2 AI.** A verifier reports that csTimer issue #488 (closed "not planned", 2025-08-02) says "The cube is recognized, but does not record cube turns" [52]. GitHub issue pages returned HTTP 403 here on 2026-10-29, so this was not re-read.
- **TokTok Cube.** No public protocol or driver; it needs a capture, reverse engineering or vendor contact.
- **New GAN features.** Do the i4 ("zero-loss sensing") and 16 ui MAX send message types beyond Gen4 moves, gyro, facelets and battery? What does the i4 send in its 200-hour mode, where "Recording [is] disabled" [8]?
- **MTU.** Do QiYi (48/96-byte notifications, 32-byte App Hello) and GoCube (66 bytes) work on Android without `requestMtu(517)`? Inferred from capture sizes, not tested.
- **Connection interval on the S25.** Does One UI 8 honour AOSP's 11.25–15 ms HIGH interval? Does a CDM association switch to the companion-device values? Measure host-vs-cube jitter on the real phone.
- **MAC stability.** Are cube addresses stable across power cycles for every model? The MoYu32 and QiYi capture MACs have the top two bits set (static random), but the Gen4 GAN MACs `0C:3D:5E:…` and `54:6C:50:…` do not. Is MoYu32's fourth octet (00/01/02) fixed per unit or per boot? `getAddressType()` needs API 35, so it cannot be checked on older phones.
- **Wide moves.** That r arrives as a single L turn is inferred from how sensors measure faces against the core (§3.5), not observed.
- **`neverForLocation`.** The docs say it filters "some BLE beacons". That no cube advertisement is filtered is assumed.
- **Kable latency.** Does collecting Kable's observe Flow add measurable delay to host receive timestamps compared with timestamping in the platform callback?
- **QiYi slice report.** SWB issue #2 (late second face on Tornado V4 slices) was closed by its reporter with no fix described. Whether it is firmware, the driver or the host is unknown [51].
- **GoCube.** Does the Edge advertise a name starting with `GoCube` (needed for SWB to enable orientation)? Can a truncated second-face frame be told apart from a genuine opposite-face turn?
- **HEYKUBE availability** is contradictory: TheCubicle says discontinued, the maker's page says in stock under a © 2022 footer [39][40].
- **Market share.** Shelf counts measure listings, not sales or what the Roux community actually owns.
- **Tooling gaps.** No EU price was found for the standard GAN16 ui MAX: kubekings.com returned a Cloudflare 403, speedcubes.be a proxy CONNECT 502 and speedcube.de a 522; cubikon.de and cubeshop.cz had TLS name mismatches (probably wrong host names). The GitHub REST API and GitHub issue pages were blocked (HTTP 403), so the state of SWB issue #2, csTimer issues #407, #488 and #518, and newer SWB issues was not re-read on 2026-10-29; git clones and `git ls-remote` worked. Maven Central metadata returned HTTP 429; the Kable 0.45.0 AAR and POM downloaded earlier.
