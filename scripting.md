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

### Add cronjob entries to automate the bash script execution
crontab -e
crontab -l
*/10 * * * * /home/ubuntu/oluwaseun/repeat.sh

2. **Utilizing Text Processing Utilities:**

Create a random log file named "seun.log", and run the following commands.

# Use grep to find lines containing a specific pattern e.g "suceesfull", not that the spelling of successfull here is inocrrect.
grep "suceesfull" $seun.log

result is displayes as expected in the screenshot sample below;

# Next, Use sed command for replace text, we will be replacing `suceesfull` to the correct word `succesfull`
sed -i 's/suceesfull/succesfull/g' $seun.log

# Next, Using awk command, we will to print specific lines [line 3] in the file
awk 'NR == 3' seun.log


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

### Check characteristics of the newly create user:
sudo chage -l seun
cat /etc/passwd | grep seun
