# Automation_Shell_Scripting

User Onboarding Automation with Shell Scripting This project automates the process of onboarding Linux users by reading their first names from a CSV file and creating new users on a server. Each new user is automatically added to the developers group, provided with a home directory, and given a preconfigured SSH setup with your public key.

## Project Instructions

The goal of this project is to:

1. Read a CSV file (names.csv) that contains first names (one per line).
2. Create new Linux user accounts for each name listed in the CSV file.
3. Add each new user to the developers group.
(Make sure to create this group beforehand using sudo groupadd developers.)
4. Verify if a user already exists before attempting to create them.
5. Ensure that each user gets a default home folder.
6. Set up SSH access for each new user by:
 - vCreating a .ssh directory (if it does not exist).
 - Creating an authorized_keys file.
 -  Appending your current user’s public key to the authorized_keys file.
7. Test a few accounts to confirm that you can connect using the provided private key.

Before deploying your script, update your own SSH keys in your home directory:

- Navigate to your .ssh folder.
- Create or update id_rsa.pub with your public key.
- Create or update id_rsa with your private key.

Finally, document your work by creating a Git repository named auxillary-projects, add your shell script, and record a demo showing the script in action. Then, submit your GitHub repository link along with a demo video link.

# SOLUTION

## Setting Up Your Project Environment

### Create a Project Folder

Once logged in to you instance, create a folder for your project:

`mkdir Shell && cd Shell`

This directory will hold all your project files (CSV, shell script, etc.) and keeps your work organized.


## Creating and Editing the CSV File

Inside the Shell directory, Create the CSV File `touch names.csv`.
Open the file `vim names.csv` and type in some random names e.g alice, bob, charlie and so on.

The CSV file holds the list of usernames that you will onboard. Each line should contain one username.

confirm that the users exist

`ls /home`

![names](./Images/Users.jpg)

Note: Linux typically expects usernames in lowercase.So, make sure you write the names in lower cases.

## Writing the Shell Script

- Create the Shell Script File 
  `touch onboarding_users.sh`
  `chmod +x onboarding_users.sh`
## Edit the Shell Script File
 Open the script file in your text editor and paste in the following script

```
#!/bin/bash

# Define CSV file and group name
CSV_FILE="names.csv"
GROUP_NAME="developers"

# Check if the group exists
if ! getent group $GROUP_NAME > /dev/null 2>&1; then
    echo "Group $GROUP_NAME does not exist. Please create it before running the script."
    exit 1
fi

# Read the CSV file line by line
while IFS=, read -r username
do
    # Skip empty lines
    if [ -z "$username" ]; then
        continue
    fi

    # Check if the user already exists
    if id "$username" &>/dev/null; then
        echo "User $username already exists. Skipping..."
    else
        echo "Creating user $username..."
        # Create the user with a home directory
        sudo adduser --quiet --gecos "" --disabled-password "$username"
        # Add the user to the 'developers' group
        sudo usermod -aG $GROUP_NAME "$username"

        # Create .ssh directory if it doesn't exist
        SSH_DIR="/home/$username/.ssh"
        if [ ! -d "$SSH_DIR" ]; then
            sudo mkdir "$SSH_DIR"
            sudo chown "$username:$username" "$SSH_DIR"
            sudo chmod 700 "$SSH_DIR"
        fi

        # Create the authorized_keys file and add the public key
        AUTH_KEYS="$SSH_DIR/authorized_keys"
        sudo cp /home/ubuntu/.ssh/id_rsa.pub "$AUTH_KEYS"
        sudo chown "$username:$username" "$AUTH_KEYS"
        sudo chmod 600 "$AUTH_KEYS"
        echo "User $username created and configured."
    fi
done < "$CSV_FILE"
```

The script starts by checking if the developers group exists.
It then reads each username from names.csv, skips blank lines, and checks if the user exists.
For new users, it creates the account, adds them to the developers group, sets up a home directory and .ssh folder, and copies the provided public key to the new user’s authorized_keys file.

## Creating the "developers" Group

Before running the script, you must ensure the group where new users will be added exists. Therefore, create the group

`sudo addgroup developers` 
![addgroup](./Images/addGroup.jpg)

Verify that the group you created exist

`getent group developers`

![getent](./Images/Groupcreate.jpg)

The script checks for the "developers" group. Creating it ensures that the script can add new users to this group without error.

## Preparing and Storing SSH Keys on the Server
The
 project instructions provide a public and private key that you must store on your server. These keys enable SSH login for the new users.

Navigate to the SSH Directory