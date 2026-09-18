# WiFiShare for iOS

The iOS app: find free and open Wi-Fi from the WiFiShare data, and join it.

## Status

**The code is not written yet.** This repository holds the plan and the license.
The iOS app is phase P3, after the data pipeline and API (P1) and the Android
app (P2). The project started in September 2026, no data is being collected yet
and no app has been released.

## What the app does

| It does | It does not |
| --- | --- |
| Read the published data and show nearby networks | Scan for Wi-Fi networks |
| Join a network on request | Join a network without asking you |
| Share the network the phone is connected to right now | Collect networks in the background |

## The scanning constraint

iOS does not allow apps to scan for Wi-Fi networks. This is a platform rule, not
a decision the project made and not something an entitlement unlocks. Apple
states it in [TN3111: iOS Wi-Fi API
overview](https://developer.apple.com/documentation/technotes/tn3111-ios-wifi-api-overview).

Everything about the app's scope follows from that:

- **Finding networks** means reading the published data, not looking at the air.
  The app knows what is nearby because the dump says so, not because the phone
  saw it.
- **Joining** uses `NEHotspotConfigurationManager`. The system shows its own
  prompt before joining; the app cannot join silently and does not try to.
- **Sharing** is limited to the network the phone is currently connected to,
  read with `NEHotspotNetwork.fetchCurrent`. One network at a time, the one you
  are on. There is no "scan walk" mode on iOS, and there cannot be. That mode
  exists only in the [Android app](https://github.com/WiFiShare/android).

If you have used the Android app, this is the difference to expect: on Android
the app can walk and collect, on iOS it can only contribute the network you are
already connected to.

## Requirements

| | |
| --- | --- |
| Minimum iOS | 17 |
| Language | Swift |

iOS 17 is the minimum because the app uses HPKE from CryptoKit, which requires
it. That is the only reason. If the crypto design changes in `spec`, this
minimum should be revisited rather than kept out of habit.

## Entitlements

Two, both requested from Apple on the app's identifier:

| Entitlement | Why |
| --- | --- |
| Access Wi-Fi Information | To read the current network with `NEHotspotNetwork.fetchCurrent`, which is what sharing is built on |
| Hotspot Configuration | To join a network with `NEHotspotConfigurationManager` |

Neither of these grants scanning. Nothing does.

## Privacy

The app follows the project's three rules: collect only what helps someone
connect, publish less than is collected, and keep nothing that ties an
observation to a person. There are no accounts, no device IDs and no advertising
IDs for contributors.

Networks whose SSID ends in `_nomap` or `_optout` are never collected, so the
app will not offer to share one. Private password-protected networks are not
collected at all unless you deliberately share one you own.

## What a contributor could start on

The API and the data format are being defined now, so the useful work is the
part that does not depend on them being finished:

- **Project skeleton.** Xcode project, iOS 17 target, the two entitlements
  declared, SwiftUI app shell, a test target, and a CI workflow that builds and
  runs tests.
- **Data client.** Fetch and parse an area GeoJSON file from the
  [data](https://github.com/WiFiShare/data) dump. The layout is
  `areas/<gh2>/<gh3>/<gh5>.geojson`; see that repository's README and SCHEMA.md,
  and the JSON Schemas in [spec](https://github.com/WiFiShare/spec).
- **Geohash.** Encode a coordinate, decode a cell to a bounding box, and find a
  cell's neighbours, so the app can work out which area files to fetch near a
  cell edge. Self-contained, testable against the fixtures in `spec`, and needed
  by everything else.
- **Offline cache.** Store fetched area files and expire them, so the app is
  useful where there is no connection, which is the situation it exists for.
- **Join flow.** A small wrapper around `NEHotspotConfigurationManager`,
  including what to show when the system prompt is declined or the join fails.
- **Share flow.** Read the current network with `NEHotspotNetwork.fetchCurrent`,
  apply the `_nomap` and `_optout` checks locally before offering to share, and
  make it obvious what will be sent.
- **Map and list UI**, including the honest presentation of precision: a
  community-found network is known to about 150 m, so it must not be drawn as a
  pin on a building.
- **Accessibility and localisation** from the start, not retrofitted.

Open an issue here before starting something substantial, so two people do not
build the same piece in different directions. See the organization's
[CONTRIBUTING.md](https://github.com/WiFiShare/.github/blob/main/CONTRIBUTING.md).
A change that affects what is collected or published needs test fixtures in
`spec`.

## License

[Mozilla Public License 2.0](LICENSE). Contributions are licensed under it. No
CLA, and no DCO sign-off required.
