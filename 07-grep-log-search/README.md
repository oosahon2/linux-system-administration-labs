# Linux Day 7 — Searching Logs with grep

## Lab purpose

I practiced finding useful information in text files with `grep`. In a simulated support investigation, I searched a small application log, located and counted error entries, and saved those entries into a separate evidence file.

**Environment:** Ubuntu in Windows Subsystem for Linux (WSL), using Windows Terminal.  
**Scope:** This was a text-search exercise using a manually written practice log. The error messages were sample text, not evidence of an actual database or backup failure on my computer.

## Verified results

| Check | Observed result |
|---|---|
| Search for `bobby` in `/etc/passwd` | Found the local account entry |
| Search for uppercase `BOBBY` without `-i` | No matching output |
| Search for uppercase `BOBBY` with `-i` | Found the lowercase account entry |
| Account line number | 27 in this lab's account file |
| Error lines, ignoring capitalization | Lines 2 and 4 |
| Error line count | 2 |
| Lines without `error`, ignoring capitalization | Three INFO lines |
| Search for `bobby` in the practice log | Line 3 |
| Saved error file | Contained both error lines |
| Original log after extraction | Still contained all five lines |
| INFO line count | 3 |

Line numbers depend on the contents of each file and may differ on another system.

## 1. Search the local account file

```bash
grep bobby /etc/passwd
```

Observed output:

```text
bobby:x:1000:1000::/home/bobby:/bin/bash
```

`grep` prints the entire matching line. This search matches `bobby` anywhere in the line; it does not restrict the search to the username field.

The colon-separated fields identify the username, password placeholder, user ID, primary group ID, account description (empty here), home folder, and login shell. `x` indicates that password information is stored separately; it is not the password.

I compared a case-sensitive search with one that ignores capitalization:

```bash
grep BOBBY /etc/passwd
grep -i BOBBY /etc/passwd
```

The first search had no matching output. The second found the account entry.

I displayed its line number and combined options:

```bash
grep -n bobby /etc/passwd
grep -in BOBBY /etc/passwd
```

Both numbered searches located the entry on line 27.

![Account search and capitalization comparison](images/01-account-case-search.jpeg)

## 2. Create a practice application log

```bash
nano ~/day7-app.log
```

I entered these five lines, saved with Ctrl+O and Enter, and exited with Ctrl+X:

```text
INFO Application started
ERROR Database connection failed
INFO User bobby logged in
error Backup failed
INFO Application stopped
```

`~` means my home directory. Nano is a text editor. A log records events; this practice log was written manually so I could learn how to search it. Real applications commonly write their own logs.

## 3. Search by message text

```bash
grep ERROR ~/day7-app.log
grep INFO ~/day7-app.log
grep error ~/day7-app.log
```

The uppercase `ERROR` search found the database message. The lowercase `error` search found the backup message. The `INFO` search found three lines.

![Search ERROR, INFO, and error in the practice log](images/02-practice-log-search.jpeg)

## 4. Find both error entries and their locations

```bash
grep -in error ~/day7-app.log
```

Observed output:

```text
2:ERROR Database connection failed
4:error Backup failed
```

`-i` ignores capitalization. `-n` prefixes matches with their original file line numbers.

![Both error lines with their line numbers](images/03-errors-with-line-numbers.jpeg)

## 5. Count matches and display nonmatching lines

```bash
grep -ic error ~/day7-app.log
```

Observed output: `2`.

`-c` counts matching lines. Multiple occurrences on one line still count as one matching line.

```bash
grep -iv error ~/day7-app.log
```

Observed output:

```text
INFO Application started
INFO User bobby logged in
INFO Application stopped
```

`-v` displays lines that do not match. It filters the displayed results without deleting file contents.

![Counting error lines and excluding error lines](images/04-count-and-exclude.jpeg)

## 6. Locate a user's log entry

```bash
grep bobby ~/day7-app.log
grep -in bobby ~/day7-app.log
```

The numbered search returned `3:INFO User bobby logged in`.

## 7. Save matching lines as evidence

```bash
grep -i error ~/day7-app.log > ~/day7-errors.txt
cat ~/day7-errors.txt
```

Observed saved contents:

```text
ERROR Database connection failed
error Backup failed
```

The shell's `>` operator sends the command's output into a file. It creates the destination or replaces its existing contents. The search results therefore did not print to the terminal during redirection. `cat` displayed the saved text afterward.

![Extracted error entries saved and displayed](images/05-save-errors.jpeg)

I checked the original separately:

```bash
cat ~/day7-app.log
```

All five original lines remained.

![Original five-line log remains intact](images/06-original-log-preserved.jpeg)

## 8. Final practice: count INFO entries

```bash
grep -c INFO ~/day7-app.log
```

Observed output: `3`.

I initially used `-n`, which correctly showed the matching line positions (1, 3, and 5), then switched to `-c` to get the requested count.

![Final INFO count is three](images/07-info-count.jpeg)

## Troubleshooting during the lab

- I accidentally typed `-i-`, which produced an invalid-option error. Retyping the option as `-i` fixed the command.
- A search with no matching lines normally prints nothing. An explicit error such as “No such file or directory” instead indicates a problem reading the specified file.
- I distinguished `-n` (where matches occur) from `-c` (how many lines match).

## Notes to remember

| Command or option | Purpose |
|---|---|
| `grep TEXT FILE` | Print lines matching the search text/pattern |
| `-i` | Ignore uppercase/lowercase differences |
| `-n` | Show original file line numbers |
| `-c` | Count matching lines |
| `-v` | Select nonmatching lines |
| `-in` | Combine case-insensitive search and line numbers |
| `>` | Redirect output to a file, replacing existing contents |
| `cat FILE` | Display a file's contents |
| `nano FILE` | Open a text file for editing |
| `~` | Refer to my home directory |

By default, grep uses search patterns; the simple words in this lab work as ordinary text searches.

## Support investigation takeaway

I can search a log for relevant messages or usernames, locate matching entries, count them, and save selected entries for a support ticket. Finding an error message provides evidence for investigation; it does not establish the root cause or fix the application.

## Included practice examples

- [Original practice log](examples/day7-app.log)
- [Extracted error lines](examples/day7-errors.txt)

These two example files were recreated from the text and terminal output recorded in the lab screenshots. The screenshots are the evidence of the commands run in my Ubuntu environment.
