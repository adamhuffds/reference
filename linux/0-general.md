# General Linux Command Cheat Sheet

Reference: https://www.gnu.org/software/coreutils/manual/coreutils.html

## Package Updates (apt)

```bash
sudo apt update            # refresh package lists (doesn't install anything)
sudo apt upgrade           # upgrade installed packages to latest versions
sudo apt full-upgrade      # upgrade + resolve/remove packages if needed for the upgrade
sudo apt autoremove        # remove packages no longer required by anything installed
```
Docs: https://manpages.ubuntu.com/manpages/noble/man8/apt.8.html

## General Navigation & File Operations

```bash
pwd                                  # print working directory
ls [options] [directory]             # list directory contents
ls -l                                # long format (permissions, size, date)
ls -a                                # show hidden files (dotfiles)
cd [directory]                       # change directory
cd ~  or  cd                         # go to home directory
cd ..                                # go up one directory level

mkdir [directory_name]               # create a new directory
rmdir [directory_name]               # remove an empty directory

touch [file_name]                    # create an empty file, or update its timestamp if it exists

cp [source] [destination]            # copy a file
cp -r [source_dir] [destination_dir] # copy a directory recursively

mv [source] [destination]            # move or rename a file/directory

rm [file_name]                       # remove a file
rm -r [directory_name]               # remove a directory and its contents recursively (use with caution)
rm -rf [directory_name]              # force remove, no confirmation prompts (use with extreme caution)
```

## Handling Duplicates & Merging

Tools: [fdupes](https://github.com/adrianlopezroche/fdupes), jdupes (a faster fork of fdupes)

```bash
# Find duplicate files between two different folders
fdupes -r folderA folderB          # identify duplicates
fdupes -rdN folderA folderB        # delete duplicates, keep first copy, prompt for confirmation

# Find duplicate files within a single folder
fdupes -r folder                   # identify duplicates
fdupes -rd folder                  # identify + prompt to delete duplicates
fdupes -rdN folder                 # delete duplicates automatically, keep first copy
```
- `-r` = recursive (search subdirectories)
- `-d` = prompt for deletion of duplicates found
- `-N` = when combined with `-d`, automatically keep the first file in each set without prompting per-file

## Processes

```bash
ps aux | grep process_name           # list processes matching a name
ps aux | grep [p]rocess_name         # bracket trick to exclude the grep command itself from results
pgrep process_name                   # prints matching PID(s) if running, nothing if not

pgrep process_name >/dev/null && echo "Running" || echo "Not running"
echo $?                               # exit code of last command: 0 = success/found, 1 = not found

# PID = Process ID, returned by pgrep (and other tools), used to inspect/control a process
ps -p PID                             # check details of a specific process
kill PID                              # send termination signal to a process
```

## File Content Display & Editing

```bash
cat [file_name]        # display full contents of a file
less [file_name]       # view file content page by page (scrollable)
head [file_name]       # display the beginning of a file
tail [file_name]       # display the end of a file
nano [file_name]       # open file in the nano text editor
vim [file_name]        # open file in the vim text editor
```

## System Information & Process Management

```bash
whoami          # display current username
hostname        # display system hostname
ps              # list running processes
top             # real-time view of system processes and resource usage
kill [PID]      # terminate a process by its Process ID
```

## Using sudo

```bash
sudo apt update                    # run a command as root (apt = package management tool)
sudo -u postgres psql my_database  # run a command as a specific user (here, the postgres system user)
```
Docs: https://manpages.ubuntu.com/manpages/noble/man8/sudo.8.html

## Permissions & Ownership

```bash
chmod [permissions] [file_name]    # change file permissions
chmod u+x script.sh                 # give the file's owner (user) execute permission
chown [user][:group] [file_name]    # change file owner and/or group
```
Docs: https://manpages.ubuntu.com/manpages/noble/man1/chmod.1.html

### Input/Output Redirection & Pipes

```bash
command > file        # redirect output to a file (overwrites existing content)
command >> file        # redirect output to a file (appends to existing content)
command < file        # use a file as input for a command
command1 | command2   # pipe command1's output as input to command2
```

## Search & Filtering

```bash
grep [pattern] [file_name]       # search for a pattern in a file
grep -i "pattern" file.txt       # case-insensitive search
find [path] [expression]          # search for files/directories matching criteria
```
Docs: https://manpages.ubuntu.com/manpages/noble/man1/grep.1.html | https://manpages.ubuntu.com/manpages/noble/man1/find.1.html