# Linux Commands – `find`, `sed`, `zip`, `unzip`, `tar`, `zcat`, `zgrep`, `tee`

---

# 1. `find` Command

## Definition

The `find` command is used to search for files and directories in Linux based on different conditions such as:

* File name
* Directory name
* File type
* Permissions
* Modification time
* File size
* Empty files

## Basic Syntax

```bash
find <path> <conditions>
```

Example:

```bash
find . -name file.txt
```

Here:

* `.` → current directory
* `-name` → search based on name
* `file.txt` → file to search

---

## 1.1 Find Empty Files

### Current Directory

```bash
find . -type f -empty
```

Meaning:

* `.` → current directory
* `-type f` → files only
* `-empty` → zero-size/empty files

Example:

```bash
find . -type f -empty
```

---

### Home Directory

```bash
find ~ -type f -empty
```

Searches for empty files inside the current user's home directory.

---

### `/tmp` Directory

```bash
find /tmp -type f -empty
```

Searches for empty files inside `/tmp`.

---

# 2. Find Directory by Name

```bash
find . -type d -name "dirName"
```

Example:

```bash
find . -type d -name "backup"
```

This searches for directories named `backup`.

Here:

```text
-type d = directory
```

---

# 3. Find File by Name

```bash
find . -name "deployment.yaml"
```

This search is **case-sensitive**.

It will match:

```text
deployment.yaml
```

But may not match:

```text
Deployment.yaml
DEPLOYMENT.yaml
```

---

# 4. Case-Insensitive Search

```bash
find . -iname "deployment.yaml"
```

`-iname` ignores uppercase and lowercase differences.

It can match:

```text
deployment.yaml
Deployment.yaml
DEPLOYMENT.yaml
```

---

# 5. Find Files Based on Permissions

```bash
find . -type f -perm 0777
```

This finds files having exactly `777` permissions.

### 777 Means

```text
Owner  = rwx
Group  = rwx
Others = rwx
```

Example output:

```text
./script.sh
./test.sh
```

---

# 6. Find Files Based on Modification Time

Linux `find` provides the `-mtime` option.

## Modified Approximately 1 Day Ago

```bash
find . -mtime 1
```

Finds files whose modification time falls in the corresponding 24-hour `find` time bucket.

---

## Modified Less Than 1 Day Ago

```bash
find . -mtime -1
```

Useful for finding recently modified files.

Example:

```bash
find /var/log -type f -mtime -1
```

Finds files modified within the last 24 hours.

---

## Modified More Than 1 Day Ago

```bash
find . -mtime +1
```

Finds files older than the corresponding one-day threshold used by `find`.

Example:

```bash
find /tmp -type f -mtime +7
```

Find files older than 7 days.

---

# 7. Find Files Based on Extension

Find all `.txt` files:

```bash
find . -type f -name "*.txt"
```

Example output:

```text
./abc.txt
./notes.txt
./linux.txt
```

Find YAML files:

```bash
find . -type f -name "*.yaml"
```

Find log files:

```bash
find /var/log -type f -name "*.log"
```

---

# 8. Useful `find` Examples

Find `.log` files older than 7 days:

```bash
find /var/log -type f -name "*.log" -mtime +7
```

Find empty directories:

```bash
find . -type d -empty
```

Find files greater than 100 MB:

```bash
find . -type f -size +100M
```

Find files owned by a particular user:

```bash
find . -type f -user ubuntu
```

Find shell scripts:

```bash
find . -type f -name "*.sh"
```

---

# `sed` Command

## Definition

`sed` stands for:

```text
Stream Editor
```

It is mainly used to:

* Search text
* Replace text
* Delete lines
* Print selected lines
* Modify text streams
* Automate text processing

Basic syntax:

```bash
sed 'operation' filename
```

---

# 9. Replace First Occurrence

File:

```bash
cat abc.txt
```

Content:

```text
unix is good
I am learning unix
unix is powerful
```

Command:

```bash
sed 's/unix/linux/' abc.txt
```

Output:

```text
linux is good
I am learning linux
linux is powerful
```

Syntax:

```bash
sed 's/old/new/' filename
```

`s` means substitute.

By default, `sed` replaces only the **first matching occurrence on each line**.

---

# 10. Replace Second Occurrence

