Unix/Linux  – Compact Cheat Sheet 
------------------------------------------------

### Navigation & listing

*   `pwd` — show current directory
    
*   `cd <dir>` — change directory
    
*   `cd ..` — go up one level
    
*   `cd ~` — go to home directory
    
*   `ls` — list files/folders
    
*   `ls -lh` — list with sizes (human-readable)
    
*   `ls -la` — list including hidden files
    
*   `ls -lah` — hidden + human sizes + details
    

* * *

### View files (safe / read-only)

*   `cat <file>` — print entire file (avoid for huge logs)
    
*   `less <file>` — scroll/view safely (`q` quit)
    
*   `head -n 20 <file>` — first 20 lines
    
*   `tail -n 20 <file>` — last 20 lines
    
*   `tail -n 200 <file>` — last 200 lines
    
*   `tail -f <file>` — follow live log updates
    
**Inside `less`**
*   `/text` — search forward
    
*   `n` — next match
    
*   `N` — previous match
    
*   `q` — quit
    

* * *

### Find a file / folder (from a starting directory)

*   `find <start_dir> -type f -name "filename.ext"` — find exact filename (case-sensitive)
    
*   `find <start_dir> -type f -iname "filename.ext"` — find exact filename (case-insensitive)
    
*   `find <start_dir> -type f -name "*.log"` — find files by pattern
    
*   `find <start_dir> -type d -name "foldername"` — find a folder/directory by name
    
**Examples**
*   `find /opt -type f -name "system.out.log"` — locate `system.out.log` under `/opt`
    
*   `find . -type f -iname "*error*.log"` — find any log with “error” in name
    

* * *

### Find text in files (must print file path)

**Recursive search (full path + line number + line content)**
*   `grep -Rni "search_text" <start_dir>` — recursive, shows full file path + line no
    
*   `grep -RniE "pattern1|pattern2" <start_dir>` — recursive regex (OR)
    
**Only show matching file paths (no line content)**
*   `grep -Rni -l "search_text" <start_dir>` — prints full paths of matching files only
    
*   `grep -Rni -l -E "exception|error|caused by|stacktrace" <start_dir>` — common error hunt, paths only
    
**Show `path:line` only (hide matched line content)**
*   `grep -Rni "search_text" <start_dir> | cut -d: -f1,2` — output `fullpath:line`
    
**Search only in certain files (find + grep)**
*   `find <start_dir> -type f -name "*.log" -exec grep -ni "Exception" {} \;` — full path + line no
    
*   `find <start_dir> -type f -name "*.log" -exec grep -ni -l -E "exception|caused by" {} \;` — only full paths
    

* * *

### Grep patterns for exceptions/errors (single file)

*   `grep -n "Exception" <file>` — search exceptions with line numbers
    
*   `grep -niE "exception|error|stacktrace|caused by" <file>` — common patterns (case-insensitive, regex)
    
*   `grep -n -C 3 "Exception" <file>` — match + 3 lines before/after (context)
    
*   `grep -n -A 20 "Exception" <file>` — match + 20 lines after (stacktrace)
    
*   `grep -niE "exception|error|caused by" <file> | tail -n 50` — last 50 matching hits
    
*   `tail -n 200 <file> | grep -i error` — search only recent 200 lines
    

* * *

### Quick counts & frequency (triage)

*   `wc -l <file>` — count lines in file
    
*   `grep -i "exception" <file> | sort | uniq -c | sort -nr | head` — top repeated matching lines
    

* * *

### Compare files (diff/cmp)

*   `diff <file1> <file2>` — show differences
    
*   `diff -u <file1> <file2>` — unified diff (best for wiki/tickets)
    
*   `diff -y <file1> <file2> | less` — side-by-side diff (paged)
    
*   `cmp -s <file1> <file2> && echo "SAME" || echo "DIFFERENT"` — quick identical check
    

* * *

### Create / copy / move / delete

*   `mkdir <dir>` — create directory
    
*   `touch <file>` — create empty file / update timestamp
    
*   `cp <src> <dst>` — copy file
    
*   `mv <old> <new>` — move/rename file
    
*   `rm <file>` — delete file (careful)
    
*   `rm -r <dir>` — delete folder recursively (very careful)
    

* * *

### Permissions (must-know)

*   `ls -l` — view permissions/owner/group
    
*   `chmod 640 <file>` — set perms to `rw-r-----`
    
*   `chmod +x <script.sh>` — make executable
    
*   `chown <user>:<group> <file>` — change owner/group (needs privilege)
    

* * *

### Edit file – Nano (simple)

*   `nano <file>` — open editor
    
*   `Ctrl+W` search | `Ctrl+K` cut line | `Ctrl+U` paste | `Ctrl+O` save | `Ctrl+X` exit
    

* * *

### Edit file – VI/VIM (minimal essentials)

*   `vi <file>` — open in vi
    
*   `i` insert mode | `Esc` normal mode
    
*   `:w` save | `:q` quit | `:wq` save+quit | `:q!` quit w/o saving
    
*   `gg` top | `G` bottom
    
*   `/text` search | `n` next | `N` previous
    
*   `dd` delete line | `yy` copy line | `p` paste below
    
*   `x` delete char | `u` undo
    
*   `:%s/old/new/g` — replace all occurrences in file
    

* * *

### Quick ops checks (support basics)

*   `ps -ef | grep java` — find java processes
    
*   `top` — live CPU/memory/process view (`q` quit)
    
*   `ss -lntp` — show listening TCP ports + owning process (Linux)
    
*   `df -h` — filesystem usage (human-readable)
    
*   `du -sh *` — size of each item in current folder
    
*   `free -h` — memory usage (Linux)
    
*   `uname -a` — kernel/system details
    

* * *

### Services & logs (systemd Linux)

*   `systemctl status <service>` — service status
    
*   `journalctl -u <service> --since "1 hour ago"` — logs for service since time window
    

* * *

### Remote basics

*   `ssh <user>@<host>` — connect to server
    
*   `scp <file> <user>@<host>:/path/` — copy file to server
