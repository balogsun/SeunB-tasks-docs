# Wk1Tsk 4: Command-Line Scripting

## To Automating Repetitive Tasks with Bash Scripts, we try an example that will create a Basic Bash Script to Backup `seun.log` if it is Older than 5 Minutes

1. **Create the Directory and File:**

    ```bash
    mkdir oluwaseun
    ```

    Add random contents to `seun.log` file using `vim`:

    ```bash
    vi seun.log
    ```

    ```
    approach to project and planning

    establish realistic timelines and allocate resources effectively

    govt project may require adherence 

    attention to detail and accuracy
    no details are missed
    how to know your methods are successful

    how do you prioritize your capital project 
    within limited budget
    example 
    ```

2. **Move `seun.log` to `oluwaseun` Directory and change to the Directory:**

    ```bash
    mv seun.log oluwaseun
    cd oluwaseun
    ```

    ![Directory Move](https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/a4be4148-4f70-4bb0-b836-b0f9eb4de47e)

3. **Create the Bash Script `repeat.sh` to Automate Backups:**

    ```bash
    vi repeat.sh
    ```

    ```bash
    #!/bin/bash

    # File and backup directory location
    dir="/home/ubuntu/oluwaseun"
    file="$dir/seun.log"
    backup_dir="$dir"

    # Time threshold in minutes
    threshold=5

    # Check if the file exists
    if [ ! -f "$file" ]; then
        echo "File $file does not exist."
        exit 1
    fi

    # Get the current time and file modification time
    current_time=$(date +%s)
    file_mod_time=$(stat -c %Y "$file")

    # Calculate the age of the file in minutes
    age_in_minutes=$(( (current_time - file_mod_time) / 60 ))

    # Check if the file is older than the threshold
    if [ $age_in_minutes -gt $threshold ]; then
        echo "File $file is older than $threshold minutes. Creating a backup."
        # Create a backup file with the current timestamp
        timestamp=$(date +%Y%m%d%H%M%S)
        backup_file="$backup_dir/seun_$timestamp.log"
        cp "$file" "$backup_file"
        echo "Backup created: $backup_file"
    else
        echo "File $file is not older than $threshold minutes. No backup needed."
    fi
    ```

    ![Repeat Script](https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/8a31aba2-6330-4f0a-ae8f-0ab7189bc380)

### Explanation of Script Variables

- `dir="/home/ubuntu/oluwaseun"`: The directory where the file and backups are located.
- `file="$dir/seun.log"`: The full path to the log file.
- `backup_dir="$dir"`: The backup directory, which is the same as the directory containing the file.

### Ensure File Exists

- `if [ ! -f "$file" ]; then`: Checks if the file exists. If not, it exits with a message.

### Get Current Time and File Modification Time

- `current_time=$(date +%s)`: Gets the current time in seconds since the epoch.
- `file_mod_time=$(stat -c %Y "$file")`: Gets the last modification time of the file in seconds since the epoch.

### Calculate Age of the File in Minutes

- `age_in_minutes=$(( (current_time - file_mod_time) / 60 ))`: Calculates the age of the file in minutes.

### Check if the File is Older than the Threshold

- `if [ $age_in_minutes -gt $threshold ]; then`: Checks if the file is older than the specified threshold.

### Create a Backup

- `timestamp=$(date +%Y%m%d%H%M%S)`: Generates a timestamp for the backup file name.
- `backup_file="$backup_dir/seun_$timestamp.log"`: Defines the backup file name.
- `cp "$file" "$backup_file"`: Copies the original file to the backup directory with the new name.

### Output Messages

Displays messages to indicate whether the file was backed up or if no backup was needed.

4. **Give `repeat.sh` Script Executable Permission:**

    ```bash
    chmod 755 repeat.sh
    ```

5. **Execute the Bash Script:**

    ```bash
    ./repeat.sh
    ```

6. **Check that the Backup has been Effected:**

    ```bash
    ls -l
    ```

    ![Backup Verification](https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/c044dfd4-3128-4c8b-b136-29956b5c00be)

7. **After Some Minutes, More Backups are Created:**

    ![Multiple Backups](https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/8efc4879-03d5-46e2-ba1f-6da5a8b39638)

### Add Cronjob Entries to Automate the Bash Script Execution

1. **Edit the Cronjob:**

    ```bash
    crontab -e
    ```

2. **List Cronjob Entries:**

    ```bash
    crontab -l
    ```

3. **Cronjob Entry to Run Every 10 Minutes:**

    ```plaintext
    */10 * * * * /home/ubuntu/oluwaseun/repeat.sh
    ```

    ![Cronjob Entry](https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/224295d4-81e4-46cc-89a7-8cdeb26505fb)

## Utilizing Text Processing Utilities

Create a random log file named `seun.log` and run the following commands.

1. **Use `grep` to Find Lines Containing a Specific Pattern (e.g., "suceesfull"):**

    ```bash
    grep "suceesfull" seun.log
    ```

    Result is displayed as expected:

2. **Use `sed` Command to Replace Text:**

    Replace `suceesfull` with the correct word `successful`:

    ```bash
    sed -i 's/suceesfull/successful/g' seun.log
    ```

    ![sed Command](https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/f4e795c9-18e9-47a8-9c64-b3a71414e344)

3. **Use `awk` Command to Print Specific Lines (Line 3) in the File:**

    ```bash
    awk 'NR == 3' seun.log
    ```

    ![awk Command](https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/33ea80ac-067d-46e9-831a-c58fb8fb7ba2)

## Automating System Administration for User Management

Create a bash script that prompts you to enter any username you want and then creates that user using `adduser` and grants sudo access using the `usermod` command.

1. **Create the Script `create_user.sh`:**

    ```bash
    vi create_user.sh
    ```

    ```bash
    #!/bin/bash

    # Add a new user and grant sudo access
    add_user() {
        read -p "Enter username: " username
        sudo adduser $username
        sudo usermod -aG sudo $username  # Grant sudo access to the new user
    }

    # Main menu
    while true; do
        echo "1. Add User"
        echo "2. Exit"
        read -p "Enter your choice: " choice

        case $choice in
            1) add_user ;;
            2) break ;;
            *) echo "Invalid option";;
        esac
    done
    ```

2. **Give the Script Executable Permission:**

    ```bash
    chmod 755 create_user.sh
    ```

3. **Execute the Script:**

    ```bash
    ./create_user.sh
    ```

    ![Create User Script](https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/bf8e903d-efd3-45e6-aadb-443802992ed7)

4. **Check Characteristics of the Newly Created User:**

    ```bash
    sudo chage -l seun
    cat /etc/passwd | grep seun
    ```

    ![User Characteristics](https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/1d20718a-83af-4e3a-a8ce-2e5842e1a0c8)
