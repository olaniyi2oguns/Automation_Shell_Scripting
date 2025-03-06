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

- The script starts by checking if the developers group exists.
- It then reads each username from names.csv, skips blank lines, and checks if the user exists.
- For new users, it creates the account, adds them to the developers group, sets up a home directory and .ssh folder, and copies the provided public key to the new user’s authorized_keys file.

## Creating the "developers" Group

Before running the script, you must ensure the group where new users will be added exists. Therefore, create the group

`sudo addgroup developers` 
![addgroup](./Images/addGroup.jpg)

Verify that the group you created exist

`getent group developers`

![getent](./Images/Groupcreate.jpg)

The script checks for the "developers" group. Creating it ensures that the script can add new users to this group without error.

## Preparing and Storing SSH Keys on the Server

The project instructions provide a public and private key that you must store on your server. These keys enable SSH login for the new users.

Store the Provided Public Key
Create the public key file:

`touch id_rsa.pub`

Open the fie `vim id_rsa.pub` and paste the key below into it

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCzKZyicHxIkklSrNlxsJyyTrcIdBIt84Z0cQb3R4k0jH53kxkaT5hP8tfWTe62LXi7vV86fY+SX7TBNM76XGCbw/6vrMGegm6J1x2i1AiLNwq5nqTjOGn0AIwku4IlCCLAB7tdfRyVuCarmBlwny3lzRyybIUAWXR/D6vpN09MsDILbKdhay+Q/p9OUBMSLPqXdY/QIh/Oe3rVv1lwY3AohNfq7V3tO88zKswfA5iiexNiSYX1myT0OrX8cBE771j9quoNZhQgaLI1mIMtAvnHQChrn9k2nUaO/BMBCQGol5XzGv1ado7hgoVPoluIUD+FGNo/pH4zcmDLICH6drXY/C9MESnkMUPLFxBXKO/OitApY71vRao9nAhAwpVMsy6FqiOb5uawhvhoHYIHTV/f4EtagVagRMP2PxYMYR6jykIV4MPJTkCm+lGhTyMlRu+qRQjdLn8AAtHf4aEV8dIkoGh088DI7eA/4o0wz4OV4upH5ewSFS+5IHmRECEW5Nc=
```

To  create the private key, you will need to navigate to the location/folder where you like to have your private key file to be stored, and create the file in there.

`touch id_rsa` 

Open the file `vim id_rsa` and paste the key below in there:

```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEAsymconB8SJJJUqzZcbCcsk63CHQSLfOGdHEG90eJNIx+d5MZGk+Y
T/LX1k3uti14u71fOn2Pkl+0wTTO+lxgm8P+r6zBnoJuidcdotQIizcKuZ6k4zhp9ACMJL
uCJQgiwAe7XX0clbgmq5gZcJ8t5c0csmyFAFl0fw+r6TdPTLAyC2ynYWsvkP6fTlATEiz6
l3WP0CIfznt61b9ZcGNwKITX6u1d7TvPMyrMHwOYonsTYkmF9Zsk9Dq1/HARO+9Y/arqDW
YUIGiyNZiDLQL5x0Aoa5/ZNp1GjvwTAQkBqJeV8xr9WnaO4YKFT6JbiFA/hRjaP6R+M3Jg
yyAh+na12PwvTBEp5DFDyxcQVyjvzorQKWO9b0WqPZwIQMKVTLMuhaojm+bmsIb4aB2CB0
1f3+BLWoFWoETD9j8WDGEeo8pCFeDDyU5ApvpRoU8jJUbvqkUI3S5/AALR3+GhFfHSJKBo
dPPAyO3gP+KNMM+DleLqR+XsEhUvuSB5kRAhFuTXAAAFgIuJ0uiLidLoAAAAB3NzaC1yc2
EAAAGBALMpnKJwfEiSSVKs2XGwnLJOtwh0Ei3zhnRxBvdHiTSMfneTGRpPmE/y19ZN7rYt
eLu9Xzp9j5JftME0zvpcYJvD/q+swZ6CbonXHaLUCIs3CrmepOM4afQAjCS7giUIIsAHu1
19HJW4JquYGXCfLeXNHLJshQBZdH8Pq+k3T0ywMgtsp2FrL5D+n05QExIs+pd1j9AiH857
etW/WXBjcCiE1+rtXe07zzMqzB8DmKJ7E2JJhfWbJPQ6tfxwETvvWP2q6g1mFCBosjWYgy
0C+cdAKGuf2TadRo78EwEJAaiXlfMa/Vp2juGChU+iW4hQP4UY2j+kfjNyYMsgIfp2tdj8
L0wRKeQxQ8sXEFco786K0CljvW9Fqj2cCEDClUyzLoWqI5vm5rCG+GgdggdNX9/gS1qBVq
BEw/Y/FgxhHqPKQhXgw8lOQKb6UaFPIyVG76pFCN0ufwAC0d/hoRXx0iSgaHTzwMjt4D/i
jTDPg5Xi6kfl7BIVL7kgeZEQIRbk1wAAAAMBAAEAAAGAPf8KOpOeDibAxKEXZWXt8y2V3J
D9sXTxc92gwXS5n7t2D76REy+zzwaDdZ7mGZhGjQCMsVq9kbMYgzrY3H2W2I/L09J99XHA
+mW71Zp1kmbriSvCdvYQg+SkmhlggZv9GmISjdk7SPu+Nead9wC+CyUc5wjyRRqvW0B7Bm
qjQDBAQP/KM8W5Yf0Z9ylyT/nMhRijOSx1wSeta8WZF3DxYLQHWz3kILFvk48dryW5bZAV
Nw+mEUUsVm7yhnXpIMpDdl7wlDlqAWcuEQKJ7WJ7swuZM/FTQW4rFMmpDO8Q8PgijqOFDQ
jl8XfCPCkOhI9JOFTbmImTxfbRZ/NYYF09cFcqhKyvEi/Egx2oUZq4M81EGpP+EZnWgZtG
/PHqrSqIW166fixe/47eGCSt+AlyeR8SZCA1jjMRf7WB1RjANUHgC59tNTMQiFg+T5c2Yj
ORmPT0PpzEtQ+WMSMI5hGoklmqXuS5iiyJx7HyLOnK7wNloj7oqboz91wcCYnYWCORAAAA
wQDUbuGf0dAtJ4Qr2vdHiIi4dHAlMQMMsw/12CmpuSoqeEIWHVpAEBpqzx67qDZ+AMpBDV
B U9KbXe7IIzzfwUvxl1WCycg/pJM0OMjyigvz4XziuSVmSuy10HNvECvpxI3Qx8iF/HgAP
eyYe369FHEBsNZ5M5KhZ4oHI/XgZB5OGOaxErJd3wXhGASHnsWcmIswIjat7hH9WlAeWAk
/aeMz92iSDnYBOr+gAycsBm/skEDrN7dD45ilSvLZ6DQ2hbKAAAADBAOhLy9Tiki1IM2Gg
ma8KkUiLrqqx8IexPd580n7KsL32U2iu6Y88+skC8pkZQmIVG2UQhjiVLpNBgrzKKDJciK
/lyen21npQjuYaJPUgVUG0sjMtTpgGwbN/IVyHO28KSOogB6MclRBW7Z2SJggSAJaQmO9g
u7kieXbtf+5A7gUSb7icD629OiYCEJMTKTpVS/Pk7NDx/ZXQVzGrkJMKdPFU8nDoOjFLSP
jdbbddYe6zuB/HwabV3Lpaxl38tNG78wAAAMEAxXHS2IRABAvX7+OmZO2JU7+9Gxh/gudJ
eXf76c10kKvUztoe8Mskw79yVq6LtYd0JGOVx0oNgMeZJHmwUc2qVPKaFGEhSG6MuFn3J2
O5+Kt+KfU5M9uAN7tob3+yG18ZJt9FY+5FTK1TV5LmF5OTGBN9XyehT2Miqa8sSu80rwpN
nhe+U/XswAp9KEVYkSIjFeoy/amsOP+qvRke1dKWBsU12IbhnMgjDHVggkYV52l7d9S2bx
kmaSGj362OnCCNAAAACWRhcmVARGFyZQE=
-----END OPENSSH PRIVATE KEY-----
```
![key](./Images/{1EB8F68D-0DCE-4F8D-89A1-77125D85A312}.jpg)

NOTE: vsc or window terminal may not allow `touch` command. So, I suggest you use git bash editor.

## Set the Correct Permissions

`chmod 600 id_rsa`
`chmod 644 id_rsa.pub`

The private key must be secure (only readable by you), while the public key is readable. This setup allows the script to copy the public key for each new user.

## Testing the Script and User Onboarding
Run the Script
In the ~/Shell directory, execute:

`./onboarding_users.sh`

![confirm](./Images/confirmconfirm.jpg)

The script processes each username in the CSV file and creates/configures the user accordingly.

## Test SSH Connection for a New User

Choose one username from names.csv (for example, alice) and, in a new terminal window, test logging in:

`ssh -i id_rsa alice@<EC2_PUBLIC_IP_ADDRESS>`

Replace <EC2_PUBLIC_IP_ADDRESS> with your instance’s public IP

This tests if the new user can log in using the SSH key copied during the onboarding process.

![Final](./Images/Final.jpg)

# Congratulations
This comprehensive guide covers everything from launching your EC2 instance to submitting your work for review. Each step includes commands and explanations to ensure even a newbie can follow along without missing any details. Happy scripting!

