# How I Built It: Eight Routers to One Firewall

The story behind the projects in this repository: turning eight unmanaged routers and a flat network into a segmented, firewalled, remotely managed infrastructure. I did it alone, in a working office, mostly on Fridays. It covers what I built, what broke, how I tracked each fault down, and what I would tell anyone starting the same job.

## What I inherited

Eight routers, and none of them talking to another.

The building houses three restaurant brands across three floors, and the network had grown the way networks grow when nobody owns them: each problem had been solved by adding a router. The cashier table had one with an unlimited 5G SIM. The ground-floor operations manager had another, because the cashier's signal didn't reach. The coffee counter had a third. The first-floor server cabinet had a fourth for the central kitchen. The CEO had his own. The office had its own. There was a separate one just for the camera recorder, added because the office router had hit its client limit.

None of them shared addressing or could see each other. There was no segmentation, so a POS terminal, a camera, a phone and a guest's laptop all sat in the same broadcast domain. There was no central visibility: no logs anyone read, no inventory, and no way to answer "what is on my network right now?" except by walking around and looking.

The cabling told the same story. POS printers at the coffee counter were connected with cables run across the floor, and staff tripped over them. Forty cameras hung off two unmanaged switches daisy-chained to each other and to a third in the kitchen.

Nobody was at fault. This is what a growing restaurant business looks like before it has an IT function. But it meant my first real job was not building anything. It was finding out what was already there.

## Phase One: nine phones

The first project was deliberately small. People were walking between departments to ask each other questions, or using personal mobiles. They wanted internal calling, nothing more.

I chose Grandstream after comparing brands on interface quality and support, and picked the GRP2612W because it is Wi-Fi. The office had no structured cabling and no realistic path to run any, so a wired handset would have meant a cabling project before the phone project. Finding an official distributor took more research than choosing the hardware.

Nine phones (13 today), one phone system, and a spare router repurposed as a voice network with no internet. It cost very little and it worked.

The lesson came in commissioning. On DHCP, the phones kept changing address, and every time one did, its SIP registration broke. Moving every phone to a static address ended it. That was the first time I understood that for an infrastructure device, a predictable address is not a nicety; it is part of whether the service works.

→ [VoIP project](01-grandstream-voip/README.md)

## Phase Two: doors, cameras and the first VLANs

With the phones working, two gaps were obvious: anyone could walk into the office, and nothing was being recorded.

Phase Two brought a video door station with a magnetic lock, an indoor station so staff could see and speak to visitors without going downstairs, a recorder with four cameras, a second switch, an access point for the CEO area and one more phone. It also brought the first real structure: two cabinets, a new cable run between them, and that run configured as a trunk carrying VLAN 20 for voice and VLAN 30 for cameras. Two VLANs is not segmentation, but it was where the whole later design started.

The physical work was the hard part, and it is the part nobody warns you about. Getting power to a magnetic lock, running low-voltage wiring, fitting the exit release and choosing cable that wouldn't cause trouble later took rounds of trial and error, and none of it is networking.

Two things from that phase still shape how I work. The cable to the entrance was first run on the surface; management then asked for it inside the wall, so the whole run was done again. **Get the cable path approved before you terminate anything.** And the door keypad appeared completely broken in testing. The device was fine: the entry sequence is star, PIN, hash, and I had been entering the PIN alone. **Suspect your own procedure before the hardware.**

→ [Access control](02-access-control-door-entry/README.md) · [CCTV](03-cctv-surveillance/README.md)

## Phase Three: a firewall in the middle of a live business

This was the one that changed everything, and the one that could have gone badly.

The plan was to put OPNsense between the office and the internet, with proper VLANs behind it. The budget was a workstation I already had, so OPNsense went on Hyper-V with an Intel four-port network card. That was a compromise I would later undo, but it was the difference between doing the project and not doing it.

