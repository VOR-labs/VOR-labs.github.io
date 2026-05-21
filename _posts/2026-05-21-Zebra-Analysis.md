---
title: "Zebra Analysis: Post-Quantum Cryptography in Developmental Ransomware"
date: 2026-05-21
author: "Matt Allan"
layout: post
tags:
  - Post-Quantum-Cryptography
  - Darkweb-forums
  - Go-binary-analysis
  - malware-analysis
  - reverse-engineering
---
## Introduction

Within the Cyber Threat Intelligence (CTI) ecosystem, defenders tend to focus on the threats being used in the wild against enterprise infrastructure. However, maintaining a forward-looking perspective requires monitoring emerging malware trends in an effort to keep up with the rapid changes taking place. In some cases, the malware samples may still be in the early stages of development. During research I was conducting on a dark web hacking forum, I encountered one such sample.

The malware, dubbed Zebra, is an early-stage ransomware variant posted by an aspiring malware developer seeking technical feedback and perhaps credibility within the community. While the sample itself is relatively immature, the implementation of the post-quantum encryption algorithm `ML-KEM-768` may signal a broader trend toward more advanced cryptographic techniques.

Although Zebra is unlikely to pose a significant threat in its current form, its adoption of post-quantum cryptography highlights how emerging encryption standards are beginning to appear even within low-maturity malware development communities. The following research documents the discovery, analysis, and technical assessment of the sample.

## Malware Metadata

    MD5 = 7bdbb641dad2537db2eb96a949cd5ac0
	SHA-1 = 8a3ddc815b8943c83fd4b6043a0f0ec30e757e6b
	SHA-256 = 1cd40fb0e66873ac9f961b7c7936e817b068b9436729ecef18387a4a376788fc
	File Type = Win32 EXE
	File Size = 3.62 MB
	Architecture = x86-64
	Language = Go

## Discovery

As part of ongoing research, I began monitoring dark web hacking forums and marketplaces to better understand the broader cybercriminal ecosystem. These spaces commonly contain scams, technical knowledge exchanges, malware development discussions, and operational guidance shared among threat actors and aspiring developers alike. During this research, I discovered an English-speaking hacking forum through Dread, a Reddit-style discussion platform widely used across dark web communities.

## Khazad

While browsing Dread, I came across a subsection dedicated to advertising hidden services. A user was advertising a new English-speaking hacking forum named Khazad. I navigated to the site, which appeared to be a typical underground forum with discussions related to exploits, scams, vulnerability research, and malware development.

In the malware development section of the forum, a user was advertising a version of a ransomware variant they referred to as Zebra. Based on the user’s post history, they appear to have an interest in malware development and are still relatively early in their technical maturity.

![Figure 1: Screenshot of the advertisement of Zebra]({{ "/assets/images/Advertisement.png"}})

Following the links provided in the initial forum post led to a thread on EndChan, a decentralized board site frequently used for anonymous technical exchanges and leaks. This secondary hosting arrangement may serve as a simple operational security measure intended to separate the malware's hosting from the original advertisement.

Further analysis of the Zebra EndChan board logs indicates that the board had been active since 2024 and was operated by a user identified as `sozph`. The board was later removed in 2024 before reappearing in 2026 under another user operating under the alias `cosponsor`.

The Zebra FAQ was hosted separately on Eternal, a privacy-focused hosting service. Eternal markets itself on the promise of mandatory server-side encryption and a strict no-logs policy, ensuring that the actor’s documentation remains accessible to potential affiliates.

![Figure 2: A snippet of the FAQ document]({{ "/assets/images/FAQ.png"}})

I obtained a copy of the ransomware sample and transferred it to my malware analysis lab for further examination.

## Analysis

Per the FAQ, this sample is cross-compiled with the ability to run on Windows, Linux, and macOS (both Intel and Apple Silicon). For the purposes of this analysis, I will be analyzing the Windows binary.

