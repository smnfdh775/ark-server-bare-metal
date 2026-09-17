# best ark dedicated server hosting: real RAM and CPU requirements, cluster setups, and when bare metal beats slot plans

Everyone hunting for ARK server hosting hits the same fork in the road. One path is slot-based game hosting: you pay $12–$15 a month for a 30-slot server, click a button, and play. The other path is renting an actual dedicated server — a whole machine to yourself — and installing the ARK server software on it.

Most "best ARK hosting" articles only cover the first path. This one covers the second, because there's a large group of players for whom slot hosting stops making sense: people running multi-map clusters, heavily modded communities, ARK: Survival Ascended servers, or tribes that have outgrown per-slot pricing. If that's you, the questions you actually need answered are things like *how much RAM does an ASA map really use*, *can I run six maps on one box*, and *what does a capable bare-metal server cost*. Those are the questions this article answers, with real numbers and a full look at one provider that specializes in exactly this kind of hardware: Sharktech.

## What an ARK dedicated server actually needs

The official ARK wiki's dedicated server setup guide is the most reliable source here, and its numbers are more specific than most hosting marketing pages. The requirements split hard between the two games, and getting this wrong is the number one cause of laggy, swap-thrashing ARK servers.

**RAM is the big one, and it depends entirely on the map.** The wiki measured memory usage per map on a fresh save with no players connected:

| Map | ASE (empty server) | ASA (empty server) |
| --- | --- | --- |
| The Island | 3.5–4.5 GiB | 8–10 GiB |
| The Center | 3–4 GiB | 10.5–12 GiB |
| Scorched Earth | 3–4 GiB | 7.5–9 GiB |
| Aberration | 3–4 GiB | 8–10 GiB |
| Extinction | 3–4 GiB | 7–8.5 GiB |
| Ragnarok | 4–5 GiB | TBD |
| Fjordur | 4–5 GiB | unavailable |
| Crystal Isles | 5.5–6.5 GiB | unavailable |
| Genesis: Part 2 | 10–12.5 GiB | unavailable |

Two things jump out. First, ASA maps eat roughly two to three times the memory of their ASE counterparts. Second, that's the *floor* — a brand-new save with zero players. Every connected player adds an average of 50–150 MiB, and usage keeps climbing as the world ages: more tames, more structures, more stored items all consume memory. A mature modded server can sit well above those baseline figures.

**CPU is about clocks, not core count.** The wiki recommends 4 logical cores per ASA server instance and 2 per ASE instance, but with a crucial caveat: gameplay runs essentially on a single thread (physics aside), so single-thread performance beats raw core count. A 3.3 GHz Xeon will often feel better than a 2.0 GHz EPYC with four times the cores. This matters a lot when you're picking hardware, and we'll come back to it.

**Storage and network are the easy parts.** A full ASE server installation takes about 18 GiB of drive space; ASA takes around 11 GiB, plus room for saves, mods, and updates. Any NVMe drive handles that without breaking a sweat. On bandwidth, the wiki estimates each connected player needs up to about 60 KiB/s shared across upstream and downstream. Even 70 players simultaneously online works out to roughly 4 MB/s — call it 10 TB a month if your server never sleeps. A 10 Gbps port with a few hundred TB of monthly transfer is enormous overkill for ARK, which is exactly the position you want to be in.

**One OS constraint worth repeating in bold:** ASA requires Windows (or a Windows-like environment) — it does not support Linux natively. ASE runs on both Linux and Windows. If you're planning an ASA cluster, your dedicated box needs a Windows option at checkout.

## The ASA wrinkle: what the Nitrado deal means for self-hosters

Here's something a lot of hosting articles gloss over. Under the ARK: Survival Ascended end-user license agreement, as summarized in the official wiki, ASA dedicated servers may only be hosted for **personal, non-commercial** purposes unless you obtain a dedicated server license from Nitrado, which holds the commercial exclusivity deal for ASA hosting. The license carve-out covers donations that don't grant anything of value in exchange and revenue that doesn't exceed operational costs.

The practical translation:

- Running an ASA server on your own rented hardware for your friends and community: fine.
- Taking donations that cover server costs: fine.
- Selling reserved slots, VIP perks, or running ads for profit on ASA: not without a Nitrado license.

For ASE, there's no such restriction — commercial community servers are a normal, established thing. If you're planning a monetized server, that distinction alone might decide which game you host.

