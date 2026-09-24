# SideStore

> SideStore is an *untethered, community driven* alternative app store for non-jailbroken iOS devices

[![License: AGPL v3](https://img.shields.io/badge/License-AGPL%20v3-blue.svg)](https://www.gnu.org/licenses/agpl-3.0)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://makeapullrequest.com)
[![Alpha SideStore build](https://github.com/Greenzin1/SideStore/actions/workflows/alpha.yml/badge.svg)](https://github.com/Greenzin1/SideStore/actions/workflows/alpha.yml)
[![Stable SideStore build](https://github.com/Greenzin1/SideStore/actions/workflows/stable.yml/badge.svg)](https://github.com/Greenzin1/SideStore/actions/workflows/stable.yml)
[![Release](https://img.shields.io/github/v/release/Greenzin1/SideStore?label=Alpha)](https://github.com/Greenzin1/SideStore/releases)

Fork de [SideStore/SideStore](https://github.com/SideStore/SideStore) mantido por [Greenzin1](https://github.com/Greenzin1), com correções para o [issue #1466](https://github.com/SideStore/SideStore/issues/1466) — *Device Reachability Error / DeviceEndpointNotInitialized no iOS 16*.

SideStore is an iOS application that allows you to sideload apps onto your iOS device with just your Apple ID. SideStore resigns apps with your personal development certificate, and then uses a [specially designed VPN](https://github.com/jkcoxson/em_proxy) in order to trick iOS into installing them. SideStore will periodically "refresh" your apps in the background, to keep their normal 7-day development period from expiring.

## Fork changes

Correções em cima da tag `0.7.0-alpha`:

- **Fix do `DeviceEndpointNotInitialized` (iOS 16)** — [SideStore#1466](https://github.com/SideStore/SideStore/issues/1466):
  - `isReady()` refaz a descoberta de endpoint enquanto ele não inicializar (com retry);
  - `refreshEndpoint()` não descarta mais a descoberta quando `ifacesChanged` é falso mas o endpoint ainda não está pronto;
  - túnel do LocalDevVPN é mantido mesmo quando o probe TCP falha (antes retornava `nil`);
  - filtro de interfaces relaxado: aceita qualquer `utun` com IPv4 quando não há IPv4 estrito;
  - probe TCP com retry (750ms) em vez de um único tiro;
  - candidatos de fallback `10.7.0.1` / `10.7.0.2`.
- **Botão "Ask for Local Network Access" na Connection Config** — se o setup foi pulado (pairing key já presente), o iOS nunca mostra o prompt de Rede Local e bloqueia silenciosamente as conexões com o túnel; o botão força o prompt e mostra o status (Granted/Denied).
- Submodule `minimuxer` aponta para o fork [Greenzin1/minimuxer](https://github.com/Greenzin1/minimuxer) com as correções acima.

## Installation

1. Baixe o IPA em [Releases → Alpha](https://github.com/Greenzin1/SideStore/releases/tag/alpha).
2. Instale com o instalador de sua escolha (iLoader, SideStore antigo, etc.).
3. LocalDevVPN: **Tunnel IP** `10.7.0.2/30`, **Device IP** `10.7.0.1/32` → reconectar.
4. No SideStore: Settings → Advanced → Connection Config → permita a Rede Local → **Confirm**.

## Requirements
- Xcode 15
- iOS 14+
- Rustup (`brew install rustup`)

Why iOS 14? Targeting such a recent version of iOS allows us to accelerate development, especially since not many developers have older devices to test on. This is corrobated by the fact that SwiftUI support is much better, allowing us to transistion to a more modern UI codebase.

## Project Overview

### SideStore
SideStore is a just regular, sandboxed iOS application. The AltStore app target contains the vast majority of SideStore's functionality, including all the logic for downloading and updating apps through SideStore. SideStore makes heavy use of standard iOS frameworks and technologies most iOS developers are familiar with.

### EM Proxy
[EM Proxy](https://github.com/jkcoxson/em_proxy) powers the defining feature of SideStore: untethered app installation. By leveraging a custom-built App Store app with additional entitlements ([LocalDevVPN](https://github.com/jkcoxson/LocalDevVPN)) to create the VPN tunnel for us, it allows SideStore to take advantage of [Jitterbug](https://github.com/osy/Jitterbug)'s loopback method without requiring a paid developer account.

### Minimuxer
[Minimuxer](https://github.com/jkcoxson/minimuxer) is a lockdown muxer that can run inside iOS’s sandbox. It replicates Apple’s usbmuxd protocol on macOS to “discover” devices to interface with LocalDevVPN on-device.

### Roxas
[Roxas](https://github.com/rileytestut/roxas) is Riley Testut's internal framework from AltStore used across many of their iOS projects, developed to simplify a variety of common tasks used in iOS development.

We're hoping to eventually eliminate our dependency on it, as it increases the amount of unnecessary Objective-C in the project.

## Contributing/Compilation Instructions

Please see [CONTRIBUTING.md](./CONTRIBUTING.md)

## Licensing

This project is licensed under the **AGPLv3 license**.
