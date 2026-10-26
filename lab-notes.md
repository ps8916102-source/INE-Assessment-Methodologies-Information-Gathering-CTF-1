# Detailed Lab Notes — Information Gathering CTF 1

> **Course:** Assessment Methodologies  
> **Exercise:** Information Gathering CTF 1 — Skill Check Lab  
> **Target:** `http://target.ine.local`  
> **Scope:** Authorized INE training environment only  
> **Safety:** Actual flag values and sensitive configuration values are intentionally redacted from this public-facing draft.

## 1. Lab goal

The lab asks the learner to perform website reconnaissance and locate five flags:

1. A file that tells search engines what they should or should not crawl.
2. The website platform and its version.
3. Information exposed through directory browsing.
4. An overlooked backup file that may reveal configuration details.
5. Interesting information found after mirroring the site.

The lab brief lists Firefox, `curl`, and HTTrack as the tools. The supplied transcript also contains DIRB and WPScan output. Because the official task lists a limited tool set, check the active lab instructions before using tools beyond that list.

---

## 2. Flag 1 — Inspecting `robots.txt`

### Objective

The clue — “This tells search engines what to and what not to avoid” — points to `robots.txt`, a text file commonly used to publish crawl directives for cooperative search-engine crawlers.

### Actual method

The learner opened the target in Firefox and navigated directly to:

`http://target.ine.local/robots.txt`

This browser-based method is the one to document. Do not replace it with a terminal command that was not used for this flag.

### Why this matters

A `robots.txt` file may reveal paths that a site owner prefers crawlers not to visit. Its directives are not an access-control mechanism: a path mentioned in the file is not automatically protected, private, or inaccessible to users.

### Evidence limitation

The supplied command transcript shows that `/robots.txt` returned HTTP 200 during the DIRB scan and that WPScan found the file. However, the transcript does not include the file's actual contents. Add a screenshot or sanitized excerpt from your browser if you want the final publication to show the exact evidence.


---

## 3. Flag 2 — Identifying WordPress and its version

### Command used

```bash
curl -s http://target.ine.local | grep -iE "generetor|wordpress"
```

### Command breakdown

- `curl` makes an HTTP request and writes the response body to standard output.
- `-s` enables silent mode, suppressing the progress meter and many non-essential messages.
- `http://target.ine.local` is the lab target URL.
- `|` sends the output from `curl` into `grep`.
- `grep` searches the received HTML for matching text.
- `-i` makes matching case-insensitive.
- `-E` enables extended regular expressions.
- `"generetor|wordpress"` matches either the literal misspelling `generetor` or `wordpress`, regardless of case.

### Observed result

The output included a generator meta tag identifying **WordPress 6.5.3**. Other returned HTML referenced WordPress assets with the same version string.

### Interpretation

The generator metadata was direct evidence of the platform and its reported version. The pattern contains a typo (`generetor` rather than `generator`), but the command still returned the relevant line because the alternative `wordpress` also matched the tag's content.

A clearer version for a future repeat run would be:

```bash
curl -s http://target.ine.local | grep -iE 'generator|wordpress'
```

This improved command is shown as a readability correction, not as a claim that it was the original command used. The original command and its output should remain part of the historical account.

---

## 4. Flag 3 — Investigating the uploads directory

### Browser path used

The learner opened this directory directly in Firefox:

`http://target.ine.local/wp-content/uploads/`

This direct browser inspection is important to preserve in the write-up; it was not merely a terminal-only discovery.

### Supporting observations in the supplied transcript

The DIRB output recorded `/wp-content/uploads/` as a directory and reported that the directory was listable. WPScan also reported that listing was enabled for the uploads directory.

### Why it matters

WordPress commonly stores media uploaded to a site under `wp-content/uploads/`. If directory indexing is enabled by the web server, a visitor may see a listing of files rather than only receiving content through the site's normal pages. Depending on what has been uploaded, this can expose unintended information.

Directory listing is the misconfiguration; it does not, by itself, prove that every listed file is sensitive or vulnerable.

### Evidence limitation

The supplied transcript does not include the actual browser listing or the names and contents of the files visible there. Do not invent file names. Add your original screenshot or notes if you want to document exactly what led to the flag.

---

## 5. Flag 4 — Finding and inspecting a backup configuration file

### Discovery evidence

The supplied WPScan output identified this publicly reachable path:

`http://target.ine.local/wp-config.bak`

The notes show that a file named `wp-config.bak` was present in the Kali `Downloads` directory and was inspected locally. The transcript does not show the exact download command or browser action, so this write-up does not claim a specific transfer method.

