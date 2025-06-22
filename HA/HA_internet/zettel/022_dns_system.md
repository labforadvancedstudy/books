# DNS System

## Core Insight
DNS is the internet's phone book - translating human-memorable names into machine-routable numbers, making the web navigable for humans.

Domain Name System performs daily miracles. You type "google.com", DNS translates to 142.250.185.78. Without DNS, we'd memorize IP addresses like phone numbers. The internet would work but be unusable. DNS makes the digital world human.

It's hierarchically beautiful. Root servers know where to find .com servers. .com servers know where to find google.com servers. google.com servers know their IP. Each level only knows the next level down. Distributed knowledge, no single point of failure.

DNS queries cascade. Your computer asks your router. Router asks ISP. ISP asks root servers. Answers cache at every level. The second visitor gets faster response. The internet remembers its lookups, learning popular destinations.

But DNS is also control. Governments block sites by poisoning DNS. ISPs redirect typos to ad pages. DNS hijacking steals traffic. Control the phonebook, control the internet. That's why DNS-over-HTTPS emerged - encrypting the lookups themselves.

DNS is trust. When you type a domain, you trust the entire chain to give honest answers. One compromised link and you're somewhere else. The internet's navigation system runs on faith in hierarchical infrastructure most users never see.

## Connections
→ [[013_domains]]
→ [[021_ip_addresses]]
→ [[023_routing]]
← [[012_urls]]
← [[046_certificate_authorities]]

---
Level: L2
Date: 2025-06-23
Tags: #dns #infrastructure #networking #translation