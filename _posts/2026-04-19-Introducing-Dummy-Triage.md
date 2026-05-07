---
layout: post
title: "Introducing Dummy Triage"
date: 2026-04-19
categories:
  - malware-analysis
tags:
  - new-tool
  - triage
---

## Dummy-triage.py

There are an overwhelming number of tools in the cybersecurity space. Between researchers building specialized tools and the growing use of AI-generated code, there is a growing number of open-source tools daily. In my experience, many open-source scripts require some tweaking to fit a specific workflow, and it can be difficult to find a tool that produces exactly what you need out of the box. To address this, I built a lightweight triage tool of my own.



**Dummy-triage.py** is a straightforward Python script designed to automate the initial "gut check" of a file. It leverages powerful libraries like [pefile](https://github.com/erocarrera/pefile) and Python's native [hashlib](https://docs.python.org/3/library/hashlib.html) to perform the following tasks:

- **File hashing:** Automatic generation of MD5 and SHA256 hashes for pivot searching.
- **String extraction:** Basic indicator parsing to find IP addresses, URLs, and file paths.
- **PE Analysis:** Extraction of Portable Executable header metadata and section information.
- **Structured JSON reporting:** Outputs results in a format that is easily ingested by other tools or databases.

The goal of this tool is to automate the very first step of my malware analysis workflow. While many enterprise tools already perform this process, I’ve found that building and maintaining your own lightweight scripts can greatly expedite the analysis process and allows for deeper customization. I plan to integrate this into future automated analysis pipelines, so I’m sharing it in case it’s useful to others in the community.

### Future Improvements
To move this from a simple script to a robust triage engine, I am working on:
- **Multi-format support:** Adding ELF (Linux) and Mach-O (macOS) parsing.
- **Intelligent Filtering:** Improved string extraction to reduce noise from legitimate library strings.
- **Heuristics:** Suspicious import detection (e.g., identifying packing or injection-related APIs).
- **YARA Integration:** Allowing the script to scan samples against custom rule sets automatically.
- **Done! Batch Processing:** Analyzing entire directories of samples at once.

Please feel free to give it a try or contribute to the project here: [https://github.com/vor-labs/dummy-triage](https://github.com/vor-labs/dummy-triage)