```bash
sed 's/unix/linux/2' abc.txt
```

Example input:

```text
unix unix unix
```

Output:

```text
unix linux unix
```

Only the second occurrence is replaced.

---

# 11. Replace All Occurrences

```bash
sed 's/unix/linux/g' abc.txt
```

`g` means:

```text
Global
```

Example input:

```text
unix unix unix
```

Output:

```text
linux linux linux
```

---

# 12. Replace Text on Specific Line

```bash
sed '3 s/unix/linux/' abc.txt
```

This performs the replacement only on line number `3`.

Example:

```text
1: unix
2: unix
3: unix
```

Result:

```text
unix
unix
linux
```

---

# 13. Print Replaced Line

```bash
sed '3 s/am/was/p' sed.txt
```

The `p` option prints the modified line.

But remember: without `-n`, `sed` also prints its normal output.

Because of this, the modified line may appear twice.

Example:

```bash
sed '3 s/am/was/p' sed.txt
```

---

# 14. Print Only Modified Lines

```bash
sed -n 's/am/was/p' sed.txt
```

Here:

```text
-n = suppress normal output
p  = print matching/replaced lines
```

Therefore only successfully replaced lines are displayed.

This is commonly used for filtering.

---

# 15. Replace Within a Line Range

```bash
sed '1,3 s/unix/linux/' abc.txt
```

This replaces `unix` with `linux` only between lines 1 and 3.

Example:

```text
Line 1 → replacement happens
Line 2 → replacement happens
Line 3 → replacement happens
Line 4 → no replacement
```

---

# 16. Delete a Specific Line

```bash
sed '5d' filename.txt
```

Deletes line number 5 from the displayed output.

Here:

```text
d = delete
```

Important:

By default, the original file is **not modified**.

---

# 17. Delete a Range of Lines

```bash
sed '3,6d' filename.txt
```

Deletes lines:

```text
3
4
5
6
```

---

# 18. Display Specific Lines

To display lines 60 to 80:

```bash
sed -n '60,80p' filename
```

Here:

```text
-n = suppress normal output
p  = print selected lines
```

Another example:

```bash
sed -n '1,10p' /etc/passwd
```

Displays the first 10 lines.

---

# 19. Modify Original File Using `sed`

Normally:

```bash
sed 's/unix/linux/g' abc.txt
```

does not permanently modify the file.

To modify the original file:

```bash
sed -i 's/unix/linux/g' abc.txt
```

Verify:

```bash
cat abc.txt
```

### Recommended Backup

```bash
sed -i.bak 's/unix/linux/g' abc.txt
```

This modifies `abc.txt` and creates a backup:

```text
abc.txt.bak
```

---

# `zip` Command

## Definition

`zip` is used to compress one or more files into a `.zip` archive.

Basic syntax:

```bash
zip archive.zip files
```

---

# 20. Compress a File

```bash
zip backup.zip abc.txt
```

Creates:

```text
backup.zip
```

---

# 21. Compress Multiple Files

```bash
zip backup.zip file1.txt file2.txt file3.txt
```

---

# 22. Compress a Directory

Use `-r` for recursive compression.

```bash
zip -r backup.zip mydir/
```

Example:

```bash
zip -r application.zip application/
```

---

# `unzip` Command

## Definition

`unzip` extracts files from a `.zip` archive.

Basic syntax:

```bash
unzip archive.zip
```

Example:

```bash
unzip backup.zip
```

---

# 23. Extract ZIP into Specific Directory

```bash
unzip backup.zip -d /tmp/backup
```

Here:

```text
-d = destination directory
```

---

# 24. View ZIP Content Without Extracting

```bash
unzip -l backup.zip
```

Useful to inspect files inside the archive.

---

# `tar` Command

## Definition

`tar` stands for:

```text
Tape Archive
```

It is commonly used to combine multiple files/directories into a single archive.

Typical extensions:

```text
.tar
.tar.gz
.tgz
.tar.bz2
```

---

# 25. Create TAR Archive

```bash
tar -cvf backup.tar mydir/
```

Options:

```text
-c = create
-v = verbose
-f = archive file
```

Example:

```bash
tar -cvf application.tar application/
```

---

# 26. Extract TAR Archive

```bash
tar -xvf backup.tar
```

