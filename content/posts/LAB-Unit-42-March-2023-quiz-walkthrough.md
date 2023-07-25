---
title: "LAB Unit 42 March 2023 Quiz Walkthrough"
date: 2023-07-23T18:20:15Z
draft: false
---
# Introduction

In this blog post, I will be reviewing the Unit42 PCAP quiz. Palo Alto has provided the quiz, and some relevant IOC information at the following links:

[Link to the Unit42 Quiz overview](https://unit42.paloaltonetworks.com/march-wireshark-gozi/)

[Link to the IOC report they provide](https://github.com/pan-unit42/tweets/blob/master/2023-03-06-IOCs-for-Gozi-infection.txt)

A rule of engagement, knowing answers are already available: I will not look at the answers until my analysis is complete. This blog post is mostly to highlight my own practice with this (Been a while since I've had to do PCAP analysis)

# Finding the Source

Looking at the [Unit42 IOC Report](https://github.com/pan-unit42/tweets/blob/master/2023-03-06-IOCs-for-Gozi-infection.txt), the infection chain starts from an email, to a link, to a download. 

Although we could attempt to find email data in the pcap, unknowing to what the pcap may have saved on that aspect, it would make more sense to start with the link first. The main reason is that DNS requests aren't _typically_ encrypted (Disregarding technologies like DNS over HTTPS for now)

After opening the file with Wireshark, I ran through these steps to narrow down DNS activity:

**Apply the first Display filter**
```
dns
```

**Find the flag for a successful query response, and tune the display filter**
```
dns.flags == 0x8180
```

**Tune the display filter to cut out some (potentially) legitimate noise**
```
dns.flags == 0x8180 && not(dns.qry.name contains "microsoft") && not(dns.qry.name contains "google")

```

**Set the URL as a Column in Wireshark**
```
In Wireshark, click on one of the DNS packets. Right click on the "Name: URL" entry in one of them and click "Apply as Column"
```

Be cautious with a filter that disallows important keywords from the display results. Although a lot of common domains are expected (gstatic, microsoft, bing, windowsupdate, etc.), being overly specific is rather cumbersome and being too vague may kick out the actual malicious action.

So this is where we're currently at.

![Resize](https://blog.cloud-ham.com/posts/Unit42-Mar2023/Unit42-Mar2023-PCAP-P1.png?width=766px)


The initial domains of interest here are:

* unapromo[.]com
* x1.c.lencr[.]org

[Lencr.org is actually the domain for Let's Encrypt](https://letsencrypt.org/docs/lencr.org/)

However the other site that stood out, unapromo, is flagged from:

* [VirusTotal](https://www.virustotal.com/gui/domain/unapromo.com/detection)
* [URLScan.io Related behaviors](https://urlscan.io/result/240924f2-ef0d-4ef7-a2d6-6a61d186d877/related/)
* [DNS Community blacklist](https://github.com/NethServer/dns-community-blacklist/blob/master/abuse_ch.dns)

Since the page is already down, URLScan doesn't show suspicious activity on the page itself. The related behaviors tab shows files downloaded from it and past activity. From here, we can investigate past actions and find out just how malicious the site really is. From any of the hits from the same domain with a file associated, click the file URL in URLScan (Don't visit the actual domain!) and grab the SHA256 hash for analysis.

[Unit42-Mar2023-PCAP-P2.png]

[Unit42-Mar2023-PCAP-P3.png]

[Unit42-Mar2023-PCAP-P4.png]

SHA256 for Cliente.zip: 33db5b2a2cc592fd10c65ba38396e4c7574ad78e786d78e8a3acdc93a90c3209<br>
[Link for Urlscan Report of file](https://urlscan.io/result/6f017575-c1f1-48ea-8a20-7c55f6021b8b/)<br>
[Link for VirusTotal Report](https://www.virustotal.com/gui/file/33db5b2a2cc592fd10c65ba38396e4c7574ad78e786d78e8a3acdc93a90c3209/detection)

VirusTotal has the file flagged as "ursnif", and that lines up with the IOC report from Unit42 (Line45 in IOC report)

And although we haven't reviewed more of the PCAP yet to determine what IP our compromised system reached out to, we can save the IP address of unapromo for later: "173.254.32.85". We should also grab the packet number of the DNS request because we'll continue reviewing the PCAP and it'll be easiest to reference to hop to the DNS request in the timeline.

What we have so far:<br>
* The Host that made the DNS request has an internal IP address of "172.16.1.137"
* The URL of interest known with distributing ursnif malware used by Gozi is "unapromo[.]com"
* unapromo[.]com has resolved to "173.254.32.85" in the past
* The packet number in Wireshark for the DNS request is "2339"
* The suspicious initial file is most likely "Cliente.zip"

# Finding our Malware

Back in Wireshark, let's clear the filters we have right now. And instead we're going to dig more into network traffic where the potentially compromised host (172.16.1.137) was involved.

**Set the following Wireshark Filter**
```
ip.src == 172.16.1.137
``` 

**Hop the timeline to find the DNS request event**
```
In Wireshark, click on "Go" on the top bar, and select "Go To Packet". 
Set the packet number to 2339 and click "Go to packet"
```

If you look carefully, right after the DNS request is an immediate connection from our host, 172.16.1.137, to the IP address that was listed in the URLscan site in the above section, "173.254.32.85". And then right after, you guessed it, the "Cliente.zip" file!

[Unit42-Mar2023-PCAP-P5]

We can do the following the extract out the file:

```
In Wireshark, go to "File > Export Objects > HTTP"
Choose the unapromo[.]com hostname with the filename "Cliente.zip"
Choose "Save" and export to a location for analysis (Reminder: Make sure you're using a VM!
```

[Unit42-Mar2023-PCAP-P6]

Once we extract the contents, we'll see that this is an internet shortcut. Opening in Notepad++, we see the following contents:

[Unit42-Mar2023-PCAP-P7]

Now let's get some hashes. If we take the zip and the file and hash them with SHA-256, we'll see the zip with the same hash we saw earlier:

Cliente.zip: 33DB5B2A2CC592FD10C65BA38396E4C7574AD78E786D78E8A3ACDC93A90C3209<br>
Cliente.url: 340A759B1C1CDC22F6FAC84044D072475E1630FBB7F47D96C4E18413DE34D570

Ok, let's take some notes and do some quick reflection...

* Although the IP Address that the file was downloaded from was "172.254.32.85", this internet shortcut gives us another malicious IP: "46.8.19.32". This doesn't necessarily mean that this will be the target IP for ongoing C2 beacons or data exfil, just that this IP hosted at *least* the next stage.
* The Hash value we have for Cliente.zip matches what we found on URLScan.io
* Why did this download not trip any malware alerts? Well, it's an Internet Shortcut and there's a couple of issues. They don't really have distinct signatures outside of what might be a little strange (Such as the IconFile in this case). Hashing is ineffective as changing the filename in the URI would be enough to avoid hash-based detections. And if we run this through VirusTotal, it comes back as clean. This file is *directing us* to something malicious, but it isn't malicious in itself.

Let's chat for a bit about what you might do as a SOC analyst so far.

# The SOC Analyst review

At this point in the investigation, the system must be quarantined if not already. The file landing on the system, the system's interactions with a known malicious domain, and the contents of the InternetShortcut should be enough for us to make the justification to quarantine for either further review, or reimaging.

So what helps you decide whether you're going to review further or just submit for a rebuild/reimage? And keep in mind that this is not the "definitive" version as you must always consider your company's stance on these things, the technical details, and criticality of the system that was impacted.

* If you had a security product in the way that was actively stopping all effects of the attack after the system was compromised, you may be good to reimage. Maybe your system went all the way to executing the malicious code but luckily your EDR killed every process. Or your firewall let the attack in, but didn't allow it out. (I've seen some strange policy assignments on Palo Altos from customers that put stricter requirements on out than in, and led to C2 beacons that could fire but not leave the network).
* If you freshly discovered this and have no proof that anything else could have stopped the attack, you will want to dig in deeper. Not only considering this host, but any others in your network who may be performing the same activity. Also look to collect more information if the system is particularly critical or holds sensitive data.

Ok, now that we've talked about what you might want to do as a SOC analyst, let's go back to the network!

# The SMB Executable

Before the SOC Analyst review section, we talked about the new IP found in the InternetShortcut file that was downloaded: "46.8.19.32". If we still have our Wireshark filter set as the ip.src == 176.16.1.137 and looking at the packet referenced earlier, we can really just scroll down a bit. After that InternetShortcut is downloaded, it's not likely very far off. Or we can adjust our Wireshark filter a bit to get a better look. 

```
ip.src == 172.16.1.137 or ip.src == 46.8.19.32
```

We'll see at Packet #2492 the initial SYN from our compromised host to the new IP. After the TCP handshake is complete, a Server Message Block (SMB) handshake begins. This is an *immediately bad* scenario, as SMB should never be in use with unknown IPs. And now that we know SMB was in use, we could try using "Export Objects" in Wireshark again as well. (Truthfully, you could always start here, but a larger PCAP that isn't targeted for a scenario like this could be rather noisy, depending on protocol use).

Except we have an issue. Not all of the executable's data was captured in the PCAP. Wireshark will display a percentage in the "Content Type" field that indicates how much of the file was captured. At best, we have 73% of the file and could be missing some critical details from it.

[Unit42-Mar2023-PCAP-P9]

So analyzing this file may not give us a conclusive answer to any questions we may have. But we can at least poke and prod at it to see what might be present. I exported this on my FlareVM, and proceeded to pop it open in PEstudio for the first look.

[Unit42-Mar2023-PCAP-P10]

The strings analysis is probably the most useful part without the rest of the file present. We have some Windows API calls that flag for suspicion. The long strings with byte size as 2552 and 2001 are interesting. I'm no expert in decoding these things, but these were my notes and attempts:

* ROT13 brute force: Fail
* ROT47 brute force: Fail
* XOR Brute force: Fail
* Google translate: Fail
* Online text decoder (Do these even work?): Fail

I have a good feeling that the strings with 94 bytes in size in here are defining the encoding alphabet, but unsure of how to proceed here.

Let's refresh our current notes on the investigation with everything we found so far:

* The host that made the DNS request for unapromo[.]com has IP address "172.16.1.137"
* The URL of interest known with distributing ursnif malware used by Gozi is "unapromo[.]com"
* unapromo[.]com has resolved to "173.254.32.85" in the past
* The packet number of the initial DNS request to "unapromo[.]com" is 2339
* The Hash value for "Cliente.zip" matches what we found on URLScan.io
* The file downloaded from unapromo[.]com is "Cliente.zip", which contains an InternetShortcut that directs to IP address "46.8.19.32"
* Our host "172.16.1.137" begins a TCP handshake at Packet #2492, and starts the SMB negotiation with "46.8.19.32" at Packet #2495
* The file sent in the SMB traffic is "server.exe", but only 70% of the file was captured.
* From analysis of server.exe, we see suspicious strings for both Windows API calls and uninterpretable text that is most likely encoded

We're almost to the end. We're going to next look at the Gozi noted activity after the compromise, and then look to finish answering the questions for this challenge.

# Gozi Post-Compromise Actions

Here, we're going to lean on [Unit42's IOC report](https://github.com/pan-unit42/tweets/blob/master/2023-03-06-IOCs-for-Gozi-infection.txt) a little bit. The detail the type of activity seen pretty well here, and we're interested in the section "GOZI (ISFB/URSNIF) C2" starting on line 45, seeing as the SMB connection lead to the compromise.

Unit42 notes that all of the observed C2 traffic activity occurred over Port 80 as a GET request. In Wireshark, we're going to update our filter a bit.

```
ip.src == 172.16.1.137 && http.request
```

And honestly, this is enough. You'll see pretty quickly that this narrows the traffic down enough to see the C2 activity. 

[Unit42-Mar2023-PCAP-P11]

And the network traffic lines up with the IOC report incredibly well. We have the same URI format of either "drew/BASE64-encoded-text" or "stilak(Architechture).rar" or "cook(Architecture).rar"

Now we have a better understanding of the compromise and how bad it has gotten. We still don't really know what all was affected, or what could have been, but ultimately it's out of scope for this review. (We would probably need more than just PCAP data, and more than what's limited for the scenario, to paint the full picture)

# Finding the System

So we have our investigation pretty much wrapped up, except for knowing what the infected system is. If our DHCP is dynamic, and by chance the lease is renewed for a different IP by the time we investigate, we may not want to go by the internal IP address only.

We need to discover the system and the user account, both that should appear in authentication traffic of some sort. Let's start with the user's authentication on the Active Directory domain and look at Kerberos.

```
ip.src == 172.16.1.137 && kerberos
```
I won't lie, I very roughly understand Kerberos. But contacting the Authentication Service (AS) should result in some detail, and there I found the Hostname of the device: DESKTOP-3GJL3PV

[Unit42-Mar2023-PCAP-P12]

After finding this, I worked my way through the rest of the AS-REQ packets and eventually found the user account after several entries of the device.

[Unit42-Mar2023-PCAP-P13]

# Review

Wrapping up, we have our answers for the packet capture:

Host device details:<br>
* 172.16.1.137
* DESKTOP-3GJL3PV
* sherita.kolb

Initial source of infection:<br>
* unapromo[.]com/mise/Cliente.zip
* SHA256: 33db5b2a2cc592fd10c65ba38396e4c7574ad78e786d78e8a3acdc93a90c3209

Notable differences between indicators from this Gozi and the Unit 42 report?<br>
* The initial URL and zip file hash were different. Where we had "unapromo[.]com" and the hash ending in "3209", we have neither of those indicators in the IOC report from Unit 42. 