## Slot hosting vs. a dedicated server: where the math flips

Let's be honest about the cheap option first, because for a lot of players it genuinely wins. Budget slot hosts currently advertise 30-slot ASE servers in the $12–$15/month range. If you're a 10-person tribe wanting one Ragnarok server, that is unambiguously the better deal than $259/month of bare metal. Don't buy a dedicated server to host one small map.

The math flips in a few specific situations:

**ARK: Survival Ascended clusters.** Each ASA map starts at 8–12 GiB of RAM before mods and players. A three-map cluster (say The Island, The Center, and Scorched Earth) needs roughly 26–31 GiB empty, plus the Windows OS, plus player overhead, plus growth headroom. You're realistically shopping for 64 GB minimum, and 128 GB is where you stop worrying. On top of that, commercial ASA slot rentals only exist through one provider thanks to the exclusivity deal. Self-hosting on your own box is the main alternative, and it's the scenario where dedicated hardware makes the most sense.

**Large player counts.** Slot hosts charge per slot, and player caps are baked into the product. On your own server, `MaxPlayers` is a config value — your real ceiling is hardware, not a pricing tier. A 64 GB box running one heavily modded ASA map with 50–70 concurrent players is a different class of server than a 10-slot rental.

**Multi-map ASE clusters with full control.** Six ASE maps (Island, Ragnarok, Fjordur, The Center, Scorched Earth, Aberration) sum to roughly 21–27 GiB empty. A single 64 GB machine runs all six with cross-travel enabled, one OS to patch, one place for backups — versus paying six separate monthly server bills at a slot host and hoping their cluster tooling works. Depending on the host, six mid-tier map servers can approach or exceed dedicated pricing anyway, and you still don't get root.

**Anything else on the box.** A dedicated server is a computer. Teamspeak, a website, a Discord bot, a second game — it all runs on the same hardware you're already paying for.

The trade-off is work. On a dedicated box, *you* handle updates, backups, mod management, and crashes at 2 AM. Slot hosts automate most of that. If that sounds like a chore rather than a hobby, stay with slot hosting — there's no shame in paying someone else to babysit a game server.

## Setting up ARK on bare metal: the short version

The full walkthrough lives in the official wiki's dedicated server setup guide, but the shape of it is simple:

1. Install SteamCMD on the server.
2. Download the server files — app ID `376030` for ASE, `2430930` for ASA.
3. Create a launch script with your map, session name, ports, and admin password.
4. Open the right ports and start the server.

The ports are fixed and worth knowing by heart, because clusters run multiple instances with incremented ports:

| Instance | Game port (UDP) | Peer port (UDP) | Query port (UDP) | RCON (TCP) |
| --- | --- | --- | --- | --- |
| 1 | 7777 | 7778 | 27015 | 27020 |
| 2 | 7779 | 7780 | 27016 | 27021 |
| 3 | 7781 | 7782 | 27017 | 27022 |
| 4 | 9999 | 10000 | 37015 | 32330 |

A couple of details that trip people up: ASA uses `-port=` instead of `?port=` in the launch arguments, and ASA wants the admin password as the *last* argument or it swallows everything after it. Also, the server autosaves every 15 minutes by default — if it crashes before the first save, you lose everything, so configure backups of `ShooterGame/Saved` from the start.

You don't have to live in a terminal, either. Ark Server Manager covers ASE with a GUI, ASA Server Manager does the same for Ascended, and PowerShellGSM handles install/backup/update/restart automation for ASA. On a dedicated box, these tools do most of what a slot host's control panel does.

## Sharktech's dedicated server line: every current configuration

If you've decided bare metal is the way to go, Sharktech is a provider built around exactly this use case. They sell bare-metal dedicated servers (direct hardware-level access, not OS-level slices), with DDoS protection included on every service, a 99.99% uptime guarantee, a server management panel, 24/7 support, and five locations: Las Vegas, Los Angeles, Denver, Chicago, and Amsterdam. Their own customer page includes game server providers — one describes weathering 3–8 Gbit DDoS attacks without service interruption, which is the kind of thing that matters when your ARK server's IP is public.

Here is every configuration currently listed on their dedicated servers page, all with free setup and all on 10 Gbps ports with 300 TB/month of transfer (upgradable to 40 or 100 Gbps):