### What the file contained

The file resembled a WordPress configuration file and included database settings and WordPress authentication keys/salts. Those values are sensitive and have been intentionally excluded from this document.

The lab transcript also records a flag comment inside the backup file; its actual value is not reproduced here.

### Why this matters

A backup copy of a configuration file can disclose information that should never be publicly downloadable, including database connection details and authentication secrets. Even when a database password appears weak or looks like a lab placeholder, exposed configuration should be treated as sensitive.

### Safe handling

- Do not publish the actual database username/password or WordPress keys/salts.
- If a real system exposes such a file, restrict access, remove the exposed copy, rotate affected credentials and secrets, and review logs for access.
- Avoid placing unredacted configuration files or screenshots in a public repository.

---

## 6. Flag 5 — Mirroring the site with HTTrack

### Tool and workflow

HTTrack was started interactively:

```text
httrack
```

The wizard was configured with:
- Project name: `target`
- Default base path: `/root/websites/`
- Target: `target.ine.local`
- Action: `1` — Mirror Web Site(s)
- Default proxy, wildcard, and additional-options fields

The resulting wizard command showed the mirror being saved under `/root/websites/target`.

### Command breakdown

- `httrack` starts the website copier's interactive wizard.
- The project name labels the mirror project.
- The base path controls where the downloaded site is stored.
- The URL identifies the authorized lab target.
- “Mirror Web Site(s)” tells HTTrack to download pages and linked resources within the selected scope, subject to its settings and the site's links.

The transcript includes a warning that HTTrack was run as root. For general practice, prefer running content-retrieval tools as a non-root user when root privileges are unnecessary.

### Search the local mirror

After mirroring, the learner navigated to the output directory and ran:

```bash
grep -r "FLAG"
```

### Command breakdown

- `grep` searches text for a pattern.
- `-r` searches recursively through files in the current directory and its subdirectories.
- `"FLAG"` is the case-sensitive text pattern being searched for.
- With no path supplied, `grep` searches from the current working directory.

### Observed result

The search returned a flag marker in a mirrored file named `xmlrpc0db0.php` under the mirrored target directory. The actual flag value is intentionally omitted.

### Interpretation

The mirror made it possible to search downloaded content locally instead of relying only on what appeared during ordinary browsing. This is useful when relevant strings occur in files that are linked indirectly or are easy to overlook during manual review.

A recursive text search only finds text matching the specified pattern in files that were successfully downloaded and are readable as text. It does not guarantee that every remote file was mirrored or that every relevant value contains the searched string.

---

## 7. Consolidated methodology

The investigation can be understood as a sequence:

1. Inspect the crawler guidance file in Firefox.
2. Examine the returned page HTML to identify WordPress and its version.
3. Browse the uploads directory and evaluate whether directory listing exposes file names.
4. Investigate the identified backup configuration file while keeping its sensitive contents private.
5. Mirror the authorized site and search the local copy for relevant markers.

The sequence demonstrates the value of combining manual browser inspection, HTTP response inspection, directory observations, configuration-file review, and offline analysis.

## 8. Defensive lessons

- **Do not rely on `robots.txt` for access control.** Enforce authorization at the server or application layer.
- **Reduce unnecessary version disclosure.** Keep the CMS, themes, and components updated; do not assume hiding a version is a substitute for patching.
- **Disable unintended directory indexing.** Review web-server configuration and check whether uploaded content should be publicly enumerable.
- **Prevent backup exposure.** Keep configuration backups outside the webroot or deny public access, and remove stale copies.
- **Protect secrets.** Rotate credentials and authentication keys/salts if exposed, and assess whether any unauthorized access occurred.
- **Review public assets and mirrored content.** Treat unexpected files as potential information disclosures and investigate their origin.
- **Use least privilege.** Run local tools without unnecessary root privileges.

## 9. Publication checklist

- [ ] Remove all actual flag values.
- [ ] Redact database credentials and WordPress keys/salts.
- [ ] Review screenshots for cookies, tokens, session data, and private lab details.
- [ ] Keep conclusions tied to the evidence actually collected.
- [ ] Check INE's current rules for publishing Skill Check solutions.
- [ ] Use only tools permitted by the active lab instructions when reproducing the exercise.

## 10. Final takeaway

This lab reinforced that information gathering is more than running a scanner. Useful discoveries can come from a small text file, HTML metadata, an exposed directory listing, a forgotten backup, or a local copy of the site. The key skill is to connect each clue to verifiable evidence, understand what that evidence does and does not prove, and document the result responsibly.
