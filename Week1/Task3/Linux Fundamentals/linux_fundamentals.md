### 1. Navigating the File System

- **pwd**: To print the current working directory to the terminal.

  ```
  $ pwd
  /home/ubuntu
  ```

- **mkdir**: To create a new directory.

  ```
  mkdir oluwaseun
  ```
  
- **cd**: To change the current working directory in the terminal.

  ```
  cd oluwaseun
  ```

- **ls**: To list files and directories in the current directory.

  ```
  ls -l
  ```

  <img width="556" alt="Screenshot 2024-06-03 133702" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/05df75b9-7c53-46a6-ac6f-6d68a1b3c8f3">

- **rm**: To remove files or directories.

  ```
  rm -fr folder2
  rm file1
  rm -fr oluwaseun
  ```

  <img width="443" alt="Screenshot 2024-06-03 134852" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/1575842b-9515-45ff-b37f-7bdef8be7a69">

### 2. Managing Files and Directories

- **cp**: To copy files or directories from one location to another.

  ```
  cp file1 file2
  ```

- **mv**: To move files or directories from one location to another.

  ```
  mv file2 folder2
  ```

- **touch**: To create an empty file.

  ```
  touch file1
  ```

  <img width="556" alt="Screenshot 2024-06-03 133702" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/a71b8c85-9f82-41eb-a15e-73b2f5969728">

- **chmod**: To change the permissions of a file or directory.

  ```
  chmod 600 file1
  ```

- **chown**: To change the owner and group of a file or directory.

  ```
  sudo chown root:root folder2
  ```

  <img width="521" alt="Screenshot 2024-06-03 134028" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/0ba61b9c-8606-460e-9ae5-1301f065a4a6">

### 3. Use Text Editors like Vim to Manipulate Text Files

- **vi**: To launch the vi text editor.

  ```
  vi file1
  ```

### 4. Leverage Command-Line Tools for System Administration Tasks

- **ps**: To display information about currently running processes.

  ```
  ps -ef|more
  ```

  ![Screenshot 2024-06-03 134343](https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/ea696832-7bf8-49c4-bfbc-3a7e33dc0376)

- **top**: To display real-time information about system processes. [cpu, memory and swap utilization]

  ```
  top
  ```

  <img width="582" alt="Screenshot 2024-06-03 134203" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/956dba02-4640-4bcd-93bb-9533fb0a96e4">

- **df**: To display information about disk space usage and filesystem partitions.

  ```
  df -h
  ```

  ![Screenshot 2024-06-03 134147](https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/b00b92d8-fdaf-42ec-a3fe-0f51e6decb3b)

- **du**: To read file space usage.

  ```
  du -ms *
  ```

  ![Screenshot 2024-06-03 134259](https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/f0751a4f-ffa8-4dce-afc7-85eb186b3450)

- **lsof**: To list all open files and the processes that opened them.

  ```
  sudo lsof | more
  ```

  ![Screenshot 2024-06-03 134502](https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/b0e6cf14-4518-40ae-8941-19a41ccf2748)

- **netstat**: To display network connections on port 80 and to display active ports that are in a listening state.

  ```
  netstat -an| grep 80
  ```

  ```
  netstat -an | grep LISTEN
  ```

  <img width="680" alt="Screenshot 2024-06-03 134739" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/ecd39507-9080-4c26-8f83-8a3f5141fa43">

