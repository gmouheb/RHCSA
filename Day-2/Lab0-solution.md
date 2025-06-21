# Lab 0: User and Group Management - Solutions

This document provides solutions for the requirements in Lab 0.

## Solutions

### Requirement 1: Create Groups
Create the sales and account groups:

```bash
sudo groupadd sales
sudo groupadd account
```

### Requirement 2: Create Users with Private Primary Groups
Create users with their own private primary groups:

```bash
# Create users with private primary groups
sudo useradd -m -g joana -G sales joana
sudo useradd -m -g john -G sales john
sudo useradd -m -g laura -G account laura
sudo useradd -m -g beatrix -G account beatrix

# Set passwords for each user 
sudo passwd joana
sudo passwd john
sudo passwd laura
sudo passwd beatrix
```

### Requirement 3: Add Users to Groups
This was already accomplished in the previous step by using the `-G` option during user creation.

### Requirement 4: Set Password Expiration Policy
There are two ways to set the password expiration policy:

#### Option 1: Set for Individual Users
```bash
# Set password expiration policy for individual users
sudo chage -M 90 joana
sudo chage -M 90 john
sudo chage -M 90 laura
sudo chage -M 90 beatrix
```

#### Option 2: Set System-Wide Policy
```bash
# Edit the login definitions file
sudo nano /etc/login.defs

# Set the following values
PASS_MAX_DAYS   90
PASS_MIN_DAYS   0
PASS_WARN_AGE   7
```

Note: The system-wide setting will only apply to new users created after the change. For existing users, you'll need to use the `chage` command.

## Verification
To verify that the users have been created with the correct group memberships:

```bash
id joana
id john
id laura
id beatrix
```