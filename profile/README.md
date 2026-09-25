# crt-bridge

**A hobby project: sending emulator frames over the network, at native resolution, to a CRT or
another computer.**

## How this was made

crt-bridge is a personal project by Konni. Most of the code, research and documentation was
written by an AI coding assistant working under my direction. I set the goals, ran every test
on the real hardware, judged the results with my own eyes and ears, and made the decisions. I am
responsible for what is here, errors included.

I'm a game developer, and real-time Linux, display drivers, modelines and network pacing were
all new territory for me. Things were measured on the bench rather than assumed — and plenty
went wrong along the way.

## What it does

The idea is not new, and this project does not start from scratch. It builds directly on
**[Groovy_MiSTer](https://github.com/psakhis/Groovy_MiSTer) by psakhis**: a MiSTer core that
receives frames from an emulator running on a PC — GroovyMAME, among others — over the network,
and shows them on a CRT at native resolution. psakhis designed the protocol and proved the
approach. Several emulators already speak it — GroovyMAME by Calamity, and psakhis's own forks
of Mednafen and 86Box — and **a RetroArch emitter already exists**: Antonio Giner's
[RetroArch fork](https://github.com/antonioginer/RetroArch/tree/mister) (`mister` branch).

crt-bridge speaks that same protocol, with its own pieces on both ends: a separate RetroArch
fork as the emitter, which captures the core's own frame through a recording driver and reuses
the gamepad driver of the fork above; and — for people without a MiSTer, like me — a Linux box
or a software receiver on the other side.

crt-bridge separates emulation from display. One machine runs the emulator. Each frame is sent
over the network at the resolution the game produced — 240p, 480i — to one or more receivers,
which try to show it at the right cadence, including mode changes:

- **a 15 kHz CRT**, driven by a small real-time Linux box with an AMD graphics card;
- **a software receiver**, a libretro core, on another computer — on the same network or over
  Wi-Fi. Playing across the internet is a goal, not yet something that works.

## Why

I don't own a MiSTer, and I like how games look on a CRT. Emulators are practical, and I have a
lot of respect for the work done on FPGA recreations. This doesn't try to compete with them —
for accuracy, FPGA remains the reference. It is simply what I could build with what I have: a PC
and a tube. The original curiosity was simple: what does PlayStation 3D look
like on a tube when it is anti-aliased but kept at native resolution — edges smoothed by
supersampling, while 2D, fonts and grain stay as the console drew them?

## How it works

```
 emitter (PC)                     network                        receivers
 ┌──────────────────────────┐   UDP/32100    ┌──────────────────────────────┐   ┌──────────┐
 │ RetroArch + record_groovy│ ─────────────► │ master: Linux box + AMD card │──►│ 15 kHz   │
 │ captures the core's      │  frames (LZ4), │ gives the clock              │   │ CRT      │
 │ native frame             │  audio, modes  └──────────────────────────────┘   └──────────┘
 └──────────────────────────┘ ────────────► ┌──────────────────────────────┐
                              ◄───────────  │ followers: libretro core on  │
                               UDP/32101:   │ another computer (up to 4)   │
                               gamepads     └──────────────────────────────┘
```

- **Emitter** — a RetroArch fork with a recording driver that captures the core's frame at its
  native size, before any shader, and sends it. Tested on Windows and Linux.
- **Master receiver** — a real-time Linux image, booted from the network, driving an AMD card's
  analog output at 15 kHz. The emitter follows its clock.
- **Followers** — up to four software receivers (a libretro core) fed from the same stream.
- **Wire protocol** — [Groovy](https://github.com/psakhis/Groovy_MiSTer), designed by psakhis
  for Groovy_MiSTer and GroovyMAME. The aim is to stay compatible with GroovyMAME.

Anti-aliased 3D at native resolution comes from the cores themselves: SwanStation (PlayStation)
already does it; paraLLEl-GS (PlayStation 2) and paraLLEl-RDP (Nintendo 64) look promising and
are still being tested. The bridge only carries the frame they produce.

## Things I would like to try

Playing with a friend over the internet. The 31 kHz band some CRTs accept, for 480p sources and
the PC-98's 400 lines. Other emitters and receivers. No promises — it depends on what the bench
shows.

## Repositories

| Repository | What it is |
|---|---|
| [RetroArch](https://github.com/crt-bridge/RetroArch) | The emitter: a RetroArch fork. The work lives on the `crt-bridge` branch. |

The rest is still in a private development repository and may be published later.

## Status

Early and personal. It runs on one bench — a JVC broadcast CRT, a Radeon HD 7750 receiver,
Windows and Linux emitters, a MacBook as follower — and has not been tried anywhere else. It is
not packaged, and there are known bugs. Feedback is welcome, but support is not something I can
promise.

## Credits

This builds on other people's work:

- **psakhis** — the Groovy protocol and the [Groovy_MiSTer](https://github.com/psakhis/Groovy_MiSTer)
  core, the work this project starts from.
- **Calamity** — [GroovyMAME](https://github.com/antonioginer/GroovyMAME) and its MiSTer video
  driver, the reference emitter for the Groovy protocol; and Switchres.
- **Antonio Giner** — the [RetroArch `mister` fork](https://github.com/antonioginer/RetroArch/tree/mister),
  the first RetroArch Groovy emitter; crt-bridge's gamepad input driver is ported from it.
- **Shane** — [MiSTerCast](https://github.com/iequalshane/MiSTerCast), a Windows desktop mirror
  for Groovy_MiSTer, consulted as a reference implementation of the protocol.
- **The Switchres authors** (Chris Kennedy, Antonio Giner, Alexandre Wodarczyk, Gil Delescluse,
  per its source) — the modeline engine the receiver uses.
- **The RetroArch and libretro teams** — the frontend this fork extends.
- **Stenzek** (DuckStation, which SwanStation derives from) and **Themaister** (paraLLEl-RDP,
  paraLLEl-GS) — the cores whose native-resolution supersampling this project relies on.
- **Yann Collet** — LZ4.

## License

The RetroArch fork is GPL-3.0, like RetroArch. crt-bridge is not affiliated with, or endorsed
by, the libretro project.

---

*Konni — built with an AI coding assistant.*
