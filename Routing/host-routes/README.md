# Host Routes (`/32` & `/128`)

## Core Concepts
A host route is a prefix with a `/32` mask (`255.255.255.255` in IPv4) or `/128` prefix in IPv6 targeting a single specific endpoint IP address. Because routers always forward based on **Longest-Prefix Match**, a `/32` or `/128` host route always takes precedence over any `/24` or `/64` subnet route regardless of Administrative Distance.