I first ran my tool [`dummy-triage.py`](https://github.com/VOR-labs/dummy-triage) against the binary to produce an initial static overview. The results indicated that the sample is a statically compiled Go binary, based on extensive runtime artifacts and Go module-style path strings. While there were no definitive execution-time indicators of ransomware behavior, the analysis did surface multiple capability-level concerns.

These included a cluster of Windows APIs commonly associated with process injection and thread manipulation (`SuspendThread`, `SetThreadContext`, `GetThreadContext`, `VirtualAlloc`, etc.), as well as cryptographic primitives consistent with file encryption or key derivation workflows (`HKDF-SHA2-256`, `HMAC-SHA2-256`). Additionally, a large encoded string was present, likely representing embedded configuration or encrypted payload data.

The next step was to dynamically analyze the malware for behavioral IOCs. Unfortunately, the malware simply printed the message `Please wait...` to the console and exited before encrypting any files.

This abrupt exit makes sense when mapped back to the symbol table. The binary contains specific functions called `sleepController` and `wakeableSleep`. This resembles a common anti-analysis technique in which malware measures execution timing to identify sandbox environments that accelerate or manipulate sleep intervals. If the environment appears suspicious, it triggers a cooldown and exits the process. It is also possible that the sample immediately encountered the ACL-related bug discussed later in this report, throwing an unhandled error before it could begin encrypting files. Outside of a few initialization API calls, there was limited value in dynamic analysis.

The malware was then loaded into Ghidra for deeper code analysis. The malware was written in Go, which leads to noisy output in the decompiler due to the way Go handles string and memory layout. Additionally, it is statically linked, meaning we have access to every library the malware uses during execution. Only `kernel32.dll` appears to be imported in this binary.

The symbol table gives us a clear understanding of how the malware is supposed to work. The overall execution flow is relatively straightforward and consistent with basic ransomware design patterns:

`main.main → main.encFiles → zebra.NewKeyData → zebra.Enc → zebra.Crypt`

Functionally, the malware is designed to scan each file (using the function `zebra.Scanner`) identifying files that match a targeted list of file extensions and are smaller than 256 MiB. If these conditions are met, the malware uses `ChaCha20` to encrypt the files. Additionally, Zebra is designed to dynamically write and drop a ransomware note named `000-README-ZEBRA.txt` in the directory where the malware was executed. Finally, encrypted files are appended with the extension `.zebra`.

![Figure 3: A snippet of code at the start of the ChaCha20 encryption]({{ "/assets/images/ChaCha20.png"}})

Additionally, Zebra has the ability to generate a file named `ZebraRestore.json`. If this file is found, the malware will decrypt encrypted files located on the system. There are no network IOCs, meaning that in its current form Zebra simply encrypts files locally and can restore them locally with the proper command-line arguments. This indicates that Zebra is still in an early developmental phase and may have been posted to the forum primarily to receive feedback from other users.

## Bleeding-Edge Cryptography

Up until this point, Zebra appears to be a pretty standard developmental piece of ransomware. However, the most interesting aspect of the malware is its use of post-quantum cryptography. 

In the `.rdata` section, there is a stack string indicating the use of `ML-KEM-768`. `ML-KEM-768` is an advanced cryptographic concept designed to be resistant to being cracked by quantum computers. How the algorithm works is outside the scope of this blog post, but you can find more information about it [here](https://en.wikipedia.org/wiki/ML-KEM) or review the [NIST publication FIPS 203](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.203.pdf).

At a high level, this algorithm is a key-encapsulation mechanism (hence KEM) that can be used to establish a shared secret key. Conceptually, it functions similarly to elliptic-curve-based key exchange mechanisms, but is designed around lattice-based cryptography intended to resist quantum attacks. The `ML-KEM-768` key is Base64 encoded and is decoded to a 1184-byte key (which is expected for a key related to this algorithm).

To put this into perspective, it helps to look at how this hybrid pipeline is supposed to function. The malware doesn’t use heavy post-quantum math to encrypt your actual files—that would be slow and inefficient. Instead, it relies on `ChaCha20` for the heavy lifting to rapidly lock down your data on disk. The `ML-KEM-768` layer is strictly used to encrypt and protect the unique symmetric keys generated for that session, which get dumped into `ZebraRestore.json`. Without the attacker’s private key to unpack that post-quantum layer, a victim can’t get to the ChaCha keys needed to decrypt their data.

![Figure 4: Hex value of the Base64 encoded `ML-KEM-768 key`]({{ "/assets/images/XXD-output-key.png"}})

Additionally, the version of Go that was used to write this malware is `1.26.1`, which was released in February of this year. This indicates that the malware author is experimenting with recently released cryptographic functionality.

## The Paradox

Zebra has multiple bugs, but the absolute fatal flaw lives inside the `zebra.encryptFile` function. In an attempt to ensure the encryption process cannot be easily interrupted by a user or security software, the malware author tries to lock down the Access Control Lists (ACLs) of target files.

To do this, the program calls `ADVAPI32!SetNamedSecurityInfoW` to modify permissions. However, the developer runs this API on the new encrypted file path (the one intended to end in `.zebra`) before actually invoking `KERNEL32!CreateFile` to create it. On Windows systems, trying to modify security descriptors on a file path that doesn't exist yet causes the API to return an immediate error `ERROR_FILE_NOT_FOUND` (or 0x2). Because the malware's error handling just forces a hard exit when this happens, the ransomware effectively defeats itself and crashes before it can encrypt a single byte of data.

This creates a massive paradox: we are looking at a malware developer who is forward-thinking enough to track bleeding-edge cryptographic standards, yet completely lacks a basic understanding of fundamental Windows internals. Whether this was a simple mistake or a flawed copy-paste job from an AI coding assistant, it means that in its current state, Zebra is not an active threat.

## Conclusion

The Zebra ransomware is a fascinating sneak peek into what the future of the ransomware landscape might look like. While the sample is highly developmental—and likely just a side project by a learning developer trying to gain notoriety on dark web forums—it highlights how easy it has become for threat actors to implement post-quantum cryptographic standards into their work.

The author didn't need an advanced degree in mathematics to pull this off; they just took advantage of Go's native standard library (`crypto/mlkem`), which makes deploying quantum-resistant keys as simple as writing a single import line. If an inexperienced malware developer can easily integrate post-quantum engineering into a broken binary, it forces us to ask a bigger question: what are highly motivated, nation-state actors building with these exact same standard libraries? Although Zebra is not operationally mature, its implementation demonstrates how quickly advanced cryptographic standards are becoming accessible even within lower-tier malware development communities.

Full IOCs and additional malware artifacts can be found on my github [here!](https://github.com/VOR-labs/malware-analysis-artifacts/tree/main/Zebra) As noted, this malware has been cross compiled to run on different Operating Systems (OS). The following are the has values of those additional samples that were not analyzed in this research.

## Yara Rule

	rule Win_Ransomware_Zebra_Dev {
	    meta:
	        description = "Detects early-stage Zebra ransomware Windows"
	        author = "Matt Allan"
	        date = "2026-05-21"
	    strings:
	        $s1 = "zebra/cmd/zebra/main.go"
	        $s2 = "ML-KEM-768definitionCounter"
	        $s3 = "000-README-ZEBRA.txt"
	        $s4 = ".zebra"
	    condition:
	        uint16(0) == 0x5A4D and all of them
	}

## Additional Samples Metadata

### Linux

	MD5 = 93bbf45f6248785864643654ccb79063
	SHA1 = f75701cb2fb7b3b7111aff660cc2eb3fd6232b6f	
	SHA256 = 50ce9928054372bc3fdd215481916ef947906438c37d93778b24b3f44f285448	

### MacOS (Intel)

	MD5 = 7e564a1b78a9f303a5bcaad13e40d61d
	SHA1 = b31bb62d8129949cf2ad8a5fbcf2d8907766e345
	SHA256 = a0c46e3ee6173a339f98fdca4f4df88087e6f226cb5254f6f55a8b4707847b3e

### MacOS (Arm)

	MD5 = 938136c53d9760b35fe7125c78578ed7
	SHA1 = 2d663e688416a73c3afe601487a0581e1cdb98ab
	SHA256 = 2c8a16b64e34bdda70db01ad930ec7498bf68d5153c0fbb619ca19d3c27afae1
