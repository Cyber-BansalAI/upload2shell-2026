# Hunt guide: can a crafted file reach Ghostscript on our servers, and did one?

As of **2026-11-24**. Read [DISCLAIMER.md](../DISCLAIMER.md) first. We have not tested these commands on your systems. Run them
yourself, on copies of the logs where you can, and adjust names and paths to your setup. Nothing in this package runs anything
for you, and every command shown only reads.

**Who this is for.** The person who runs a website, a webmail server or a print server for a business: an IT lead, or the
provider that does it for them. The business-owner version, with five questions to put to that person, is in
[our blog post](https://blog.bansalai.com/news/upload2shell-ghostscript-uploads-what-to-ask/?ref=github).

## What happened, in five lines

- Ghostscript, a free program that converts PostScript and PDF files, has a flaw in the part that loads fonts
  (CVE-2026-125396, nicknamed Upload2Shell). A crafted file can get out of Ghostscript's `-dSAFER` sandbox and run commands as
  the account that rendered it.
- Artifex fixed it in Ghostscript **10.08.0 on 2026-09-08**. The flaw got its public number on 2026-11-09. The researchers at
  pwn.ai wrote it up in November 2026.
- A file does not need a PostScript name. The researchers' files were named like photos or PDFs, and the image tool passed
  them on because of what was inside. They showed it working through a shared print queue, Roundcube webmail, and WordPress
  uploads (core and two plugins), among others.
- **No attacker indicators have been published.** Sophos, writing after pwn.ai described a working demonstration on
  2026-11-21, said there were no reports of active attacks. The flaw was not among the ones CISA added to its Known Exploited
  Vulnerabilities catalog on 2026-11-20. A working demonstration is public, so this can change.
- So this guide has two jobs: find where Ghostscript can be reached, and look for traces of the kind of activity the
  researchers showed. A clean result is not proof.

## Two questions, kept apart

1. *Are we exposed?* Answered from package versions, settings and the list of tools that hand files to Ghostscript.
2. *Did anything use it?* Answered from the servers: odd files in upload folders, Ghostscript run by the wrong account or with
   the wrong child process, print-job records. There is no published log signature, so this half is weaker.

## 0. Before you touch anything

Our reasoning, not a vendor's: **copy the evidence first, then update.** Updating Ghostscript is quick and needs no reboot, but
restarting the services that use it can clear in-memory state and rotate logs.

1. Copy the web, mail and print logs you can reach (section 4) to somewhere the server cannot write to.
2. Note the date of the oldest log you have. The fix existed from 2026-09-08 and the details were public in November, so
   search from 2026-08-01 if your records go back that far. If they do not, write that down.
3. Update, then restart (section 1), then look (sections 3 to 5).

## 1. Where is Ghostscript, and is it fixed?

Find every copy. The host package is one place; containers and application bundles can carry their own.

```sh
gs --version
which -a gs
```

Package versions on Debian or Ubuntu, and on Red Hat family systems:

```sh
dpkg -l | grep -i ghostscript
rpm -q ghostscript libgs
```

Fixed versions we could read on the page itself:

| You run | Fixed version | Note |
|---|---|---|
| Ghostscript from the project | 10.08.0 | Released 2026-09-08 |
| AlmaLinux 8 | ghostscript-9.27-20.el8_10.alma.1 | In production repositories from 2026-11-23 |
| AlmaLinux 9 | ghostscript-9.54.0-19.el9_8.3.alma.1 | Same |
| AlmaLinux 10 | ghostscript-10.02.1-16.el10_2.3.alma.1 | Same |
| Plesk for Linux | Plesk Obsidian 18.0.81 or later | Fixes the bundled Roundcube only. Plesk says it does not update Ghostscript |
| cPanel | Roundcube 1.6.20 or later; WordPress 6.8.8, 6.9.7 or 7.0.4 or later | cPanel's own article. Ghostscript still comes from the operating system |
| Ubuntu, Debian, SUSE, Amazon Linux, Red Hat family | Read your vendor's tracker for CVE-2026-125396 | See below |

For the rows we did not read ourselves, the researchers' tables are the best list we have: as of 2026-11-12 they show Ubuntu
fixed (notice USN-8949-1), Debian 13 fixed, **Debian 12 still listed vulnerable**, Amazon Linux fix pending, and several SUSE
lines in progress. They say they did not establish whether current Red Hat family packages hold the fix, and AlmaLinux wrote
on 2026-11-22 that it knew of no fixed Red Hat packages. Those statuses change quickly: **check the vendor's tracker today.**
A backported fix can keep an old version number, so read the package changelog for the CVE.

After the update:

- Restart every service that loaded Ghostscript's library. Running programs keep the old copy until they restart.
  AlmaLinux says this command lists them:

```sh
sudo dnf needs-restarting
```

- Containers and application bundles: the host update does not reach them. List the running containers and ask for each
  image's package list from whoever builds it.

```sh
docker ps
```

## 2. Which routes reach Ghostscript

A route needs an application that hands an uploaded, mailed or printed file to an image tool, and an image tool that passes it
to Ghostscript. Check each one you run.

**ImageMagick.** This is the common middle step. Its policy file decides which formats it will hand on. cPanel's temporary
measure adds lines that disable the `gs` delegate and the PS, PS2, PS3, EPS, PDF and XPS coders. See what yours says:

```sh
grep -n -i -E "gs|PS|EPS|PDF|XPS" /etc/ImageMagick-*/policy.xml
```

If nothing disables those, ImageMagick may hand such files to Ghostscript. (We have not tested cPanel's lines and cannot say
they are enough. They can break previews and thumbnails.)

**WordPress.** Both conditions must hold: Imagick and Ghostscript on the server, and an account that can upload files (Author
or higher). Read the version and who can upload, and the plugins:

```sh
php -m
wp core version
wp user list --fields=ID,user_login,roles
wp plugin list --fields=name,status,version
```

Look for `imagick` in the `php -m` list. Look for public upload forms: the researchers tested the plugins Forminator (a public
PDF field that stores files in the Media Library) and Essential Addons for Elementor (a public registration widget with an
upload field). The flaw is in Ghostscript, not in those plugins: a public upload form just lets a visitor with no login reach
the renderer. Your own form builder may do the same.

**Roundcube webmail.** The researchers' proof needed only that the recipient open the message, and a permissive ImageMagick
setting. Fixed in Roundcube 1.6.20 and 1.7.5. On cPanel:

```sh
rpm -q cpanel-roundcubemail
```

**Print servers (CUPS).** A shared queue that accepts jobs without a login, reachable from networks you do not trust, is the
route in the researchers' network proof. Read the settings and what is listening:

```sh
lpstat -t
cupsctl
ss -ltn
```

In `cupsctl` output, a share setting of 1 means printers are shared. Ask whoever runs the firewall which networks can reach
the print port, and whether the queue requires a login.

**Other tools.** The researchers also reached Ghostscript through a wiki, a document-management system and several image
libraries. Their finding is that an application does not need to call Ghostscript itself: any step that thumbnails, resizes
or converts an uploaded file may. Some image decoders never start it, so check how your tools do the conversion.

## 3. Uploaded files that are not what their name says

The researchers' files were PostScript saved under a picture or PDF name. Search upload folders for files with picture names
whose contents start like PostScript. Replace the path with yours. These only read:

```sh
grep -rla --include="*.jpg" --include="*.jpeg" --include="*.png" -e "%!PS" /var/www
```

How to read it:

- A file with a picture name that holds `%!PS` is a strong lead. Keep it as it is and do not open it with an image viewer.
- A genuine PDF can embed font data that starts the same way, so we did not search PDFs here. If you must, expect false hits and
  check each one by hand.
- The signature can have bytes or a comment before it, so a miss proves little. We took the first-line signature from the
  file formats, not from a source: the researchers say only that the files were PostScript.

For webmail, we found no published method for searching stored messages. Ask the mail administrator whether the mail store
can be searched for image attachments whose contents are PostScript.

## 4. Process and log traces

There is **no published log signature** for this flaw. What the researchers describe is the shape of the result: Ghostscript
running as a service account, then a command running as that same account.

Currently running (read only):

```sh
ps -eo pid,ppid,user,args
```

Look for `gs` started by `www-data`, a mail account or `lp`, and any shell whose parent is `gs`. Starting `gs` this way is
normal for thumbnails and print jobs: the signs are odd timing, odd volume, or a shell as its child.

Past activity needs data you already collect. If the audit service records program starts, or you have an endpoint product,
search it from 2026-08-01 for `gs` and its children:

```sh
ausearch -c gs -ts 08/01/2026
```

(We did not test this on your systems. An empty result means nothing was recorded, not that nothing happened.)

CUPS keeps an access log. On Debian and Ubuntu it is usually here (check your distribution):

```sh
grep -n "Print-Job" /var/log/cups/access_log
```

Look for print jobs from addresses that should not be printing, and for jobs sent to a shared queue from outside your
network. Completed jobs and their senders are listed by:

```sh
lpstat -W completed -o
```

## 5. What the results mean

| You found | It suggests | Next step |
|---|---|---|
| Ghostscript below the fixed version, and a route from section 2 | Exposed, no sign of use | Update and restart (section 1), then keep looking |
| A shared print queue open to untrusted networks, no login | The researchers' network route | Restrict it, then update |
| A file with a picture name that starts with a PostScript signature | A crafted file was uploaded or mailed | Keep it, check who sent it and when, treat as an incident |
| `gs` run by a service account with a shell as its child | Possible command run through Ghostscript | Treat as an incident |
| Nothing | Not proof of safety: no published signature, and your logs may not record this | Fix the basics in section 7 anyway |

## 6. If you think it is real

Preserve first, change later:

1. Keep the logs, the suspect file and the server as they are. Do not rebuild yet.
2. Call your IT provider or an incident-response firm. Do not use a machine that may be affected to talk about it.
3. Report to CISA and to the FBI's Internet Crime Complaint Center through their websites.
4. Then, in this order. **This order is our reasoning, not a vendor's:** the sources we read give update advice but no
   compromise-response steps for this flaw.
   1. Update Ghostscript and restart what uses it, so the same file cannot be used again.
   2. Assume anything the rendering account can read may be exposed. The researchers note that the account can read
      application files, write to disk and start other programs. Rotate the passwords and keys that account could reach
      (for a website, the database login and the keys in its configuration), from a clean machine.
   3. Look for new accounts, new files in the web folders and new scheduled tasks. Treat a server where a command ran as one
      you may need to rebuild from a known-good copy.

## 7. Reduce the risk

From the researchers' and vendors' guidance. Read them for the full list.

- Update Ghostscript everywhere it runs, including inside containers and bundles, and restart what uses it.
- Keep WordPress, Roundcube and your control panel on fixed versions. Those fixes close routes into Ghostscript. They do not
  replace updating Ghostscript.
- Require a login for shared print queues, and do not share them with networks you do not trust.
- If you do not need PostScript, EPS or PDF conversion in ImageMagick, disable those formats in its policy. Test first.
- Check a file's real type before it is thumbnailed or converted, not after. Turn off previews for document types you do not
  need.
- The researchers add that a rule that stops the rendering account from starting a shell can limit the damage even after
  Ghostscript is compromised. Run converters under an account with as little access as possible.

## What we could not confirm

- Attacker addresses, file hashes or domains. None have been published, so none are in `iocs/iocs.csv`.
- Whether anyone has used the flaw. Sophos reported no active attacks after 2026-11-21. That can change.
- A log signature for the attack. We found none in any source.
- The fixed package versions for Ubuntu, Debian, SUSE, Amazon Linux and the Red Hat family. We link to the researchers' tables
  and the vendors' trackers instead of copying them.
- Whether cPanel's temporary policy lines are enough.
- Whether Windows or macOS Ghostscript copies are reached by the same routes. The researchers list fixed packages for them,
  not tested routes.

## Not sure what you found?

Bring the log lines to your IT person first. If you want a second pair of eyes,
[SOC-Lite by BansalAI](https://cyber.bansalai.com/?ref=github) offers a free 20-minute
[visibility check](https://cyber.bansalai.com/?ref=github#book).

## Sources for this guide

pwn.ai, "How We Hacked the Pipes: Upload2Shell" (November 2026); Sophos, its blog post on the Ghostscript Upload2Shell
flaw (after 2026-11-21); CISA, "CISA Adds Four Known Exploited Vulnerabilities to Catalog" (2026-11-20);
WordPress, "WordPress 7.0.4 Release" (2026-08-12); Roundcube, "Security updates 1.6.20 and 1.7.5 released" (2026-10-19);
cPanel and Plesk support articles on CVE-2026-125396; AlmaLinux, "Upload2Shell (CVE-2026-125396): Patched Ghostscript
released" (2026-11-23); the Debian Security Tracker entry. Full credit, links and licence notes are in
[CREDITS.md](../CREDITS.md). The shell commands are standard tools, not from the sources, and we have not run them on your
systems.
