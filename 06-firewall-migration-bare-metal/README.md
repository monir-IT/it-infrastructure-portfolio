# Firewall Migration: Hyper-V Virtual Machine to Bare-Metal Appliance

Moved the company's production OPNsense firewall off a Windows workstation and onto a dedicated fanless appliance, with a tested rollback.

| | |
| --- | --- |
| **Role** | Sole IT engineer: hardware selection, build, configuration restore, cutover planning |
| **From** | OPNsense VM on Hyper-V, Windows 11 Pro workstation, Intel i350-T4 network card |
| **To** | CWWK N150 fanless appliance, 4 × Intel i226-V ports, 8 GB RAM, 128 GB NVMe |
| **Related** | [OPNsense firewall](../05-opnsense-firewall/README.md) |
| **Status** | Cut over October 2026 and in service; old VM removed |

## Stage one: OPNsense on Hyper-V

To get the network off eight consumer routers quickly and with hardware the company already had, I first ran OPNsense as a virtual machine on a well-specified office PC.

1. Installed an Intel i350 four-port network card and enabled Hyper-V.
2. Disabled automatic Windows updates through the registry, so installs and restarts need my approval.
3. Set the BIOS to power on automatically after a power cut, and tested it.
4. Connected the internet router to network port 1 and **ran the PC for a week** to confirm it stayed up before putting anything critical on it.
5. On the next day off, configured OPNsense, connected port 2 to the core switch and brought up the VLANs.

The VM's network adapter needed an explicit VLAN trunk:

```powershell
Set-VMNetworkAdapterVlan -VMName "<firewall VM name>" -Trunk `
  -AllowedVlanIdList "10,20,30,40,50,99,150" -NativeVlanId 0
```

## Why move off it

The workstation was also a desktop and ran other services. Between the business and the internet sat four layers (BIOS, Windows, Windows Update and the hypervisor's autostart), and a failure in any of them meant the office was offline until someone came in. A firewall should be one boring box that does one job.

## Choosing the hardware

The deciding specification was the network chip. The earlier Intel i225 generation has a known history of links dropping and renegotiating, and an intermittent link looks exactly like the faults this network had already taken weeks to chase. I rejected a cheaper N5105 unit with i225 ports and DDR3 for that reason and chose an N150 unit with i226-V ports.

## The build

| Step | What happened | What I did |
| --- | --- | --- |
| First power-up | Power light on, but no video, no keyboard response, no reaction to the power button | Unplugged the adapter for a minute and reconnected. A stuck power state, not a fault |
| Installer | Several OPNsense images to choose from | Used the **VGA** image (the USB installer with monitor output) and wrote it with Rufus, which handles the compressed image and switches to DD mode |
| Install wizard | Offered to set up LAGG and VLANs | Declined both: the configuration restore brings the VLANs, and building them by hand first only creates conflicts |
| Interfaces | | Assigned port 0 as LAN and port 1 as WAN |

## Restoring the configuration, and the lockout

**Interface mismatch.** The backup referred to Hyper-V's virtual interface names, which don't exist on real hardware. Remapping LAN and WAN to the appliance's ports in Interfaces → Assignments brought all seven VLANs across intact.

**Locked out of the web interface.** The restore also brought the user database and the two-factor authentication server. The password set during installation no longer applied, and the web interface was checking codes against a clock the new appliance had not yet synchronised.

- Booting into single-user mode (`boot -s` at the loader) and using the console reset the root password, but the web interface still refused because it was not using the local database.
- I chose **not** to edit `config.xml` by hand under pressure.
- **Clean fix:** on the production firewall, temporarily switch web authentication to the local database, take a fresh backup, restore that onto the appliance, then restore two-factor once settled.

## Testing recovery

- **Auto power-on after a power cut:** set in the BIOS and tested, as on the old PC: pulled the power cable, waited ten seconds, reconnected, hands off. It came back by itself.
- **Temperature:** about 50 °C at idle, normal for a passively cooled N150.
- Tailscale and FreeRADIUS installed on the appliance, so remote access no longer depends on the workstation.

## Cutover

The swap is two cables (WAN and the switch trunk) moved from the old card to the appliance, then power on. Subnets, gateways and DHCP ranges are identical, so devices keep working; the DHCP lease table starts empty and relearns devices as they renew.

**Rollback:** move the two cables back and start the VM. The old workstation stayed untouched until the appliance had proved itself; then I removed the VM and its virtual switches, and the workstation went back to being an ordinary PC.

**One thing changes at cutover:** the appliance's WAN port has a different MAC address, so the 5G router gives it a different address, which breaks the router's DMZ entry and all external camera viewing. Post-cutover checklist:

1. Confirm the firewall's new WAN address.
2. Update the 5G router's DMZ entry to the new address.
3. Check a handset re-registers to the phone system.
4. Check one recorder in Hik-Connect from outside.
5. Confirm a client on each SSID gets an address in the right VLAN.
6. Remove the subnet routes from the old workstation's Tailscale so two nodes don't advertise the same networks.

**Still open:** reserve the appliance's WAN address on the 5G router so it can't drift again (it has broken camera viewing twice), and switch two-factor login back on for the web interface.

## Lessons learned

- Start with what you have, prove the design, then move it onto proper hardware.
- Choose hardware by the component that has caused you problems before.
- Restore the configuration rather than rebuilding by hand, and remap interfaces after.
- A restored configuration brings its authentication with it. Plan how you will log in before you restore.
- Test the recovery path. "It should power back on" is not the same as watching it do so.
- Always keep a rollback that is one cable move away.

## Skills demonstrated

`OPNsense` · `Hyper-V` · `PowerShell` · `Hardware selection` · `Configuration backup and restore` · `Change and rollback planning` · `Recovery testing`
