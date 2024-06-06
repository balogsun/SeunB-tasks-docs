###Write and execute basic Bash scripts to automate repetitive tasks

### To create a basic Bash scripts that will backup a file named "seun.log" once the file is older than 5 minutes.

### mkdir oluwaseun

### Add any random contents to seun.log file with the vim command
vi seun.log

```
approach to project and planning

establish realistic timelines and alocate resources effectively

govt project may require adherence 

attention to detail and accuracy
no details are missed
how to know your methods are suceesfull

how do you priotize ur capital project 
within limited budget
example 
```
### move seun.log to oluwaseun directory and enter into the directory `oluwaseun`
mv seun.log oluwaseun
cd oluwaseun
<img width="413" alt="Screenshot 2024-06-05 194856" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/a4be4148-4f70-4bb0-b836-b0f9eb4de47e">


### Create the Bash scripts to automate repetitive tasks named `repeat.sh`

vi repeat.sh

```
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
<img width="456" alt="Screenshot 2024-06-05 195225" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/8a31aba2-6330-4f0a-ae8f-0ab7189bc380">

### Explaining how the variables in the script works:

dir="/home/ubuntu/oluwaseun": The directory where the file and backups are located.
file="$dir/seun.log": The full path to the log file.
backup_dir="$dir": The backup directory, which is the same as the directory containing the file.
Ensure File Exists:

if [ ! -f "$file" ]; then: Checks if the file exists. If not, it exits with a message.
Get Current Time and File Modification Time:

current_time=$(date +%s): Gets the current time in seconds since the epoch.
file_mod_time=$(stat -c %Y "$file"): Gets the last modification time of the file in seconds since the epoch.
Calculate Age of the File in Minutes:

age_in_minutes=$(( (current_time - file_mod_time) / 60 )): Calculates the age of the file in minutes.
Check if the File is Older than the Threshold:

if [ $age_in_minutes -gt $threshold ]; then: Checks if the file is older than the specified threshold.
Create a Backup:

timestamp=$(date +%Y%m%d%H%M%S): Generates a timestamp for the backup file name.
backup_file="$backup_dir/seun_$timestamp.log": Defines the backup file name.
cp "$file" "$backup_file": Copies the original file to the backup directory with the new name.
Output Messages:

Displays messages to indicate whether the file was backed up or if no backup was needed.

### Give `repeat.sh` script executable permission
chmod 755 repeat.sh

### Execute the bash script
./repeat.sh

### check that the backup has been effected, see that new log files were created.
ls -l
<img width="544" alt="Screenshot 2024-06-05 195429" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/c044dfd4-3128-4c8b-b136-29956b5c00be">

### After some minutes, more backups have now been taken in screenshot below
<img width="479" alt="Screenshot 2024-06-05 203142" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/8efc4879-03d5-46e2-ba1f-6da5a8b39638">


### Add cronjob entries to automate the bash script execution
crontab -e
crontab -l
*/10 * * * * /home/ubuntu/oluwaseun/repeat.sh
<img width="563" alt="Screenshot 2024-06-05 195906" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/224295d4-81e4-46cc-89a7-8cdeb26505fb">

2. **Utilizing Text Processing Utilities:**

Create a random log file named "seun.log", and run the following commands.

# Use grep to find lines containing a specific pattern e.g "suceesfull", not that the spelling of successfull here is inocrrect.
grep "suceesfull" $seun.log

result is displayes as expected in the screenshot sample below;

# Next, Use sed command for replace text, we will be replacing `suceesfull` to the correct word `succesfull`
sed -i 's/suceesfull/succesfull/g' $seun.log

<img width="521" alt="Screenshot 2024-06-05 201029" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/f4e795c9-18e9-47a8-9c64-b3a71414e344">


# Next, Using awk command, we will to print specific lines [line 3] in the file
awk 'NR == 3' seun.log
<img width="556" alt="Screenshot 2024-06-05 201541" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/33ea80ac-067d-46e9-831a-c58fb8fb7ba2">


3. **Automating System Administration for User Management:**

To create a bash script that will prompt you to enter any username you want, and then it will create that user using `adduser` and grant sudo access using `usermod` command.
I can add as many users as you need by running the script multiple times.
The script will display "Enter your choice: " on the terminal, and the user can type their choice (such as 1 or 2) followed by pressing Enter. 
The input provided by the user will be stored in the variable choice, which can then be used later in the script.

vi create_user.sh


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
give it executable permission:
chmod 755 create_user.sh

Execute the script
./create_user.sh
<img width="493" alt="image" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/bf8e903d-efd3-45e6-aadb-443802992ed7">

### Check characteristics of the newly create user:
sudo chage -l seun
cat /etc/passwd | grep seun
<img width="485" alt="Screenshot 2024-06-05 202905" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/1d20718a-83af-4e3a-a8ce-2e5842e1a0c8">

