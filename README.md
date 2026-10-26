# INE Skill Check: Information Gathering CTF 1

> **Focus:** Website reconnaissance, WordPress fingerprinting, exposed directories, backup-file exposure, and offline website analysis  
> **Environment:** Authorized INE Skill Check lab  
> **Target:** `http://target.ine.local`  
> **Publication safety:** Flag values, database credentials, WordPress salts/keys, and other sensitive configuration values are intentionally omitted.

## Overview

This repository documents an authorized INE Skill Check exercise for **Assessment Methodologies: Information Gathering**. The objective was to investigate a target website using reconnaissance techniques and locate five flag objectives.

The investigation combined browser-based inspection, HTTP response/source inspection with `curl`, directory-enumeration observations recorded in the lab notes, WordPress fingerprinting, and an offline mirror created with HTTrack.

## Objectives and findings

| Objective | Investigation | High-level finding |
|---|---|---|
| Flag 1 | Inspect `/robots.txt` in Firefox | Examined the file used to communicate crawl guidance to search engines. |
| Flag 2 | Inspect page metadata with `curl` | Identified WordPress and its reported version from the generator metadata. |
| Flag 3 | Browse the uploads path | Found that the WordPress uploads directory was listable in the lab. |
| Flag 4 | Investigate a backup configuration file | WPScan identified `wp-config.bak`; its contents included sensitive configuration material. |
| Flag 5 | Mirror the site with HTTrack and search local files | A flag marker was found in a mirrored XML-RPC-related file. |

The supplied transcript does not contain the full `robots.txt` contents or the exact files displayed in the uploads directory, so this overview does not invent those details.

## Repository contents

- [`lab-notes.md`](lab-notes.md) — detailed methodology, command explanations, observations, and defensive lessons.

## Tools

The lab brief listed Firefox, `curl`, and HTTrack. The supplied solved transcript also records DIRB and WPScan output. If reproducing the skill check, follow the tool restrictions currently stated by INE for that lab; treat any additional-tool results as part of the recorded session only where permitted.

## Key lessons

- Reconnaissance clues should be validated rather than treated as proof by themselves.
- Generator metadata and asset version strings can disclose the website platform and version.
- Directory listing can expose uploaded files and help reveal unexpected content.
- Backup configuration files can disclose database settings and authentication secrets.
- Mirroring a site creates a local copy that can be searched for information not immediately obvious during normal browsing.

## Responsible disclosure and publication note

This is a record of work performed in an authorized training environment. Public versions should not contain actual flag values, credentials, secret keys/salts, session data, or other restricted lab content. Check INE's current rules before publishing detailed Skill Check solutions. Do not test these techniques against systems without explicit authorization.