I didn't rush it. On a Friday with the office closed, I built the machine: Hyper-V enabled, Windows auto-update disabled, BIOS set to power on after a power cut, and that tested. Then I left it running for a week to see whether it stayed up before trusting it with anything. The following Friday I connected it to the core switch and built the VLANs: management, office, voice, cameras, employee, business, and later the CEO room.

### What I learned building it

**A VLAN has to exist at every hop or it silently does nothing.** The hypervisor's trunk list, the firewall interface, its DHCP range, its rules, the tags on the switch ports upstream and the access-point port downstream. Miss one and the device connects, waits and gets no address, with no error anywhere. I made this exact mistake three times, with VLANs 40, 50 and 150.

**Firewall rules are evaluated top to bottom, and the first match wins.** I once moved a broad "pass to any" rule to the top of a tab to test something, and every rule beneath it stopped existing.

**Block internal, then pass everything else.** Blocking a VLAN's traffic to the private address ranges, then passing the rest, gives it internet and nothing internal. The block checks the destination, so internet traffic never matches it. Understanding that one mechanism made the whole policy possible.

**A vendor's words are not the standard's words.** Hikvision's TRUNK, Grandstream's Hybrid and Ruijie's Trunk are three spellings of nearly the same idea.

→ [Switching and VLANs](04-switching-and-vlans/README.md) · [OPNsense firewall](05-opnsense-firewall/README.md)

## The faults that took weeks

### The phones that connected to nothing

Every handset would join the Wi-Fi and then report an IP configuration failure. The SSID had been given a tagged VLAN 20, on access-point ports whose untagged VLAN was already 20. A tagged frame arriving at a port expecting untagged traffic is discarded without a word. Setting the SSID's VLAN back to blank fixed it instantly. That silent drop is the most common failure in this whole build, and it always looks the same: connects fine, never gets an address.

### Printing that worked over Wi-Fi but not from my own desk

A laptop on the office Wi-Fi could print. My workstation on the management VLAN couldn't, but it could ping the printer perfectly.

Ping succeeding is what cracked it. The route was fine. The print job left my PC, but the printer's reply arrived on the office interface, where the rule blocking office traffic to private networks caught it. Firewall rules are directional, and I had only thought about one direction.

My first fix had source and destination reversed. The second worked but was too broad: it would have let any office device reach my admin machine. I caught that by asking what the rule actually permitted rather than whether printing worked. The final rule lets only the printer reply to admin devices. Then printing still failed, until Windows had the printer removed and re-added as an IPP device. The network and the client both have to be right.

### The cameras, which took the longest

This ran for weeks, and I was wrong about it several times.

From mobile data, the seven branch recorders worked and the two head-office ones didn't. From the office Wi-Fi, it was exactly the reverse. Something was breaking both directions at once.

I chased the firewall first: CPU load 0.26, memory under 20%, so not overload. Disabling state tracking on the camera rule changed nothing. The recorders' DNS pointed at a public resolver on a VLAN that only allows the firewall's DNS, which was genuinely wrong, but fixing it didn't fix the problem.

I suspected carrier-grade NAT, since the business has no public IP anywhere. Then I realised all nine sites sit behind the same carrier NAT, and seven work perfectly. That one observation killed the theory and reframed the question: what does head office have that the branches don't? Only one thing: OPNsense in the path.

OPNsense randomises source ports on outbound NAT by default, and Hik-Connect's peer-to-peer hole-punching can't survive that in either direction. One mechanism, both symptoms. The fix was Hybrid outbound NAT with a static-port rule per VLAN, plus Conservative optimisation so idle UDP mappings don't expire mid-stream. Adding the rule for the camera VLAN alone fixed exactly one direction, which confirmed the diagnosis.

## Getting the firewall onto its own hardware

The Hyper-V setup worked, but four things had to go right for the office to have internet: the BIOS, Windows, Windows Update and the VM's autostart. The same machine was also a desktop and ran other services. One box doing everything is one box whose failure stops everything.

