---
layout: post
title: The pains of entropy in the cloud
subtitle: A history of entropy on virtualisation
bigimg: /img/entropyequation.gif
---

I noted a new  feature listed on the Ubuntu Security page called 'Cloud PRNG seed'. This got my interest and so I decided to research further, to see what the drivers were for such an implementation.

However, before I delve into Ubuntu's solution, I feel it's prudent to do a quick primer / refresher as to what entropy is, and why its considered a potential achilles heel within virtualisation & cloud.

# ENTROPY

A majority of devices with a need to identify or authenticate themselves or others,  utilize private encryption keys.

For these private keys to generate, they require an initial random seed (a block of random numbers). This seed is then used as a base formula to mathematically compute the private keys, an “initialization vector” as it's technically known. Producing these random number blocks, is referred to as Entropy.

Private keys, need to be as unique as possible. If keys are generated using the same seed, then it would be trivial to compromise the system, or for a malicious system to falsely appear as a trusted device (opening up the possibility of MITM attacks).

They are actually a significant amount of duplicate keys around, so this is not just typical security guy scaremongering. An example of duplicated keys was recently discovery by [John Matherly](https://blog.shodan.io/duplicate-ssh-keys-everywhere/) founder of the most excellent shodan.io. Telefonica Espana (or their partner manufacturer) appear to have circulated around 250,000 routers with duplicate keys from using the same OS image on every instance.

To insure a unique non-duplicated seed, systems and applications utilize Entropy. To prevent duplicate private keys being distributed in the same OS image, well that's an educational matter.

![dilbert](https://raw.githubusercontent.com/lukehinds/lukehinds.github.io/master/img/dt011025.gif)

The Linux Kernel utilizes a file to which it spools its entropy, /dev/random.

# /DEV/RANDOM & HWRNG (NOT THE HEAVY METAL MAGAZINE)

A Linux operating system utilizes entropy based crypto generation for a good number private keys

~~~
SSL keys
SSH keys
GPG keys
/etc/shadow salts
TCP sequence numbers
UUIDs
dm-crypt keys
eCryptfs keys
~~~

Within Linux, entropy is typically provided by /dev/random.

/dev/random is a special file that serves as a pseudorandom number generator. It collects environmental noise from device drivers such as keyboard, mouse input, HDD seek timing, Interrupt requests and network I/O (to name but a few). The file can then be utilized as an initial seed for key generation (or any type of randomized function!).

Its also important to note that entropy is exhaustible. So when the pool is depleted, reads from /dev/random will be blocked and the application needing to generate a key will have to sit and wait. This results in OS or application start up being significantly delayed, compounded even more by the delay in turn resulting in less activity folded back into the entropy pool.

Note: There is /dev/urandom, but its considered less secure, as after depletion of the pool of entropy, it continues to serve.

If entropy generation using just /dev/random is an issue, then hardware solutions exist (hw-rng)and have done for around 15 years or so (picture below is a serial based hwrng from Protego in Sweden manufactured around 2000). More recent hwrng's are USB or PCI/e card based.

![Protego](https://raw.githubusercontent.com/lukehinds/lukehinds.github.io/master/img/protego-300x168.jpg)

To utilize a hwrng, the Linux kernel contains a  function called hwrng (surprise!).  The rng-tools package comes with rngd, a daemon, that reads input from hwrngs and feeds them into the kernel’s entropy pool.

However, to utilize a hwrng within a virtualized enviroment, is a  challenge. Unlike a dedicated physical machine, its harder to scale and provide hw-based rng's to multiple virtualized machines.

It is also difficult for virtual machines to provide a plentiful pool of entropy for key generation algorithms to draw from.  A VM does not have its own hardware to utilize for environmental noise.

This reliance on Kernal fed /dev/random in a cloud type deployment, can cause significant performance issues, especially in virtual machines, where we need elasticity and the ability to scale up new VM's on demand. Virtual Machines are notorious for being a bit of a desert when it comes to entropy sources.

## VIRTIO RNG & HAVEGED(8)

One solution to a VM guests lack of entropy, is virtio-rng or haveged(8)

## VIRTIO RNG

Best to quote the qemu-project description for VirtIO RNG, they put it simply and succinctly...

"VirtIO RNG is a paravirtualized device that is exposed as a hardware RNG device to the guest. On the host side, it can be wired up to one of several sources of entropy, including a real hardware RNG device as well as the host's /dev/random if hardware support doesn't exist. "

So as you can see vRNG resolves the scale issue. It's not to different in principle to SRV-IO. Hardware is paravirtualized for use by multiple VM's.

## HAVEGED(8)

HAVEGE is essentially an algorithm. Haveged(8) *the application* in Linux systems, leverages timing information based on the processor's high resolution timer (the RDTSC instruction) to create Entropy. A good explanation on haveged(8) can be found on Fraser Tweedale's blog.

So to summarize:

VirtIO RNG provides a scalable type entropy for guests, as long as a hardware rng is present.

Haveged(8) is also a good option, as long as the "golden master" image has haveged installed (post OS installation is no good, this needs to happen on first boot)

Now if you have your own private cloud, then it's of course possible to implement haveged(8) or buy some hw-rng's and harness, virtIO-rng. The problem however is for people who use cloud providers with set MI's (Machine Images), for example Amazon web services.

AWS and like do not provide hw-rng's, nor do the AMI's (Amazon Machine Images) contain haveged(8). I have heard that even if you could find a way to get haveged onto Amazon, it would not work as they have a modified xen kernel to block the ability for guests sniff CPU timing info. The other point is you're rely on someone else to give you good random data to create your keys.

There are also known potential exploits of the AMI lifecycle, that raise exposure for cloud consumers...

A potential PRG cloud based attack was highlighted back in 2009  by iSEC partners at a blackhat event.

The theory was to fingerprint a victims SSH fingerprint, pull down a AMI (Amazon Machine Image) and then use the PRG seed to re-bundle a compromised guest. See below for a high level overview:

So essentially this then leads to cloud guests requiring multiple sources of Entropy to stop MI spoofing. Reliance on one (possible replicable) source, results in the iSEC compromise being a possible (I say possible, as it is only a theoretical attack at present)

## ENTROPY-AS-A-SERVICE

Cloud PRNG (Pseudo Random Number Generator) is a project termed [pollinate](http://bazaar.launchpad.net/~kirkland/pollen/trunk/view/head:/README) developed by Ubuntu.

Pollinate itself is a client that calls a 'server' over HTTP/S for additional entropy for when a virtual machine first starts up. It can then addition this entropy alongside others sources, such as its parent hosts entropy.

Here is the description from launchpad.net

Pollen is a scalable, high performance, free software (AGPL) web server, that provides small strings of en  tropy, over TLS-encrypted HTTPS or clear text HTTP connections.  You might think of this as 'Entropy-as-a-Service'.

Pollinate is a free software (GPLv3) script that retrieves entropy from one or more Pollen servers and seeds the local Pseudo Random Number Generator (PRNG).  You might think of this as PRNG-seeding via Entropy-as-a-Service.

Please understand...neither Pollen nor Pollinate increase the amount of entropy available on the system!  Rather, Pollinate adequately and securely seeds the PRNG in cloud virtual machines through communications with a Pollen server.

Pollen itself can be self-hosted, or you can utilize an online service they have at entropy.ubuntu.com. I guess more people will want to host themselves, but for others needing a quick on-demand cloud service, the ubuntu nodes will suffice. The extra plus is the ubuntu hosted service is fully HA.

Now generally one would get very nervous about obtaining anything in anyway crypto related over the internet, but in this instance its not an issue. You have to keep in mind that seed data from 'entropy.ubuntu.com', is 'mixed' in with your locally generated entropy pool and that pool in-turn is influenced by the network traffic generated I/O activity to utilize the pollinate service. This makes for something extremely difficult to replicate (impossible I would argue). Another key factor is even if the client / server connection was sniffed, it does not weaken the security state beyond its primary entropy collection,  so in other words, you can never be worse off...

To quote reddit user Uguuuu  on the /r/netsec thread..'but even if you can intercept and feed the response with a controllable result, it does not make the entropy and less secure that it would be had there been no extra entropy feed.'

I think for a later post, I will implement pollinate in AWS and let's see how much of a difference it makes to the entropy pool and start up times.

sources / references used for this post:

http://people.canonical.com/~kirkland/Random%20Seeds%20in%20Ubuntu%2014.04%20LTS%20Cloud%20Instances.pdf

http://log.amitshah.net/2013/01/about-random-numbers-and-virtual-machines/

http://blog-ftweedal.rhcloud.com/2014/05/more-entropy-with-haveged/

## UPDATE

I noted that red hat now support virtio RNG as of RHEL 7, so implementation is made a lot easier for users. More info can be found [here](http://rhelblog.redhat.com/2015/03/09/red-hat-enterprise-linux-virtual-machines-access-to-random-numbers-made-easy) on the red hat site.