| Config | CPU | RAM | Storage | Network | Price | Order |
| --- | --- | --- | --- | --- | --- | --- |
| Dual Xeon E5-2695v4 | 36 × 2.1 GHz | 64 GB DDR4 (upgradable to 1 TB) | 2 TB M.2 NVMe + 6× 2.5" SATA/SAS bays | 10 Gbps, 300 TB/mo | $259/mo | [Order the 64 GB Dual E5-2695v4](https://portal.sharktech.net/cart.php?a=add&pid=741&aff=1611) |
| Dual Xeon E5-2695v4 | 36 × 2.1 GHz | 64 GB DDR4 (upgradable to 1 TB) | 2 TB M.2 NVMe + 6× 3.5" bays (HDD options to 16 TB) | 10 Gbps, 300 TB/mo | $269/mo | [Contact sales about this config](https://bit.ly/SharKTech) |
| Dual Xeon Gold 6248 | 40 × 2.5 GHz | 128 GB DDR4 | 2 TB M.2 NVMe + 3× 3.5" bays | 10 Gbps, 300 TB/mo | $299/mo | [Order the 128 GB Dual Gold 6248](https://portal.sharktech.net/cart.php?a=add&pid=660&aff=1611) |
| Dual Xeon Gold 6248 | 40 × 2.5 GHz | 128 GB DDR4 | 2 TB M.2 NVMe + 6× 2.5" bays | 10 Gbps, 300 TB/mo | $309/mo | [Order the 6-bay Gold 6248](https://portal.sharktech.net/cart.php?a=add&pid=636&aff=1611) |
| Dual Xeon Gold 6246 | 24 × 3.3 GHz | 128 GB DDR4 | 2 TB M.2 NVMe + 3× 3.5" bays | 10 Gbps, 300 TB/mo | $309/mo | [Order the 3.3 GHz Gold 6246](https://portal.sharktech.net/cart.php?a=add&pid=814&aff=1611) |
| Dual Xeon Gold 6248 | 40 × 2.5 GHz | 128 GB DDR4 | 2 TB M.2 NVMe + 6× U.2 bays | 10 Gbps, 300 TB/mo | $329/mo | [Order the U.2 NVMe config](https://portal.sharktech.net/cart.php?a=add&pid=766&aff=1611) |
| AMD EPYC 7702P | 64 × 2.0 GHz | 128 GB DDR4 | 2 TB M.2 NVMe + 10× U.2 bays | 10 Gbps, 300 TB/mo | $499/mo | [Order the EPYC 7702P](https://portal.sharktech.net/cart.php?a=add&pid=729&aff=1611) |
| Dual AMD EPYC 7702 | 128 × 2.0 GHz | 128 GB DDR4 | 2 TB M.2 NVMe + 10× U.2 bays | 10 Gbps, 300 TB/mo | $699/mo | [Contact sales about the dual EPYC](https://bit.ly/SharKTech) |

Monthly billing is the default, but every order page offers quarterly, semi-annual, and annual cycles at a discount — the $259 config drops to $2,641.80 billed annually, which works out to about $220/month, roughly 15% off. RAM upgrades (up to 1 TB on several configs), storage swaps, and a 100 Gbps DDoS protection upgrade are all selectable during checkout, and the OS choice is part of the order flow too — remember ASA needs Windows, so check the available options before committing. If none of the listed configs fit, Sharktech's sales team builds custom hardware on request.

## Matching the config to your ARK plans

Spec-based reasoning, using the requirements from the official wiki:

**The 64 GB Dual E5-2695v4 at $259/mo is the ASE cluster workhorse.** Six ASE maps need ~21–27 GiB empty; add Linux overhead, player memory, mod stacks, and years of world growth, and 64 GB still leaves breathing room. The 36 cores at 2.1 GHz are plenty for running many 2-core ASE instances side by side. The weaker single-thread clocks matter less for ASE than for ASA. If your community is ASE and wants The Island through Fjordur linked up with cross-travel, this is the value pick. 👉 [Check out the Dual E5-2695v4 plan here](https://portal.sharktech.net/cart.php?a=add&pid=741&aff=1611).

**The Dual Xeon Gold 6246 at $309/mo is the ASA pick.** ASA wants 4 logical cores per instance and favors single-thread speed — and at 3.3 GHz, the 6246 has the highest base clocks in this entire lineup. With 128 GB you can run a three-map ASA cluster (~26–31 GiB empty) plus Windows plus mods plus serious player counts without sweating. For a single big modded ASA map with 50+ players, it's even more comfortable.

**The 128 GB Gold 6248 configs ($299–$329/mo) are the balanced middle.** 2.5 GHz clocks are a step down from the 6246 for pure ASA grunt, but 40 cores handle mixed workloads well: an ASA map or two plus an ASE cluster plus your community's voice server and website on one box. Choose between them by storage layout — the $299 has 3.5" bays, the $309 adds more 2.5" bays, and the $329 brings six U.2 NVMe bays if you want serious all-NVMe capacity.

**The EPYC boxes are for people running a lot of everything.** 64 or 128 cores at 2.0 GHz are aimed at dense multi-instance hosting or heavy virtualization rather than peak ARK single-thread performance. If you're a game server operator running ASE maps by the dozen, that's their territory.

One honest caveat: these are previous-generation server platforms (Broadwell-era E5 v4, Cascade Lake Gold, Rome EPYC). That's part of why the prices are what they are, and for ARK's single-threaded, RAM-hungry workload it's a reasonable trade — but if your community also demands bleeding-edge tick rates for other games, factor the CPU generation into your decision. Sharktech does offer hardware upgrades and custom builds if you want something newer.

## Location, latency, and the stuff that decides ping

Sharktech's five data centers cover the US West (Las Vegas, Los Angeles), US Midwest (Denver, Chicago), and Europe (Amsterdam). For ARK, the rule is boring and effective: pick the location closest to the largest concentration of your players. A Chicago server gives most of the continental US playable ping; Amsterdam covers Europe; the West Coast locations serve US-Pacific and, thanks to Sharktech's heavy Asia-Pacific peering, work well for communities with China and Oceania members. A distributed international tribe may need to just accept that someone's at 150 ms and pick the location that minimizes complaints.

## Quick checklist before you order

- **Count your maps first.** Sum the empty-server RAM from the table above, add OS overhead, then add room for players (50–150 MiB each), mods, and a year of world growth. Buy the RAM that number points to, not the RAM that's cheapest.
- **ASE or ASA?** ASA means Windows and roughly 2–3× the RAM per map. It also means the personal/non-commercial license terms unless you go through Nitrado commercially.
- **Clocks over cores for ASA.** 3.3 GHz beats 2.0 GHz for ARK's single-threaded gameplay loop.
- **Plan your ports.** 7777/7778/27015 UDP plus 27020 TCP for instance one; increment for each additional map in the cluster.
- **Backups from day one.** Copy `ShooterGame/Saved` on a schedule; the 15-minute autosave won't save you from a crash in minute 14.
- **Verify the OS option at checkout** if you're going ASA.

And if you read all this and realized your actual need is one small ASE server for a handful of friends — Sharktech also runs a game server hosting line from $7.95/month that lists ARK among its supported games, 👉 [see their game server hosting here](https://bit.ly/SharKTech). Not every problem needs 64 GB of RAM. But if your ARK ambitions involve clusters, mods, Survival Ascended, or player counts that make slot-host invoices uncomfortable, 👉 [the dedicated server lineup and its current pricing are worth a look](https://bit.ly/SharKTech).

## FAQ

**Can I run ARK: Survival Ascended on a Linux dedicated server?** No. ASA requires Windows or a Windows-like environment and doesn't support Linux natively. ASE runs on both.

**How much RAM do I need for a modded ARK server?** Baseline is 8–12 GiB per ASA map or 3–5 GiB per ASE map on an empty save, then add for mods, players (50–150 MiB each), and world growth over time. Heavily modded mature servers commonly want double the empty-map figure.

**Can I run multiple ARK servers on one dedicated box?** Yes — the server software supports multiple instances on one host, each with its own incremented port set, which is exactly how cross-travel clusters are built.

**Is self-hosting ARK legal?** For ASE, yes, including commercially. For ASA, the EULA permits personal, non-commercial hosting (donations that cover costs are fine); commercial ASA hosting requires a Nitrado license.

**How much bandwidth does an ARK server use?** Up to about 60 KiB/s per connected player. Even a busy 70-player server lands around 10 TB/month — far below the 300 TB/month included with Sharktech's dedicated plans.
