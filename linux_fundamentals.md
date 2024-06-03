### 1. Navigating the File System:
- **pwd**: To print the current working directory to the terminal.
  ```
  $ pwd
  /home/ubuntu
  ```

- **mkdir**: To create a new directory.
  ```
  $ mkdir oluwaseun
  ```
  
- **cd**: To change the current working directory in the terminal.
  ```
  $ cd oluwaseun
  ```
- **ls**: To list files and directories in the current directory.
  ```
  $ ls -l
  ```

- **rm**: To remove files or directories.
  ```
  $ rm -fr folder2
  $ rm file1
  $ rm -fr oluwaseun
  ```

### 2. Managing Files and Directories:
- **cp**: To copy files or directories from one location to another.
  ```
  $ cp file1 file2
  ```
- **mv**: To move files or directories from one location to another.
  ```
  $ mv file1 folder2
  ```
- **touch**: To create an empty file.
  ```
  $ touch file1
  ```
- **chmod**: To change the permissions of a file or directory.
  ```
  $ chmod 600 file1
  ```
- **chown**: To change the owner and group of a file or directory.
  ```
  $ sudo chown root:root folder2
  ```

### 3. Use Text Editors like Vim to Manipulate Text Files:
- **vi**: To launch the vi text editor.
  ```
  $ vi file1
  ```

### 4. Leverage Command-Line Tools for System Administration Tasks:
- **ps**: To display information about currently running processes.
  ```
  $ ps -ef|more
  ```
- **top**: To display real-time information about system processes.
  ```
  $ top
  ```
- **df**: To display information about disk space usage.
  ```
  $ df -h
  ```
- **du**: To estimate file space usage.
  ```
  $ du -ms *
  ```
- **lsof**: To list all open files and the processes that opened them.
  ```
  $ sudo lsof | more
  ```
- **netstat**: To display network connections.
  ```
  $ netstat -an| grep 80
  ```
  ```
  $ netstat -neopa | more
  ```
  ```
  $ netstat -an | grep LISTEN
  ```

