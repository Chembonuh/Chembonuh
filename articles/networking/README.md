# Host Discovery in Modern Infrastructure: DNS, ARP, and Virtualized Environments

Host discovery is straightforward when DNS is accurate—and much less so when working in isolated labs, temporary virtual networks, or partially managed environments.

For infrastructure and security engineers, the key is understanding **which layer is providing visibility**.

## DNS: Best When the Environment Is Managed

In a well-managed environment, DNS provides the fastest path from an IP address to a meaningful hostname.

```bash
nslookup 192.168.1.20
dig -x 192.168.1.20
host 192.168.1.20
```

Forward lookups are equally useful when the hostname is known:

```bash
dig server01.example.local
nslookup server01.example.local
```

When reverse DNS records are maintained correctly, tools such as `nmap` can often display hostnames automatically:

```bash
nmap -sn 192.168.1.0/24
```

Example:

```text
Nmap scan report for win10.lab.local (192.168.1.25)
Host is up.
```

The limitation is simple: **DNS is only as useful as the records behind it**. Temporary VMs, lab systems, unmanaged devices, and stale records can make DNS incomplete or misleading.

## ARP: Reliable on the Local Layer 2 Segment

When DNS is unavailable, ARP is one of the most reliable ways to identify systems on the same broadcast domain.

```bash
sudo arp-scan --localnet
```

Typical output:

```text
192.168.1.10    52:54:00:aa:bb:cc
192.168.1.20    00:15:5d:11:22:33
192.168.1.30    08:00:27:44:55:66
```

This gives a direct **IP-to-MAC mapping**.

You can also inspect the local neighbor table:

```bash
ip neigh
```

ARP is especially useful in:

- KVM/libvirt labs
- Hyper-V environments
- isolated VLANs
- penetration-testing ranges
- temporary test networks
- environments without internal DNS

Its limitation is that ARP works only on the **local Layer 2 network**. It does not traverse routers.

## KVM / libvirt Visibility

On KVM systems, `virsh` can often provide VM names, MAC addresses, and sometimes IP addresses:

```bash
virsh list --all
virsh domiflist <vm-name>
virsh domifaddr <vm-name>
virsh net-dhcp-leases default
```

A useful workflow is:

```text
VM Name -> MAC Address -> ARP / DHCP Lease -> IP Address
```

This is often more reliable than expecting guest DNS registration.

## Hyper-V Visibility

On Hyper-V, PowerShell provides a similar view:

```powershell
Get-VM | Get-VMNetworkAdapter |
Select-Object VMName, MacAddress, IPAddresses
```

This gives a direct mapping between the VM name and its network identity when Hyper-V integration services can report the guest address.

## Practical Discovery Strategy

For advanced environments, use the strongest source available in this order:

1. **Hypervisor inventory** — know which VMs exist.
2. **DNS** — resolve managed systems by name.
3. **DHCP leases** — map MAC addresses to assigned IPs.
4. **ARP / neighbor tables** — validate local presence.
5. **Nmap host discovery** — identify responsive hosts across the subnet.

Example:

```bash
sudo nmap -sn 192.168.1.0/24
sudo arp-scan --localnet
ip neigh
```

Then correlate the results with the hypervisor.

## Key Takeaway

In managed enterprise environments, DNS usually gives the cleanest view of host identity.

In labs and virtualized environments, **MAC addresses are often the more reliable source of truth**.

Understanding how DNS, DHCP, ARP, and the hypervisor relate to each other makes host discovery faster, more accurate, and easier to troubleshoot.

---

**Useful mental model**

```text
Hostname
   |
   v
DNS
   |
   v
IP Address
   |
   v
ARP / Neighbor Table
   |
   v
MAC Address
   |
   v
Virtual NIC / Hypervisor
```
