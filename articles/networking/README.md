# Host Discovery in Virtualized Environments: ARP, Hypervisor Data, and Local Network Mapping

Host discovery in isolated labs and virtualized environments often comes down to correlating information from the hypervisor with what is visible on the local Layer 2 network.

For infrastructure and security engineers, the goal is simple: identify the relationship between a VM name, its virtual NIC, its MAC address, and the IP address currently in use.

## ARP: Reliable on the Local Layer 2 Segment

ARP is one of the most useful ways to identify systems on the same broadcast domain.

```bash
sudo arp-scan --localnet
```

Typical output:

```text
192.168.1.10    52:54:00:aa:bb:cc
192.168.1.20    00:15:5d:11:22:33
192.168.1.30    08:00:27:44:55:66
```

This provides a direct **IP-to-MAC mapping** for hosts that respond on the local network.

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
- environments where host inventory is incomplete

Its main limitation is that ARP works only on the **local Layer 2 network**. It does not traverse routers.

## KVM / libvirt Visibility

On KVM systems, `virsh` can provide VM names, MAC addresses, and sometimes IP addresses:

```bash
virsh list --all
virsh domiflist <vm-name>
virsh domifaddr <vm-name>
virsh net-dhcp-leases default
```

A useful correlation workflow is:

```text
VM Name -> Virtual NIC -> MAC Address -> DHCP/ARP -> IP Address
```

For example, `virsh domiflist` identifies the VM's virtual NIC and MAC address, while `virsh net-dhcp-leases default` may show the IP address associated with that MAC.

## Hyper-V Visibility

On Hyper-V, PowerShell provides a similar view:

```powershell
Get-VM | Get-VMNetworkAdapter |
Select-Object VMName, MacAddress, IPAddresses
```

This can provide a direct mapping between the VM name and its network identity when Hyper-V integration services are reporting guest addressing information.

## Nmap Host Discovery

For a quick view of responsive hosts across a subnet:

```bash
sudo nmap -sn 192.168.1.0/24
```

This performs host discovery without a full port scan.

A practical combination is:

```bash
sudo nmap -sn 192.168.1.0/24
sudo arp-scan --localnet
ip neigh
```

Then compare the IP and MAC information with the hypervisor inventory.

## Practical Discovery Strategy

For a virtualized lab, a useful workflow is:

1. **Hypervisor inventory** — identify which VMs exist.
2. **Virtual NIC mapping** — collect each VM's MAC address.
3. **DHCP leases** — map MAC addresses to assigned IP addresses when available.
4. **ARP / neighbor tables** — validate which systems are present locally.
5. **Nmap host discovery** — confirm responsive hosts across the subnet.

This method works well because each layer provides a different piece of the same identity chain.

## Key Takeaway

In virtualized environments, **MAC addresses are often the most reliable point of correlation** between the hypervisor and the network.

When guest IP reporting is incomplete, combining hypervisor data with DHCP leases, ARP results, and neighbor tables provides a fast and repeatable way to identify systems.

---

**Useful mental model**

```text
VM Name
   |
   v
Virtual NIC
   |
   v
MAC Address
   |
   +----> DHCP Lease
   |
   +----> ARP / Neighbor Table
               |
               v
          IP Address
```
