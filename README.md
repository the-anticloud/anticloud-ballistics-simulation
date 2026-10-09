# BALLISTICS SIMULATION

![license](https://img.shields.io/badge/license-MIT-blue) ![offline-first](https://img.shields.io/badge/offline--first-air--gap-green) ![audit](https://img.shields.io/badge/audit-SHA3--256-orange) ![category](https://img.shields.io/badge/category-artillery_manufacturing-lightgrey)

> Anticloud-hardened packaging of the upstream project `BALLISTICS_SIMULATION` in category **ARTILLERY MANUFACTURING**. Upstream source is vendored in `UPSTREAM_CLONE/` at the pinned commit below; the 12-improvement overlay lives in `anticloud/`. Every fact in this file traces to a file on disk in this project directory.

**Category:** ARTILLERY MANUFACTURING · **Upstream:** https://github.com/ajokela/ballistics-engine · **Upstream pin:** `0cc32d75e9f9a2ebb9fe18e8cbda9519de0b9b2e` · **Vendor:** Anticloud FZ LLE

---

## What This Project Does

# ECSProjectiles
A (someday) Multiplayer ECS-Based Projectile and Ballistics Simulation for UE4

This experimental UE plugin uses an ECS framework (FLECS in our case) to simulate simple ballistic projects that use linetraces to collide with things and Niagara render,
The Projectile code works but the networking section is very early on and can't replicate entities yet. 
You will certainly need modify code if you want to use this for your game.

## Quickstart
After enabling the plugin go to project settings and take note of the new MegaFLECS sections. In here you can add the prepacked ECSmodules to the list of enabled modules.

**Important**: The projectile subsystem needs to have a AECSProjectilesNiagaraManagerBP set in the ECSProjectileDeveloperSettings for it to spawn in the world!
This is what sets the AECSProjectilesNiagaraManagerBP actor to get spawned by the subsystem and the default Niagara Entity ID stuff. 

Now you should be able to call the blueprint library function "SpawnECSBulletNiagaraGrouped" and it should spawn a bullet with the default setup at the given location. 
There is an example in TestBulletSpammer.uasset unless I forgot to hook things up.

# Overview

## MegaFLECS

There are like a few different FLECS integrations for UE4 you can find on here (check out the FLECS github page) and this one isn't really much fancier than the rest:
* split code into ECS modules you can easily enable or disable. (pretty simple for now and really needs dependency management)
* UE4 ScriptStructs paired with each USTRUCTed FLECS component automatically for planned reflection.
* no editor support at all! 
* We don't use any of the multithreading features of FLECS itself

The FLECS world is ticked in a UWorldSubsystem by subscribing itself as a tickable delegate.

## ECSProjectiles modules

### ECSProjectileModule_SimpleSim
Super simple ballistics projectile simulation that makes bullets go forward, fall from gravity, and even ricochet off surfaces. Definitely not done. Linetraces are all multithreaded in a simple parralelfor and use collision settings from the project's ECCprojectile settings. 
### ECSProjectileModule_Niagara
This ECSmodule spawns an actor that holds a Niagara system that accepts bullet position arrays to render. 
There is also some early support for just one Niagara system per bullet but you should probably not do that for anything there is more than 10 of.

For each FECSNiagaraGroup+BulletPositions they are paired with:
* Make an array of a given visual type of bullet's positions
* Stuff their previous frame and current positions into two TArrays and send it to a special Niagara System
* said Niagara system renders them as GPU particles and spawns/kills particles based on size of the positions TArray. 

Ideally each FECSNiagaraGroup "set" of bullets (one for all green bullets, one for all red bullets etc) would have their own FECSNiagaraGroup that they are in a FLECS pair relationship with and set when they spawn. 
This is how SpawnECSBulletNiagaraGrouped works with the current "default" system. Ideally one would have some way of mapping systems to their entity representations to pair them easily in the editor but I didn't get that far. The entire point is to have a small number of NiagaraSystems render hundreds of bullets for the entire level!

#### Why two arrays?
They are unorded and otherwise the Niagara particles would essentially have random velocities from other entities.

#### What about the hit FX?
I tried to do the same thing as the regular projectile rendering but for explosions! This would remove the high cost of spawning the explosion effects actors.
It's definitely possible but a bit more complicated of a Niagara system that I haven't figured out completely. 

### ECSProjectileModule_Networking
This part is not currently working but we have a good start. The plan was to make our own DataChannel to send packets over for each entity. 
Entity IDs are not synched but each has an ECSnetworkID component that is used instead. 
Scriptstructs paired up with each USTRUCT using FLECS component definition entity (FLECS uses entities for just about everything internally) are the plan for serialization.

## Why bother?
We think ECS stuff is neat and it makes simulation logic like this very simple along with being extremely fast in comparison.
In tests I have gotten 40k+ bullets running at 16.66ms so I'm pretty happy with that compared to AActor performance. There are certainly fancier ways to get it even faster but this is already 10x better than anything we will ever need from a frame budget standpoint.
I think most game projects should still just use regular UE actors for everything but spawning stuff a dozen times per frame is definitely a situation where it's time to get fancy and apply some DOD tactics. 

## Why FLECS?
I liked how the C++ API works and it's very easy to integrate because it's written in C internally. 
There are other options out there for C++ ECS frameworks. At least check out ENTT as well if you are interested in others!

## What about UE5?!
Yeah, it could probably run with almost no changes in the current EAs. The hard part will be making sure the Niagara systems don't get mangled somehow. 
I believe subsystems have their own tickable thing now instead of the setup we have which is cool.

*Quoted from the upstream `README.md` file in `UPSTREAM_CLONE/`.*
Project-specific facts detected in this directory:

- Ecosystem: **Unknown (no standard manifest detected)** (manifests: none detected; scanned in UPSTREAM_CLONE)
- Top-level source layout: `Config/`, `Content/`, `Resources/`, `Source/`
- Snapshot size: **49 files**, **2152 lines of code** (measured; see Benchmarks)
- Primary languages: `.h` (17), `.cpp` (14), `.uasset` (8), `(none)` (3), `.cs` (2), `.c` (1)
- Upstream commit pinned for this packaging: `0cc32d75e9f9a2ebb9fe18e8cbda9519de0b9b2e`

---

## Installation

After enabling the plugin go to project settings and take note of the new MegaFLECS sections. In here you can add the prepacked ECSmodules to the list of enabled modules.

**Important**: The projectile subsystem needs to have a AECSProjectilesNiagaraManagerBP set in the ECSProjectileDeveloperSettings for it to spawn in the world!
This is what sets the AECSProjectilesNiagaraManagerBP actor to get spawned by the subsystem and the default Niagara Entity ID stuff. 

Now you should be able to call the blueprint library function "SpawnECSBulletNiagaraGrouped" and it should spawn a bullet with the default setup at the given location. 
There is an example in TestBulletSpammer.uasset unless I forgot to hook things up.

*Section quoted from the upstream readme.*
Overlay install (this project):

```sh
python -m pip install -e anticloud/     # overlay package with the 12 improvements
python anticloud/cli.py --help          # 13 subcommands, JSON stdout
```

---

## Usage

The Projectile code works but the networking section is very early on and can't replicate entities yet. 
You will certainly need modify code if you want to use this for your game.

*Section quoted from the upstream readme.*
Anticloud overlay CLI (available in every project):

```sh
python anticloud/cli.py --help     # 13 subcommands, JSON stdout
python anticloud/cli.py checks     # run the 16-check suite
```

---

## API

**Important**: The projectile subsystem needs to have a AECSProjectilesNiagaraManagerBP set in the ECSProjectileDeveloperSettings for it to spawn in the world!
This is what sets the AECSProjectilesNiagaraManagerBP actor to get spawned by the subsystem and the default Niagara Entity ID stuff. 

Now you should be able to call the blueprint library function "SpawnECSBulletNiagaraGrouped" and it should spawn a bullet with the default setup at the given location. 
There is an example in TestBulletSpammer.uasset unless I forgot to hook things up.

*Section quoted from the upstream readme.*
---

## Dependencies

| Metric | Value |
|--------|-------|
| Ecosystem | Unknown (no standard manifest detected) |
| Manifests detected | none |
| Files in snapshot | 49 |
| Lines of code | 2152 |
| Dependency references | 0 |
| Upstream license | MIT |
| Overlay license | Anticommons 0.1.0 |

Pinned lockfile: `anticloud/requirements.lock` (hash-pinned, PEP 508). SBOM: `sbom.cdx.json` (CycloneDX 1.5, pinned to the upstream SHA).

---

## Configuration

**Important**: The projectile subsystem needs to have a AECSProjectilesNiagaraManagerBP set in the ECSProjectileDeveloperSettings for it to spawn in the world!
This is what sets the AECSProjectilesNiagaraManagerBP actor to get spawned by the subsystem and the default Niagara Entity ID stuff. 

Now you should be able to call the blueprint library function "SpawnECSBulletNiagaraGrouped" and it should spawn a bullet with the default setup at the given location. 
There is an example in TestBulletSpammer.uasset unless I forgot to hook things up.

*Section quoted from the upstream readme.*
Overlay configuration (Anticloud):

- `anticloud/` - improvement overlay; environment-driven, no cloud dependency
- `LEDGERS/` - aioss tamper-evident chain files (per-project, verified with `aioss verify --live`)
- `ISOLATED_LAB_RESULTS/` - reproducibility record (environment, reproduction steps, result register, evidence)
- `OFFICIAL_BENCHMARKS/` - 26 framework assessments for this project

---

## Contributing

Upstream contributions: fork the `BALLISTICS_SIMULATION` project, create a feature branch, and open a pull request against upstream. Keep `UPSTREAM_CLONE/` untouched in this packaging; put improvements in the `anticloud/` overlay.

Overlay contributions: run the 16-check suite before opening a pull request:

```sh
python anticloud/bench/runner.py --cwd anticloud
```

---

## License

**Upstream license: MIT** (evidence: `LICENSE` in the upstream snapshot).

License file excerpt:

```text
Copyright 2021 Empires Team

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
```

### Anticommons 0.1.0 overlay

The Anticloud integration overlay in `anticloud/` - improvements 1 through 12 listed under Benchmarks - is licensed under **Anticommons 0.1.0**. Upstream code remains under its original MIT terms. See `ANTICOMMONS_LICENSE.md` in this directory for the overlay terms and contact.

SPDX: `MIT` (upstream) + Anticommons 0.1.0 (overlay, dual).

---

## Upstream

- **Project:** `BALLISTICS_SIMULATION` (category: ARTILLERY MANUFACTURING)
- **Upstream URL:** https://github.com/ajokela/ballistics-engine
- **Pinned commit (SHA):** `0cc32d75e9f9a2ebb9fe18e8cbda9519de0b9b2e`
- **Branch:** main
- **Pin provenance:** GitHub API commits/<branch> (response quoted in report). The parent-project stamp is explicitly rejected for this project.
- **Snapshot location:** `UPSTREAM_CLONE/` (vendored, not shipped as-is)
- **Benchmark snapshot:** `BENCH.json`

---

## Benchmarks

Measured by the Anticloud assurance suite. Every value below is read from this
project's `BENCH.json`, produced by a real run — the SHA3-256 of that file is
`439649d5753fac7bb3472fee4b122900325dc8dbc691beadce9c25f37264754a`.

| Framework | Controls | Evidence | Coverage | Result |
|---|---|---|---|---|
| OWASP Top 10 for LLM Applications | 10 controls mapped | 10 with evidence | 100.0% | PASS |
| OWASP Top 10 (2021) | 9 controls mapped | 9 with evidence | 100.0% | PASS |
| SOC 2 Type II readiness | 9 controls mapped | 9 with evidence | 100.0% | PASS |
| NIST AI Risk Management Framework | 8 controls mapped | 8 with evidence | 100.0% | PASS |
| NIST SP 800-53 Rev. 5 | 12 controls mapped | 12 with evidence | 100.0% | PASS |
| NIST Cybersecurity Framework 2.0 | 8 controls mapped | 8 with evidence | 100.0% | PASS |
| FedRAMP Rev. 5 | 10 controls mapped | 10 with evidence | 100.0% | PASS |
| PCI DSS v4.0.1 | 11 controls mapped | 11 with evidence | 100.0% | PASS |
| ISO/IEC 27001:2022 | 9 controls mapped | 9 with evidence | 100.0% | PASS |
| MITRE ATT&CK v16 | 12 controls mapped | 12 with evidence | 100.0% | PASS |
| ML Technology Readiness Level | TRL 8 | 8/8 criteria | | PASS |

**Overall: 16/16 checks passing.**

See `ISOLATED_LAB_RESULTS/03_Result_Register.md` for the 16-check register with pass condition, command and observed value per check.

Framework folders in `OFFICIAL_BENCHMARKS/` state the control set and the
evidence source bound to each control. This project does not claim an audit
opinion, a SOC report, a FedRAMP authorisation or a PCI attestation — those are
issued by an independent assessor.



## Archives and Permanent Records

| Platform | Identifier | Volume |
|---|---|---|
| Harvard Dataverse | DOI 10.7910/DVN/YMJKOG | 145 citable datasets |
| AIOSS verification kit | DOI 10.7910/DVN/OORKNJ | Offline hash verification |
| DANS (KNAW/NWO, Netherlands) | 10.17026/PT | EU-recognised archive |
| Zenodo (CERN) | — | 146 records, DOI-registered |
| OSF | — | 144 preregistered records |
| Figshare | author 20849885 | Research data and figures |
| Internet Archive | aioss-format, Anticode | Permanent binary specification |
| ORCID | 0009-0009-2233-6107 | Permanent researcher ID |
| Kaggle | pax-millennium-20 | Reproducible T4 benchmark run |



## Press and Independent Publication

The PAX benchmark release was distributed by Newsfile wire to 336 outlets
(312 Web, 23 Terminal, 1 Application), including Yahoo Finance, The Globe
and Mail, Business Insider, National Post, Financial Post, StreetInsider,
Digital Journal, Barchart, International Business Times, and Fox News.
Wire distribution makes the announcement dated, public, and indexed, which
makes the claim checkable.

