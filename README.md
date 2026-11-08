# Minecraft dedicated server: rent or self-host, what it costs, how many players fit, and which port to open (plan table included)

Microsoft hands out the Bedrock Dedicated Server software as a free .zip for Windows and Ubuntu, so the download itself costs nothing. The money goes to the machine that stays on, the upload bandwidth, and the hour you spend rebuilding a world from a backup. The first real decision is whether you rent that machine or use hardware you already own. The plan table below covers the rental route; the rest of the article covers the things that push people toward renting in the first place.

> **1–4 friends on Bedrock:** the 2 vCPU / 2GB KVM VPS at **$12.00/month**. **Around ten players, or anyone who wants daily backups and a spare test instance:** 6 vCPU / 8GB at **$39.00/month**, which is $10 more than the 4GB plan for double the RAM. 👉 [See Spin Servers' current VPS plans and monthly prices](https://bit.ly/Spinservers)

## Spin Servers plans and prices

Spin Servers publishes monthly prices for its KVM VPS line, which is the shape most Minecraft groups actually need: root access, a fixed amount of RAM, and a bandwidth allowance. The company also advertises dedicated servers and colocation out of Dallas in its own public posts, but no monthly price for those appeared on the page I checked, so none of that is in the table.

| Plan | CPU / RAM | Storage / transfer | Price (USD) | Order |
| --- | --- | --- | --- | --- |
| KVM 2 vCPU 2GB | 2 vCPU, 2GB DDR4 | 40GB RAID-10 SSD, 4TB/mo, 1Gbps | $12.00/mo | [Check current price](https://bit.ly/Spinservers) |
| KVM 4 vCPU 4GB | 4 vCPU, 4GB DDR4 | 80GB RAID-10 SSD, 8TB/mo, 1Gbps | $29.00/mo | [Check current price](https://bit.ly/Spinservers) |
| KVM 6 vCPU 8GB | 6 vCPU, 8GB DDR4 | 120GB RAID-10 SSD, 8TB/mo, 1Gbps | $39.00/mo | [Check current price](https://bit.ly/Spinservers) |
| KVM 16 vCPU 16GB | 16 vCPU Xeon, 16GB DDR4 | 320GB RAID-10 SSD, 15TB/mo, 1Gbps | $74.00/mo | [Check current price](https://bit.ly/Spinservers) |

Everything above includes one IPv4 and one IPv6 address and a 1Gbps network port. Prices are listed as "starting from" and billed monthly in USD.

Three more configurations appear on the same page without a monthly figure I could confirm: 8 vCPU / 8GB with 160GB and 10TB, 12 vCPU / 12GB with 160GB and 10TB, and 32 vCPU / 32GB with 500GB and 20TB. I left their prices blank rather than estimate them.

One detail worth knowing if you plan to run anything besides the game: port 25 (SMTP) is closed by default on these VPS, and you have to open a ticket with a justification to get it. It has no effect on Minecraft traffic, but it tells you this is general-purpose infrastructure rather than a games-only product with a panel bolted on top.

## First, work out which server you are actually running

Search results for "Minecraft dedicated server" mix two products that share nothing except the name. Bedrock Dedicated Server is Microsoft's official software for Bedrock Edition: phones, consoles, Windows. Java Edition uses a separate server jar and a separate ecosystem of Paper, Purpur and mod loaders. The ports, the protocol and the permission files are all different.

| Edition | Protocol | Default ports | Operator file |
| --- | --- | --- | --- |
| Bedrock (BDS) | UDP | 19132 (IPv4), 19133 (IPv6) | `permissions.json` |
| Java | TCP | 25565 | `ops.json` |

A Bedrock server will not accept a Java client, and opening 25565 on a Bedrock box does nothing. If your friends play Java with mods, you need a Java server; if half the group is on Switch or Xbox, Bedrock is the only edition they can join at all.

Console players come with their own catch. BDS has no invite system comparable to Realms, so Xbox, PlayStation and Switch users reach a Bedrock server through a custom DNS entry or a LAN connection. That constraint alone decides the hosting question for some groups.

## What self-hosting really costs

A spare desktop is enough hardware. The recurring costs are electricity for a machine that runs 8,760 hours a year, a connection that survives an ISP address change, and your evenings.

Microsoft publishes no official minimum or recommended specification for BDS. The number most often quoted, 6GB of RAM, comes from a community wiki page whose figures were last revised in November 2021 and are explicitly labelled as assumptions based on watching resource use on Windows. That 6GB covers Windows plus the server. The 1GB figure that circulates for small servers refers to the server process alone, which is why the two numbers look contradictory and are not. Ubuntu Server without a desktop sits in the low hundreds of megabytes, so 2GB is a workable floor for a Bedrock world and 4GB leaves space for backups and a second instance.

Two hardware notes from the same wiki are more useful than the RAM figure. BDS performs badly on many-core server CPUs with low clock speeds, and well on high-clock desktop chips, so core count is the wrong thing to shop for. Under heavy load, with piles of entities or complex chunks loading, CPU utilisation can collapse while the whole server stalls, which looks like a network problem and is not.

The repetitive parts of running BDS on Linux have been scripted for years. A community installer sets the server up as a systemd service with start-on-boot, automatic updates, scheduled backups, a nightly restart and support for multiple instances. It is MIT licensed, the repository dates from February 2019, and its changelog entries stop in mid-2022, so treat it as stable rather than actively developed. The maintainer's warning is worth repeating: keep the default root path, because a path that was off by one directory once caused a user's server files to be pruned.

## Six things that break a server at 2 a.m.

**Protocol mismatch.** Clients and servers only talk when their protocol versions match, and the protocol changes with minor versions: 1.20.0 and 1.21.0 are incompatible. Patch releases are not safe either, since 1.21.0 and 1.21.30 also differ. A client on the wrong build gets "Outdated Client" or "Outdated Server", which is a version problem, not a firewall problem. Update the server and every client in the same session, and remember that a Preview server only accepts Preview clients.

**A capital letter in `level-name`.** Imported worlds sit in `%appdata%\Minecraft Bedrock\users\games\com.mojang\minecraftWorlds` under names like `eplC8tYRD04=`; the real name is inside `levelname.txt`. Copy the folder into `worlds/`, then set `level-name` to match the folder name exactly, including case, or the server starts a fresh world and your build is gone.

**Windows loopback.** Testing from the same PC that runs the server fails by default. Run `CheckNetIsolation LoopbackExempt -a -n="Microsoft.MinecraftUWP_8wekyb3d8bbwe"` in an administrator prompt. On Linux there is nothing to exempt; connect from another machine on the LAN.

**The player cap nobody mentions.** Connected players are capped at whichever is higher, 30 or your `max_players` value plus one. Setting `ignoresPlayerLimit` on an allowlist entry does not lift that ceiling. Plan around 30 as the practical maximum.

**No `/reload`.** BDS cannot reload settings the way a Java server can, and the command is deliberately absent to avoid crashes. Every change to `server.properties`, and every added pack, needs a restart. Allowlist edits made directly to `allowlist.json` need `/allowlist reload` before they apply.

**Old versions with a known exploit.** In 1.19.80 and earlier, a malicious user can log in as another player and grant themselves operator. Do not run those builds on a server anyone else can reach.

The wider picture: BDS is still labelled early-development software whose support "may be stopped at any time". It has no RCON, no WebSocket server, no way to run outside Windows and Ubuntu, a long list of item duplication bugs, and closed source code that makes third-party plugins expensive to build. None of that disqualifies it for a private world. All of it argues against betting a public community on it without backups.

## How many players fit on which plan

| Group | Plan | Why |
| --- | --- | --- |
| 1–4 friends, Bedrock survival | 2 vCPU / 2GB, $12/mo | The server process is small; the rest of the RAM goes to the OS |
| Up to ~10, a few packs, daily backups | 4 vCPU / 4GB, $29/mo | Backup archives and chunk-loading spikes need the headroom |
| 15–25, or Java with plugins and several worlds | 6 vCPU / 8GB, $39/mo | Best step in the lineup: $10 over the 4GB plan doubles the RAM |
| Public server, or a heavy Java modpack | 16 vCPU / 16GB, $74/mo | Capacity for parallel instances and modded Java, not for single-thread speed |

The 16 vCPU plan is not automatically faster at ticking a Bedrock world. BDS cares about single-core clock speed, so the extra cores buy you room to run several servers or a modded Java instance alongside, not a higher tick rate. A 32 vCPU machine for five people building a base is money spent on nothing.

If you are running Java rather than Bedrock, the memory guidance changes completely: 8GB minimum for vanilla and 16GB or more for modded or large player counts. That figure applies to Java and should not be carried back to BDS, which uses memory differently. 👉 [Check the 8GB plan's current price](https://bit.ly/Spinservers)

## When a managed game host is the better buy

A rented VPS gives you root and nothing else. You install BDS, you write the systemd unit, you notice the disk filling with backups. That is the trade for the price.

Managed Minecraft hosting sells the opposite. Host Havoc's cheapest tier, GRASS, is 1GB of RAM on NVMe storage for **£2.75/month** (GBP, Chicago), advertised for 1–4 vanilla players and explicitly not recommended for mods. The same provider includes DDoS protection, a control panel with scheduled backups and one-click modpack installs, 13 locations across North America, Europe and Oceania, sub-accounts and 2FA, and a 72-hour money-back window. Per gigabyte of RAM it undercuts a VPS, and someone else handles updates.

The catch is what that product is. It is a Java host: versions 1.7.10 through 1.21, Forge, Fabric, Paper, Spigot and vanilla, with Bedrock players joining through the Geyser plugin, which runs on Paper, Spigot, Purpur and Folia but not on a vanilla server. If you specifically want native Bedrock with BDS, a managed Java plan is not the thing, and a VPS is.

Rough rule: pick managed hosting when you want a panel and support more than control, and pick a VPS when you want Bedrock natively, root access, or the ability to run three servers on one box. 👉 [See what a root-access VPS costs at Spin Servers](https://bit.ly/Spinservers)

## Before you pay

Match the plan to the edition and the headcount first. Bedrock for a handful of friends runs fine on 2GB; anything with backups, plugins or a second world wants 8GB, and the $39 step is the one that makes financial sense. Skip the big core counts unless you are running multiple servers.

Two things to settle with support before you order, because they were not published where I could verify them: whether the "starting from" monthly price is the renewal price or a first-term rate, and what the cancellation and refund terms are. Both change the real first-year cost, and neither is guessable.

👉 [Check Spin Servers' current plans and confirm the terms before ordering](https://bit.ly/Spinservers)