Options:

```text
-x = extract
-v = verbose
-f = archive file
```

---

# 27. Create Compressed TAR.GZ Archive

```bash
tar -czvf backup.tar.gz mydir/
```

Here:

```text
-z = gzip compression
```

Example:

```bash
tar -czvf logs.tar.gz /var/log/
```

---

# 28. Extract TAR.GZ Archive

```bash
tar -xzvf backup.tar.gz
```

---

# 29. View TAR Archive Content

```bash
tar -tvf backup.tar
```

For `.tar.gz`:

```bash
tar -tzvf backup.tar.gz
```

---

# 30. Extract TAR into Specific Directory

```bash
tar -xzvf backup.tar.gz -C /tmp/
```

Here:

```text
-C = destination directory
```

---

# ZIP vs TAR

| Feature                  | ZIP                       | TAR                              |
| ------------------------ | ------------------------- | -------------------------------- |
| Create archive           | Yes                       | Yes                              |
| Compression              | Built in                  | Usually combined with gzip/bzip2 |
| Common extension         | `.zip`                    | `.tar`, `.tar.gz`                |
| Windows support          | Excellent                 | Good                             |
| Linux usage              | Common                    | Very common                      |
| Directory backup         | Yes                       | Yes                              |
| Permissions preservation | Limited compared with tar | Very good                        |

For Linux server backups, `tar.gz` is very commonly used.

---

# `zcat` Command

## Definition

`zcat` is used to display the content of a gzip-compressed file without manually extracting it first.

Example compressed file:

```text
application.log.gz
```

Command:

```bash
zcat application.log.gz
```

---

# 31. Read Compressed Log File

```bash
zcat access.log.gz
```

Instead of:

```bash
gunzip access.log.gz
cat access.log
```

you can directly use:

```bash
zcat access.log.gz
```

---

# 32. Read Compressed File with `less`

```bash
zcat application.log.gz | less
```

Useful for large compressed log files.

---

# `zgrep` Command

## Definition

`zgrep` is used to search inside gzip-compressed files.

It works similarly to the normal `grep` command.

Example:

```bash
zgrep "ERROR" application.log.gz
```

Searches for:

```text
ERROR
```

inside the compressed file.

---

# 33. Search Error Messages

```bash
zgrep "ERROR" application.log.gz
```

---

# 34. Case-Insensitive Search

```bash
zgrep -i "error" application.log.gz
```

This can match:

```text
error
ERROR
Error
```

---

# 35. Show Line Numbers

```bash
zgrep -n "ERROR" application.log.gz
```

Example:

```text
120:ERROR Database connection failed
```

---

# 36. Search Multiple Compressed Files

```bash
zgrep "ERROR" *.gz
```

Very useful when working with rotated logs.

Example:

```text
application.log.1.gz
application.log.2.gz
application.log.3.gz
```

Command:

```bash
zgrep "ERROR" application*.gz
```

---

# `tee` Command

## Definition

The `tee` command reads input from standard input and sends it to:

1. The screen
2. A file

at the same time.

Basic syntax:

```bash
command | tee filename
```

---

# 37. Save Command Output

```bash
ls -l | tee output.txt
```

The output is:

* Displayed on the terminal
* Stored in `output.txt`

---

# 38. Save `df` Output

```bash
df -h | tee disk.txt
```

Verify:

```bash
cat disk.txt
```

---

# 39. Append Output to Existing File

By default:

```bash
tee file.txt
```

overwrites the file.

To append:

```bash
tee -a file.txt
```

Example:

```bash
date | tee -a system.log
```

Here:

```text
-a = append
```

---

# 40. Use `tee` with Root Permissions

This command may fail:

```bash
sudo echo "hello" > /etc/test.conf
```

because shell redirection happens before `sudo`.

Instead use:

```bash
echo "hello" | sudo tee /etc/test.conf
```

For append:

```bash
echo "hello" | sudo tee -a /etc/test.conf
```

This is a very common Linux administration use case.

---

# Practical DevOps Examples

## Find Kubernetes YAML Files

```bash
find . -type f -name "*.yaml"
```

---

## Find `deployment.yaml`

```bash
find . -name "deployment.yaml"
```

---

## Search Without Case Sensitivity

```bash
find . -iname "deployment.yaml"
```

