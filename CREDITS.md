# Credits and sources

This package is aggregated from the public research and vendor documents below. **We thank their authors for publishing
them.** Each row in `iocs/iocs.csv` names the source it came from. Being listed here does not mean any of them reviewed,
tested or endorsed this package.

We copy facts (version numbers, package names, identifiers and dates) with attribution, and link to everything else. Where a
source holds more than we reproduce, follow the link for the full, current version.

## The research that supplies the indicators

| Who | What we used | Link |
|---|---|---|
| **pwn.ai**, "How We Hacked the Pipes: Upload2Shell" (the article credits the finding to an AI model working with Paulos Yibelo; WordPress and AlmaLinux thank pwn.ai and Paulos Yibelo for reporting it) | The flaw's description and dates (Ghostscript fix 2026-09-08, public number 2026-11-09, WordPress fix 2026-08-12); the routes the researchers tested (print queue, Roundcube, WordPress and two plugins, other tools) and the accounts the renderer ran as; that the test files were PostScript under picture or PDF names; the point that containers and bundled copies are not fixed by a host update; the status of Linux distributions as of 2026-11-12 (linked, not copied) | https://pwn.ai/upload2shell |

## Vendor and project documents behind the exposure rows and the fixed versions

| Who | What we used | Link |
|---|---|---|
| **WordPress**, "WordPress 7.0.4 Release" (2026-08-12) | The security fix for Author-plus remote code execution on sites that use Imagick and Ghostscript, the backports to older branches, the advisory identifier CVE-2026-65640 | https://wordpress.org/news/2026/08/wordpress-7-0-4-release |
| **Roundcube**, "Security updates 1.6.20 and 1.7.5 released" (2026-10-19) | The fix for passing unsafe attachment content to ImageMagick, and the fixed versions 1.6.20 and 1.7.5 | https://roundcube.net/news/2026/10/19/security-updates-1.6.20-and-1.7.5 |
| **cPanel**, "Security: CVE-2026-125396 - Ghostscript - Upload2Shell" | The fixed WordPress and Roundcube versions for cPanel servers, the location and the intent of the ImageMagick policy measure, and its warning that previews may break | https://support.cpanel.net/hc/en-us/articles/45216747700503-Security-CVE-2026-125396-Ghostscript-Upload2Shell |
| **Plesk**, "Vulnerability CVE-2026-125396 and CVE-2026-114727 (Upload2Shell)" | The fixed Plesk version (18.0.81 or later), that the related Roundcube CVE is CVE-2026-114727, and that Plesk does not update Ghostscript | https://support.plesk.com/hc/en-us/articles/45221939870871-Vulnerability-CVE-2026-125396-and-CVE-2026-114727-Upload2Shell-Remote-code-execution-via-image-processing-in-Roundcube-shipped-with-Plesk-for-Linux |
| **AlmaLinux**, "Upload2Shell (CVE-2026-125396): Patched Ghostscript released" (updated 2026-11-23) | The fixed AlmaLinux 8, 9 and 10 package versions, the restart advice, the container point, and that it knew of no fixed Red Hat packages on 2026-11-22 | https://almalinux.org/blog/2026-11-22-upload2shell-cve-2026-125396 |
| **Debian Security Tracker**, entry for CVE-2026-125396 | The description of the flaw, as a second source for the identifier | https://security-tracker.debian.org/tracker/CVE-2026-125396 |
| **Sophos**, its blog post on the Ghostscript Upload2Shell flaw (after 2026-11-21) | That there were no reports of active attacks and that a working demonstration exists | https://www.sophos.com/en-us/blog/proof-of-concept-available-for-ghostscript-upload2shell |
| **CISA**, "CISA Adds Four Known Exploited Vulnerabilities to Catalog" (2026-11-20) | That the flaw was not among the four added that day | https://www.cisa.gov/news-events/alerts/2026/11/20/cisa-adds-four-known-exploited-vulnerabilities-catalog |

## Further detail (linked, not reproduced)

- **The researchers' package tables.** pwn.ai lists fixed Ghostscript package versions for Ubuntu, Debian and other
  distributions and the vendor trackers still open as of 2026-11-12. They change quickly: use the link above and your
  vendor's own tracker.
- **The upstream bug and fix.** AlmaLinux names the upstream fix as commit `1484854e0c8d` and the Artifex bug as 709684. We
  have not read them.

## What we left out on purpose

- **Attacker addresses, hashes, domains and key fingerprints.** None have been published, so none are in the dataset.
- **Exploit details.** The researchers' write-up describes how the sandbox is bypassed and includes a working demonstration.
  We copy neither the method nor the code, and we do not link to exploit repositories.
- **Detection rules.** We found none we could use under a clear licence.
