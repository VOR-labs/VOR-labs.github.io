---
title: Home
layout: default
---

# Vector of Research Labs
**Technical Intelligence | Binary Analysis | Adversary Intent**

Welcome to the lab. This site serves as a technical notebook documenting my work in deconstructing complex software and uncovering the logic behind modern threats.

### Core Research Areas
*   **Malware Analysis:** Behavioral and static deconstruction of high-impact threats.
*   **Vulnerability Research:** Deep-dives into firmware and application-layer flaws.
*   **Reverse Engineering:** Exploring tools and techniques to help dive deep into low-level code.
*   **Threat Intellegence:** Analyzing threat actors actions and objectives.

I am **Matt Allan**, a Technical Security Researcher and Malware Reverse Engineer. This platform is where I transform deep-dive reverse engineering into scalable threat intelligence.

---

## Recent Research
Explore the latest technical advisories and lab notes.

{% for post in site.posts limit:5 %}
### [{{ post.title }}]({{ post.url | relative_url }})
*{{ post.date | date: "%B %d, %Y" }}*  
{{ post.excerpt | strip_html | truncatewords: 25 }}

{% endfor %}

<p align="center">
  <a href="{{ '/research/' | relative_url }}">View Full Research Archive →</a>
</p>
