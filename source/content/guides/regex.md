# Regex One-Liners for Linux
A practical, readable reference for `grep`, `sed`, `awk`, `cut`, `sort`, and `uniq`.

This guide is structured by **skill level**, then finishes with **real-world log analysis recipes**.
Each entry explains **what problem the command solves**, followed by:
- a **template** you can adapt
- a **real-world example** you can copy

Designed to be **beginner-friendly**, while letting experienced users **copy-paste and move on**.

---

## How to Think About These Tools (Mental Model)

- `grep` → **find lines**
- `sed` → **change lines**
- `awk` → **understand columns and apply logic**
- `cut` → **fast, simple field extraction**
- `sort | uniq` → **organize, count, and rank data**

If you feel stuck, start with `grep`, then layer tools one at a time.

---

## 🟢 Beginner — Everyday Text Searching

### Find a literal word
Searches for lines containing an exact string.
Great for first-pass searching in files or logs.

**Template**
```bash
grep "PATTERN" file
```

**Example**
```bash
grep "error" app.log
```

---

### Case-insensitive search
Matches text regardless of capitalization.
Useful when logs or user input are inconsistent.

**Template**
```bash
grep -i "PATTERN" file
```

**Example**
```bash
grep -i "warning" app.log
```

---

### Match whole words only
Avoids matching substrings inside larger words.
Prevents false positives.

**Template**
```bash
grep "\bWORD\b" file
```

**Example**
```bash
grep "\bcat\b" notes.txt
```

---

### Show line numbers
Adds line numbers to matches.
Helpful when debugging or editing files.

**Template**
```bash
grep -n "PATTERN" file
```

**Example**
```bash
grep -n "TODO" project.txt
```

---

### Use extended regex
Enables richer patterns like alternation without heavy escaping.
This is commonly what people *mean* when they say “regex”.

**Template**
```bash
grep -E "PATTERN" file
```

**Example**
```bash
grep -E "error|warning|failed" app.log
```

---

## 🟡 Intermediate — Transforming & Structuring Data

### Replace text (preview)
Replaces the first occurrence per line.
Good for testing changes before committing them.

**Template**
```bash
sed 's/OLD/NEW/' file
```

**Example**
```bash
sed 's/foo/bar/' config.txt
```

---

### Replace text everywhere
Replaces all occurrences per line.
Common for cleanup and normalization.

**Template**
```bash
sed 's/OLD/NEW/g' file
```

**Example**
```bash
sed 's/,/;/g' data.csv
```

---

### Delete unwanted lines
Removes lines matching a pattern entirely.
Great for stripping comments or noise.

**Template**
```bash
sed '/PATTERN/d' file
```

**Example**
```bash
sed '/^#/d' config.ini
```

---

### Extract a column
Pulls a specific field from whitespace-separated output.
Often used with command pipelines.

**Template**
```bash
awk '{print $N}' file
```

**Example**
```bash
awk '{print $1}' users.txt
```

---

### Use a delimiter (CSV / logs)
Splits data using a custom delimiter.
Essential for structured data formats.

**Template**
```bash
awk -F'DELIM' '{print $N}' file
```

**Example**
```bash
awk -F',' '{print $2}' data.csv
```

---

### Conditional filtering
Filters rows using numeric or string comparisons.
Lets you apply logic inline.

**Template**
```bash
awk '$N > VALUE' file
```

**Example**
```bash
awk '$3 > 100' scores.txt
```

---

### Extract CSV fields quickly
Fast field extraction without full parsing logic.
Prefer `awk` when spacing is inconsistent.

**Template**
```bash
cut -d',' -fCOL file.csv
```

**Example**
```bash
cut -d',' -f1,3 data.csv
```

---

### Count unique values
Counts how many times each line appears.
Common for summaries and frequency analysis.

**Template**
```bash
sort file | uniq -c
```

**Example**
```bash
sort ips.txt | uniq -c
```

> Common pitfall:  
> `uniq` only removes **adjacent** duplicates. Use `sort | uniq` unless your data is already sorted.

---

## 🔴 Log Analysis — Real-World Power Pipelines

### How to Read Pipelines
Read pipelines **left → right**.
Each command narrows, reshapes, or summarizes the data before passing it on.

---

## Nginx / Apache Access Logs

### Top IPs hitting your server
Identifies the most frequent clients.
Useful for traffic analysis or abuse detection.

**Template**
```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head
```

**Example**
```bash
grep "GET" access.log | awk '{print $1}' | sort | uniq -c | sort -nr | head -20
```

---

### Most requested endpoints
Finds which URLs are hit most often.

**Template**
```bash
awk '{print $7}' access.log | sort | uniq -c | sort -nr
```

**Example**
```bash
awk '{print $7}' access.log | sort | uniq -c | sort -nr | head
```

---

## systemd / journalctl Logs

### View errors only
Filters system logs to error-level messages.

**Template**
```bash
journalctl | grep "error"
```

**Example**
```bash
journalctl -xe | grep -i "failed"
```

---

### Count repeated failures
Detects recurring issues in services.

**Template**
```bash
journalctl | grep "failed" | sort | uniq -c | sort -nr
```

**Example**
```bash
journalctl -u ssh | grep "Failed" | sort | uniq -c | sort -nr
```

---

## Docker Logs

### Inspect container errors
Extracts error messages from a container’s logs.

**Template**
```bash
docker logs CONTAINER | grep "ERROR"
```

**Example**
```bash
docker logs web-app | grep -i "exception"
```

---

### Most common error messages
Ranks repeated failures for faster diagnosis.

**Template**
```bash
docker logs CONTAINER | grep "ERROR" | sort | uniq -c | sort -nr
```

**Example**
```bash
docker logs api | grep -i "error" | sort | uniq -c | sort -nr | head
```

---

## Regex Quick Reference

| Symbol | Meaning |
|------|--------|
| `.` | Any single character |
| `*` | 0 or more of previous token |
| `+` | 1 or more of previous token |
| `?` | 0 or 1 of previous token |
| `^` | Start of line |
| `$` | End of line |
| `[]` | Character class |
| `()` | Grouping |

---

## Pro Tip
Test regex instantly without touching files:
```bash
echo "test string" | grep "REGEX"
```
