# Upload2Shell (CVE-2026-125396, Ghostscript): exposure indicators and hunt guide

> **Read first.** This package is aggregated from public research and vendor documents by SOC-Lite by BansalAI. It is not
> tested by us, it comes with no warranty, and it is not professional advice. It is as of **2026-11-24** and it will go stale.
> See [DISCLAIMER.md](DISCLAIMER.md). We are not affiliated with any company credited in [CREDITS.md](CREDITS.md), and none
> of them reviewed or endorsed this package.

Ghostscript is a free program that converts PostScript and PDF files. Many websites, webmail servers and print servers run it
behind the scenes, usually through an image tool. In November 2026 researchers at pwn.ai described a flaw in its font loader,
**CVE-2026-125396** ("Upload2Shell"), that lets a crafted file escape Ghostscript's sandbox and run commands as the account that
rendered it. Artifex fixed it in Ghostscript 10.08.0 on 2026-09-08. The researchers showed it working through a shared print
queue, Roundcube webmail, WordPress uploads and other tools. Sophos reported no active attacks after 2026-11-21, and a working
demonstration is public.

**There are no attacker indicators to copy.** No source has published attacker addresses, file hashes or domains. So this
repository does two other things: it lists the **exposure and hunt indicators** the sources describe (versions, settings,
routes and behaviours to look for), and it gives a guide to checking your own systems.

**Not technical? Start with the plain-language version:**
[Upload2Shell: What to Ask Whoever Runs Your Website, Webmail or Print Server](https://blog.bansalai.com/news/upload2shell-ghostscript-uploads-what-to-ask/?ref=github)
on the SOC-Lite blog. It gives five questions to put to your IT person or provider.

## What is in here

| File | What |
|---|---|
| [`iocs/iocs.csv`](iocs/iocs.csv) | One row per exposure or hunt indicator, with the source, the date, how fragile it is, and a review date |
| [`docs/hunt-guide.md`](docs/hunt-guide.md) | How to find Ghostscript and check the version, which routes reach it, what to look for in uploads, processes and print logs, what the results mean, and what to do if something looks real |
| [`CREDITS.md`](CREDITS.md) | The research and documents this is built from, and what we took from each |
| [`DISCLAIMER.md`](DISCLAIMER.md) | The full disclaimer |

## Where the indicators come from

**Rows:** 19 · **verified (two or more sources):** 3 · **single source:** 16

These counts match [`iocs/iocs.csv`](iocs/iocs.csv); we check them every time we change the file (a test checks them in our
private working copy). Every value was read from the publisher's own page. What we took, and where it came from, is in
[CREDITS.md](CREDITS.md). When a page shows no date, `source_date` is the day we read it.

### What the rows are

| Type | Rows | What they are |
|---|---|---|
| `cve` | 3 | The identifiers to look for in a vulnerability scanner or package record |
| `string` | 8 | The usual first line of PostScript and EPS files, two upload form plugins the researchers tested, and fixed package or product versions |
| `behaviour` | 7 | Setups and activity to look for: Ghostscript run by a service account, a shell started by Ghostscript, a print queue open without a login, a permissive image-tool policy, bundled copies of Ghostscript, services not restarted |
| `file-path` | 1 | The ImageMagick policy file to read |

Rows by type: 3 cve, 8 string, 7 behaviour, 1 file-path

### What is not here, and why

- **No attacker addresses, file hashes, domains or key fingerprints.** None have been published. We do not guess or
  reconstruct values.
- **No exploit details or demonstration code.** The researchers' write-up contains a working demonstration. We do not copy
  it, describe its method, or link to exploit repositories.
- **No fixed package versions for Ubuntu, Debian, SUSE, Amazon Linux or the Red Hat family**, apart from AlmaLinux's, which
  we read on the page. Those change quickly: read your vendor's tracker. The hunt guide says where.
- **No detection rules.** We found none we could use under a clear licence.

## Read the results carefully

- **A match is a reason to look, not a verdict.** Starting Ghostscript from a web account is normal for thumbnails. A
  PostScript signature in a file with a picture name is a strong lead but a miss proves little.
- **Most rows are exposure checks.** They tell you whether a route exists, not whether anyone used it.
- **There is no published log signature for the attack.** An empty search is not an all-clear.
- **The plugin rows are not accusations.** The flaw is in Ghostscript. Those plugins were part of the researchers' tested
  stacks because they let a visitor with no login upload a file.
- **Sources can change.** Distribution fix status is moving fast. Follow the links in [CREDITS.md](CREDITS.md).

## Using `iocs.csv`

| Column | Meaning |
|---|---|
| `id` | Stable row ID (`U2-nnnn`) |
| `wave` | Always `upload2shell` in this release |
| `type` | `cve`, `string`, `behaviour` or `file-path` |
| `value` | The indicator. A version, a file signature, a path or a short description of the behaviour |
| `match` | `exact`, `prefix`, `substring` or `wildcard` |
| `evidence` | Where it would show: `package`, `file`, `process`, `network` |
| `fragile` | `yes` if the attacker can change it trivially |
| `verified` | `yes` only when two or more independent primary sources report it. `no` means a single source |
| `source_url`, `source_date` | Where we took it from and the date of that source |
| `fetched`, `review_by` | When we read it, and when it should be re-checked (30 days for every row here) |
| `notes` | Context and caveats. Sentences starting "Our note" are ours, not the source's. A "Second source" link marks a verified row |

Search your own systems with the steps in the hunt guide. This package does not ship anything that runs on your systems or
collects data from them.

## Licence

The text we wrote is licensed under CC BY 4.0 ([LICENSE](LICENSE)). The indicator values and facts belong to the research and
documents that reported them: they are credited in [CREDITS.md](CREDITS.md), and we do not relicense them. Details and how to
credit us are in [NOTICE.md](NOTICE.md).

## Where to go for the full, current picture

pwn.ai's write-up is the authority on what the flaw is and how it was reached. The distribution and application vendors are
the authority on their own fixed versions. Both may be updated: follow the links in [CREDITS.md](CREDITS.md).

## Help with a suspected compromise

If something looks real, preserve the logs first and call your IT provider. Section 6 of the hunt guide lists what to do and
in what order. [SOC-Lite by BansalAI](https://cyber.bansalai.com/?ref=github) helps small and mid-sized Central Florida
businesses review alerts and work with their IT person or MSP.
[Book a free 20-minute visibility check](https://cyber.bansalai.com/?ref=github#book). More plain-language security
write-ups are on the [SOC-Lite blog](https://blog.bansalai.com/?ref=github).
