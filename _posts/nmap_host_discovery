---
layout: post
title: Nmap Host Detection
---

Originally posted on Medium.com

**Before Scanning Ports, Find Out Who’s There: Nmap Live Host Discovery**
*A practical companion walkthrough for understanding the commands, reading the responses, and avoiding assumptions.*

Learning cybersecurity means getting comfortable with tools. It also means knowing what their results actually prove.

That is how I'm approaching [TryHackMe's Nmap Live Host Discovery room](https://tryhackme.com/room/nmap01). Before looking for services on a device, we need to establish whether we can a) find it and b)get a response from it. And when we cannot, we need to resist turning "no response" into "nothing is there."

This walkthrough explains the room's core concepts and gives you a process to follow alongside the exercises. Use your own lab results for the interactive questions; device counts and responses depend on the scenario.

The purpose of this room is to show you options for host discovery: finding out what is reachable on the network. Knowing your OSI layers will help you understand what scans work with what protocols, and therefore what they will be able to reach. It gives tips on certain behaviors you might encounter. From ARP at layer 2 (data link), IMCP at layer 3 and finally to TCP/UDP options at layer 4 (network), this room covers nmap scans that will help you with host discovery before moving on to enumeration and port scanning.

Get the lab ready

Open the room and follow its instructions for starting the required environment. Use the AttackBox or your connected lab machine for terminal commands, and the room's interactive exercises where directed.

The examples below use a Bash variable:\
TARGET="10.10.10.10"

Replace that example address with the target assigned to you. Do not type 'MACHINE_IP' literally. Run subnet discovery only against the range the exercise authorizes; adding '/24' expands a single target into 256 addresses.

Check that Nmap is available:\
nmap -version

Their examples use 'sudo' where raw packet privileges matter. If your AttackBox terminal already runs as root, you can omit it.

1\. Understand what host discovery is asking.

Host discovery asks: Which target addresses respond to these probes?

Port scanning asks a different question: What is the state of the ports on those targets?

Nmap normally performs discovery before scanning ports.

A default nmap scan, when run without sudo, uses TCP Connect Scan (-sT) as the default scanning method, completing the whole TCP handshake. With sudo, it runs the SYN Scan (-sS), sending only SYN packets and waits for a response.

For discovery without the subsequent port scan, use:\
sudo nmap -sn $TARGET

The '-sn' option disables port scanning. It does not mean Nmap sends only a traditional ICMP ping. The probes depend on privileges and whether the destination is local or routed.

**My memory cue:** '-sn' stops the port scan. It does not choose the discovery protocol.

2\. Subnetworks: where can an ARP request go?

ARP connects an IPv4 address to a MAC address on the local network link. *An ARP broadcast stays within its broadcast domain; routers do not forward it to another subnet.*

When following the room's network diagram, ask yourself: Which devices share the sender's local segment? Which sit beyond a router?

For each question, follow this process:

1\. Identify the device sending the request.

2\. Follow the broadcast across its local segment.

3\. Stop at the router boundary.

4\. Separate devices that receive the request from the device that replies.

Receiving a broadcast and answering it are different events. Count what the exercise shows rather than assuming every computer in the diagram receives every packet.

For a remote destination, your machine resolves the local next hop's MAC address. It does not learn the remote host's MAC address through a router.

*Note --- a broadcast is indicated by the sender and receiver of the packet being the same in this room.*

3\. Enumerate the targets before probing them.

Nmap accepts individual addresses, ranges, CIDR networks, and files containing targets.\
# Two individual addresses\
nmap -sL -n 10.10.10.10 10.10.10.11\
# An inclusive range: six addresses\
nmap -sL -n 10.10.10.10--15\
# A CIDR block\
nmap -sL -n 10.10.12.13/29\
# Addresses stored in a file\
nmap -sL -n -iL targets.txt

These are list scans.

'-sL' previews the target list without probing the target hosts; '-n' also disables name resolution through the DNS server.

The '/29' example contains eight addresses: '10.10.12.8' through '10.10.12.15'. The address before the slash is inside the block, rather than necessarily being its first address.

The calculation is:

32--29 = 3 host bits

2³ = 8 addresses

Nmap enumerates all eight addresses, including the conventional network and broadcast addresses. "Addresses in the block" and "normally usable host addresses" are different counts.

Previewing the range is a useful habit. A command can be syntactically correct and still target the wrong systems.

4\. Discover local hosts with ARP.

For an authorized local IPv4 range, the command pattern is:\
sudo nmap -sn -PR -n LOCAL_LAB_CIDR

Replace 'LOCAL_LAB_CIDR' with the exercise's actual local range.

Here, '-PR' selects ARP discovery. Nmap generally uses it automatically for privileged scans on a local Ethernet network.

If you are connected through a routed VPN, do not assume the lab target shares your local link. The route matters more than two addresses looking similar.

The room also introduces 'arp-scan'. Its local discovery purpose follows the same boundary: ARP does not become a remote discovery technique because you use another tool.

> When no host discovery options are provided, Nmap follows the following approaches to discover live hosts:
>
> When a privileged user tries to scan targets on a local network (Ethernet), Nmap uses ARP requests. A privileged user is `root` or a user who belongs to `sudoers` and can run `sudo`.
>
> When a privileged user tries to scan targets outside the local network, Nmap uses ICMP echo requests, TCP ACK (Acknowledge) to port 80, TCP SYN (Synchronise) to port 443, and ICMP timestamp requests.
>
> When an unprivileged user tries to scan targets outside the local network, Nmap resorts to a TCP 3-way handshake by sending SYN packets to ports 80 and 443.
>
> Nmap, by default, uses a ping scan to find live hosts, then proceeds to scan live hosts only. If you want to use Nmap to discover online hosts without port-scanning the live systems, you can issue `nmap -sn TARGETS`. Let's dig deeper to gain a solid understanding of the different techniques used.

5\. Compare ICMP discovery methods.

Try each probe against your assigned target:\
sudo nmap -sn -PE -n $TARGET\
sudo nmap -sn -PP -n $TARGET\
sudo nmap -sn -PM -n $TARGET

'-PE' sends an echo request; '-PP' requests a timestamp; '-PM' requests an address mask. Echo uses ICMP request type 8 and reply type 0.

[](https://medium.com/download-app?source=promotion_paragraph---post_body_banner_better_screentime--e70a027e5e5e-----------------------------------------)

A response supports discovery. Silence does not establish that the host is offline: filtering or unsupported requests can explain it.

On local Ethernet, Nmap may still use ARP even when you select these probes. For an experiment that needs IP-level discovery rather than implicit ARP discovery, add the flag "- -disable-arp-ping". Link-layer address resolution may still be necessary to deliver the packets.

6\. Compare TCP and UDP discovery\
sudo nmap -sn -PS22,80,443 -n $TARGET\
sudo nmap -sn -PA80,443 -n $TARGET\
sudo nmap -sn -PU40125 -n $TARGET

SYN discovery ('-PS') can recognize either SYN/ACK or RST replies. A closed port can therefore still reveal a live host.

ACK discovery ('-PA') looks for a response to an unsolicited ACK, commonly RST. Stateful filtering may drop it.

UDP discovery ('-PU') can receive an ICMP port-unreachable response from a closed port. A service response can also reveal the host; silence remains ambiguous.

Keep the port list attached to the option: '-PS22,80,443'.

These discovery options differ from port scan options such as '-sS' and '-sU'. Keep '-sn' when your goal is discovery alone without port scanning.

The room also introduces Masscan. Keep the focus here on interpreting responses; speed does not fix a misunderstanding about what you are measuring.

7\. Use reverse DNS for context.

Reverse DNS attempts to find a hostname associated with an IP address.\
# Disable name resolution\
sudo nmap -sn -n $TARGET\
# Request reverse lookups for every target address\
nmap -sL -R 10.10.12.13/29

Normally, Nmap performs reverse lookups for responsive hosts; list scans look up the listed addresses. '-R' requests reverse lookups for all target addresses, and '-n' disables resolution. You can select a resolver with "- -dns-servers".

A returned name may suggest a system's purpose, but names can be stale or misleading. DNS is context, not proof of a running service or an active host.

8\. Ask why Nmap reached its conclusion.

My preferred command for reviewing a discovery result is:\
sudo nmap -sn -n -reason $TARGET

`'-` reason' shows the response behind the reported state. For more detail, add '-packet-trace' to inspect sent and received packets. Or to understand exactly what nmap did, send add the flag -vv for extra verbosity.

Record the probe, response, and conclusion. If two methods disagree, compare the evidence and consider the route, privileges, and filtering before deciding what happened.

One final distinction: '-Pn' skips discovery and proceeds as though the targets are up. It does not confirm that they responded. Using it with a normal port scan may help when discovery is blocked, but that is a different step from this room's discovery exercises.

Check out <https://nmap.org/book/host-discovery-strategies.html> for more.

What I want to retain

The command flags are useful. The reasoning is what makes them useful.

Before starting a scan, I want to refine the command to suit the situation depending on what I already know (or don't) about the target(s). Doing some footprinting before host discovery may help with understanding the network layout and give me an idea on what scans to start with to avoid making too much noise.

Below is a quick summary of the Nmap command-line options for host discovery that were covered in this room.

Scan Type

Example Command

ARP Scan

sudo nmap -PR -sn 10.200.6.0/24

ICMP Echo Scan

sudo nmap -PE -sn 10.200.6.0/24

ICMP Timestamp Scan

sudo nmap -PP -sn 10.200.6.0/24

ICMP Address Mask Scan

sudo nmap -PM -sn 10.200.6.0/24

TCP SYN Ping Scan

sudo nmap -PS22,80,443 -sn 10.200.6.0/30

TCP ACK Ping Scan

sudo nmap -PA22,80,443 -sn 10.200.6.0/30

UDP Ping Scan

sudo nmap -PU53,161,162 -sn 10.200.6.0/30

Remember to add -sn if you are only interested in host discovery without port-scanning. Omitting -sn will let Nmap default to scanning live hosts for ports

Option

Purpose

-n

Skip DNS

-R

Reverse DNS for all hosts

-sn

Discover hosts only

References

- [TryHackMe: Nmap Live Host Discovery](https://tryhackme.com/room/nmap01)

- [Nmap: Specifying Targets](https://nmap.org/book/host-discovery-specify-targets.html)

- [Nmap: Host Discovery Controls](https://nmap.org/book/host-discovery-controls.html)

- [Nmap: Host Discovery Techniques](https://nmap.org/book/host-discovery-techniques.html)

- [Nmap: DNS Resolution](https://nmap.org/book/host-discovery-dns.html)

- [Nmap: Host Discovery Reference](https://nmap.org/book/man-host-discovery.html)

- [Nmap: Host Discovery Strategies](https://nmap.org/book/host-discovery-strategies.html)

-[Subnet Cheat Sheet](<https://www.freecodecamp.org/news/subnet-cheat-sheet-24-subnet-mask-30-26-27-29-and-other-ip-address-cidr-network-references/>)
