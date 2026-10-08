# Cloud Resources

## Overview

This lesson covers fully passive discovery of misconfigured public cloud storage (S3, Azure Blob, GCP) using Google dorking, a target's own source code, and specialized search engines.

## Learning Objectives

- Use Google dorks to find publicly exposed cloud storage referencing a company
- Inspect a site's own source code for cloud storage URLs
- Use domain.glass and GrayHatWarfare to discover exposed cloud buckets
- Recognize the real-world risk of credentials/keys left in public buckets

## Misconfigured Cloud Storage

Misconfigured cloud storage (S3 buckets, Azure Blobs, GCP storage) can be publicly accessible without authentication — and all of this is discoverable fully passively.

## Google Dorks

```
intext:"<COMPANY_NAME>" inurl:amazonaws.com
intext:"<COMPANY_NAME>" inurl:blob.core.windows.net
```

Results often include PDFs, docs, presentations, and code stored publicly.

## Source Code Inspection

A company's own site often already references its cloud storage URLs — images, JS, and CSS get loaded from there (DNS prefetch/preconnect links) to offload the web server, inadvertently revealing the storage endpoint.

## Specialized Tools

**domain.glass** — a third-party infrastructure overview tool; also shows Cloudflare security assessment status (useful as a Gateway-layer note).

**GrayHatWarfare** — a specialized search engine for publicly exposed cloud storage buckets (AWS/Azure/GCP), filterable by file format and searchable by company name or internal abbreviations.

## Real-World Risk

A real risk found this way: SSH private keys (`id_rsa`/`id_rsa.pub`) accidentally left in a public bucket by an overworked employee — leading to direct, password-less login to company machines.

## Skills Practiced

- Google dorking for exposed cloud storage referencing a target company
- Reading a site's own front-end source for cloud storage endpoints
- Using domain.glass and GrayHatWarfare for cloud bucket discovery
- Recognizing high-impact findings (leaked private keys) in exposed storage

## Key Takeaways

- Exposed cloud storage is often found passively, with no interaction with the target's own infrastructure at all — just search engines and public dork queries.
- A site's own front-end code frequently leaks its cloud storage endpoints through prefetch/preconnect hints meant purely for performance.
- A single misconfigured bucket can turn into full unauthorized access — leaked private keys are a realistic, high-impact outcome of this kind of passive recon.