I chose a fanless appliance with Intel i226-V ports. The network chip was the deciding factor: the earlier i225 generation has a known history of links dropping, and after weeks of chasing intermittent faults, hardware that produces them was the last thing I wanted.

It didn't go smoothly. The appliance wouldn't start at all until I unplugged it for a minute. Then the configuration restore brought everything across, including the two-factor authentication server, and the new box's clock hadn't synchronised. I was locked out of a firewall sitting on my own desk. The clean way out was to switch the production firewall to local authentication, take a fresh backup and restore that. I could have edited the configuration file by hand under pressure. I'm glad I didn't.

Then I set the BIOS to power on after a power cut and tested it again on the new box: pulled the cable, waited, plugged it back in, hands off. It came up by itself. It takes a minute, and it is the difference between knowing the office comes back at 3 a.m. and hoping it does.

→ [Firewall migration](06-firewall-migration-bare-metal/README.md)

## Where it stands

One firewall, seven routed VLANs, five managed switches, managed Wi-Fi, two camera recorders, a phone system and door entry, all under one policy and visible from one place.

Voice has no internet and doesn't need it. Cameras reach their cloud and nothing internal. Office staff get internet and no internal networks except the printer. Only named admin devices can reach the management network. I can reach all of it from home through Tailscale, which runs on the firewall itself. FreeRADIUS is configured and ready for per-person Wi-Fi logins; the last step is replacing two older access points that can't do it.

The eight routers are down to one, and that one does nothing but carry the 5G uplink to the firewall.

Still to do: the physical cutover to the new appliance, off-box configuration backups, a UPS, reserved addresses for the devices other configuration points at, and per-person Wi-Fi logins. None of it is urgent. All of it is the difference between a network that works and a network that keeps working without me.

## The hardest part

It wasn't the technology.

The camera fault was the worst technically: weeks of it, several confident wrong theories, and a cause hidden in a default setting nobody thinks to question. But it was solvable, and patience plus one good question solved it.

The hard part was doing all of this to a live business, alone, with no second opinion and no rollback except my own memory. If I broke voice, nobody could call the kitchen. If I broke the cameras, nothing was recording. If I locked myself out of the firewall, the office lost internet and only I could fix it. There was no colleague to check my rule order before I pressed Apply, and no test environment: the test environment was the office.

That pressure pushes you to make several changes at once to save a trip, which is exactly how you end up unable to tell which one broke things. It makes you put off the boring tasks, like backups, documentation and address reservations, because nothing is visibly wrong today.

The second hardest thing was writing nothing down for too long. I carried the whole topology in my head, until the day the access points changed address after a power cut and I couldn't tell what had moved.

## What I'd tell someone doing this alone

1. **Document as you build, not after.** A switch-port table with device, VLAN, tagged or untagged, address and location is most of the value.
2. **Change one thing, then test.** It feels slower. It is dramatically faster.
3. **Know your way back in before you touch management.** Console cable, keyboard, spare port, spare address: decide the recovery before the change.
4. **Test the recovery path; don't assume it.** An untested assumption is a hope with a configuration page.
5. **Reserve the address of anything other configuration points at.** Aliases, DMZ targets and cloud records all break silently when an address drifts.
6. **Isolate variables; don't collect theories.** Ask what the failing thing has that the working ones don't.
7. **Check capacity before configuration.** My upload swings between 1.6 and 11.6 Mbps through the day. Plenty of "faults" were a full pipe.
8. **Don't let one machine do everything.** Separate what must not fail from what is merely useful.
9. **Judge a rule by what it permits, not by whether the task works.** Working and correct are different tests, and only one of them is about security.
10. **Suspect your own procedure before the hardware.**
11. **Expect the physical work to take longer than the configuration.** The design fits in your head; the building fights you.
12. **Keep a record of the problems, not just the solutions.** The configuration shows what the network is. The fault log shows why it is that way, and nobody else can reconstruct that.