---

## Find Large Log Files

```bash
find /var/log -type f -size +100M
```

---

## Find Logs Older Than 7 Days

```bash
find /var/log -type f -mtime +7
```

---

## Replace Image Version in Kubernetes YAML

Before:

```yaml
image: nginx:1.24
```

Command:

```bash
sed -i 's/nginx:1.24/nginx:1.25/g' deployment.yaml
```

After:

```yaml
image: nginx:1.25
```

---

## Replace Environment Name

```bash
sed -i 's/dev/prod/g' deployment.yaml
```

---

## Create Application Backup

```bash
tar -czvf application-backup.tar.gz application/
```

---

## Extract Backup

```bash
tar -xzvf application-backup.tar.gz
```

---

## Search Errors in Old Compressed Logs

```bash
zgrep -i "error" application.log.2.gz
```

---

## Monitor and Save Output

```bash
ping google.com | tee ping-output.txt
```

Both the terminal and file receive the output.

---

# Important Commands – Quick Reference

```bash
# FIND

find . -type f -empty
find ~ -type f -empty
find /tmp -type f -empty
find . -type d -name "dirname"

find . -name "deployment.yaml"
find . -iname "deployment.yaml"

find . -type f -perm 0777

find . -mtime 1
find . -mtime -1
find . -mtime +1

find . -type f -name "*.txt"
find . -type f -size +100M


# SED

sed 's/unix/linux/' abc.txt
sed 's/unix/linux/2' abc.txt
sed 's/unix/linux/g' abc.txt

sed '3 s/unix/linux/' abc.txt
sed '3 s/am/was/p' sed.txt
sed -n 's/am/was/p' sed.txt

sed '1,3 s/unix/linux/' abc.txt

sed '5d' filename.txt
sed '3,6d' filename.txt

sed -n '60,80p' filename

sed -i 's/unix/linux/g' abc.txt


# ZIP

zip backup.zip file.txt
zip backup.zip file1.txt file2.txt
zip -r backup.zip directory/


# UNZIP

unzip backup.zip
unzip backup.zip -d /tmp/backup
unzip -l backup.zip


# TAR

tar -cvf backup.tar directory/
tar -xvf backup.tar

tar -czvf backup.tar.gz directory/
tar -xzvf backup.tar.gz

tar -tvf backup.tar
tar -tzvf backup.tar.gz


# ZCAT

zcat application.log.gz
zcat application.log.gz | less


# ZGREP

zgrep "ERROR" application.log.gz
zgrep -i "error" application.log.gz
zgrep -n "ERROR" application.log.gz
zgrep "ERROR" *.gz


# TEE

ls -l | tee output.txt
df -h | tee disk.txt

date | tee -a system.log

echo "hello" | sudo tee /etc/test.conf
echo "hello" | sudo tee -a /etc/test.conf
```

# Interview Points

**What is `find`?**
`find` searches files and directories based on name, type, size, permissions, modification time, and other conditions.

**Difference between `-name` and `-iname`?**

```text
-name  → Case-sensitive
-iname → Case-insensitive
```

**What is `sed`?**
`sed` is a stream editor used to search, replace, delete, print, and transform text.

**What does `g` mean in `sed`?**

```bash
sed 's/dev/prod/g' file.txt
```

`g` means replace all matching occurrences on each processed line.

**Difference between `sed` and `sed -i`?**

```text
sed     → Displays changed output
sed -i  → Modifies the original file
```

**What is `tar`?**
`tar` combines multiple files and directories into an archive and is commonly combined with gzip compression.

**Difference between `.tar` and `.tar.gz`?**

```text
.tar     → Archive
.tar.gz  → Archive compressed with gzip
```

**What is `zcat`?**
`zcat` displays gzip-compressed file contents without manually extracting the file.

**What is `zgrep`?**
`zgrep` searches text inside gzip-compressed files.

**What is `tee`?**
`tee` displays command output on the terminal while simultaneously writing the same output to a file.

# Easy Memory

```text
find   → Search files/directories
sed    → Search/replace/edit text
zip    → Compress files into ZIP
unzip  → Extract ZIP
tar    → Archive files/directories
zcat   → Read compressed file
zgrep  → Search compressed file
tee    → Screen + File output
```
