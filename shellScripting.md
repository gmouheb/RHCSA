# Shell Scripting in Linux

## Introduction to Shell Scripting

Shell scripting is a powerful method to automate tasks in Linux. A shell script is a text file containing a series of commands that the shell can execute. The most common shell used for scripting is Bash (Bourne Again SHell), which is the default shell in most Linux distributions.

## Creating Your First Shell Script

1. Create a new file with a `.sh` extension:
   ```bash
   touch myscript.sh
   ```

2. Make the script executable:
   ```bash
   chmod +x myscript.sh
   ```

3. Add the shebang line at the beginning of the script to specify which shell to use:
   ```bash
   #!/bin/bash
   ```

4. Add commands to your script:
   ```bash
   #!/bin/bash
   echo "Hello, World!"
   ```

5. Run your script:
   ```bash
   ./myscript.sh
   ```

## Basic Shell Script Elements

### Variables

```bash
# Defining variables
name="John"
age=30

# Using variables
echo "My name is $name and I am $age years old."

# Command substitution
current_date=$(date)
echo "Current date and time: $current_date"

# Arithmetic operations
result=$((5 + 3))
echo "5 + 3 = $result"
```

### User Input

```bash
# Reading user input
echo "Enter your name:"
read username
echo "Hello, $username!"

# Reading with a prompt
read -p "Enter your age: " userage
echo "You are $userage years old."

# Reading password (hidden input)
read -sp "Enter your password: " userpassword
echo -e "\nPassword received."
```

### Conditional Statements

#### If-Else Statements

```bash
#!/bin/bash

age=25

if [ $age -lt 18 ]; then
    echo "You are a minor."
elif [ $age -ge 18 ] && [ $age -lt 65 ]; then
    echo "You are an adult."
else
    echo "You are a senior citizen."
fi
```

#### Case Statements

```bash
#!/bin/bash

read -p "Enter a fruit name: " fruit

case $fruit in
    "apple")
        echo "Apple is red."
        ;;
    "banana")
        echo "Banana is yellow."
        ;;
    "orange")
        echo "Orange is orange."
        ;;
    *)
        echo "Unknown fruit."
        ;;
esac
```

### Loops

#### For Loop

```bash
#!/bin/bash

# Iterate over a sequence
for i in {1..5}; do
    echo "Number: $i"
done

# Iterate over files
for file in *.txt; do
    echo "Found file: $file"
done

# C-style for loop
for ((i=0; i<5; i++)); do
    echo "Count: $i"
done
```

#### While Loop

```bash
#!/bin/bash

count=1

while [ $count -le 5 ]; do
    echo "Count: $count"
    ((count++))
done
```

#### Until Loop

```bash
#!/bin/bash

count=1

until [ $count -gt 5 ]; do
    echo "Count: $count"
    ((count++))
done
```

### Functions

```bash
#!/bin/bash

# Defining a function
greet() {
    echo "Hello, $1!"
}

# Function with return value
add() {
    local result=$(($1 + $2))
    return $result
}

# Calling functions
greet "World"

add 5 3
echo "5 + 3 = $?"  # $? contains the return value of the last command
```

## Working with Files and Directories

```bash
#!/bin/bash

# Check if file exists
if [ -f "/path/to/file" ]; then
    echo "File exists."
fi

# Check if directory exists
if [ -d "/path/to/directory" ]; then
    echo "Directory exists."
fi

# Create a directory if it doesn't exist
if [ ! -d "/path/to/directory" ]; then
    mkdir -p "/path/to/directory"
fi

# Read file line by line
while IFS= read -r line; do
    echo "Line: $line"
done < input.txt
```

## Command Line Arguments

```bash
#!/bin/bash

# $0 - Script name
# $1, $2, ... - Arguments
# $# - Number of arguments
# $@ - All arguments

echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "Number of arguments: $#"
echo "All arguments: $@"
```

## Error Handling

```bash
#!/bin/bash

# Exit on error
set -e

# Custom error handling
error_exit() {
    echo "Error: $1" >&2
    exit 1
}

# Using error handling
if [ ! -f "/path/to/file" ]; then
    error_exit "File not found!"
fi

# Capturing command output and status
if ! output=$(some_command 2>&1); then
    error_exit "Command failed: $output"
fi
```

## Advanced Shell Scripting Techniques

### Arrays

```bash
#!/bin/bash

# Declaring an array
fruits=("apple" "banana" "orange" "grape")

# Accessing array elements
echo "First fruit: ${fruits[0]}"
echo "All fruits: ${fruits[@]}"
echo "Number of fruits: ${#fruits[@]}"

# Iterating over an array
for fruit in "${fruits[@]}"; do
    echo "Fruit: $fruit"
done
```

### Regular Expressions

```bash
#!/bin/bash

# Using regex with if statement
if [[ "example@email.com" =~ [a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,} ]]; then
    echo "Valid email format."
fi

# Using regex with grep
if grep -q "^[0-9]{3}-[0-9]{3}-[0-9]{4}$" <<< "123-456-7890"; then
    echo "Valid phone number format."
fi
```

### Here Documents

```bash
#!/bin/bash

# Using here document to create a file
cat > config.txt << EOF
# Configuration file
name=John Doe
email=john@example.com
version=1.0
EOF

# Using here document with variables
name="John"
cat << EOF
Hello, $name!
Welcome to shell scripting.
EOF
```

## Best Practices for Shell Scripting

1. **Always use the shebang line**: `#!/bin/bash`
2. **Make scripts executable**: `chmod +x script.sh`
3. **Use meaningful variable names**
4. **Comment your code**
5. **Use functions for reusable code**
6. **Handle errors and edge cases**
7. **Use proper indentation for readability**
8. **Quote variables to prevent word splitting**: `echo "$variable"`
9. **Use exit codes to indicate success or failure**
10. **Test your scripts thoroughly**

## Debugging Shell Scripts

```bash
# Run script in debug mode
bash -x script.sh

# Enable debugging within the script
#!/bin/bash
set -x  # Enable debugging
# ... commands ...
set +x  # Disable debugging

# Verbose mode
set -v

# Exit on error
set -e
```

## Useful Shell Scripting Tools

- **shellcheck**: A tool that gives warnings and suggestions for shell scripts
- **bash -n script.sh**: Check script syntax without executing
- **trap**: Catch signals and execute code when they occur
- **getopts**: Parse command-line options
- **logger**: Send messages to the system log
- **cron**: Schedule scripts to run automatically

## Example: System Backup Script

```bash
#!/bin/bash

# Simple backup script

# Configuration
backup_dir="/backup"
source_dir="/home/user/documents"
date_format=$(date +%Y%m%d_%H%M%S)
backup_file="backup_${date_format}.tar.gz"

# Create backup directory if it doesn't exist
if [ ! -d "$backup_dir" ]; then
    mkdir -p "$backup_dir"
    if [ $? -ne 0 ]; then
        echo "Error: Failed to create backup directory."
        exit 1
    fi
fi

# Create backup
echo "Creating backup of $source_dir..."
tar -czf "${backup_dir}/${backup_file}" "$source_dir"

# Check if backup was successful
if [ $? -eq 0 ]; then
    echo "Backup created successfully: ${backup_dir}/${backup_file}"
    exit 0
else
    echo "Error: Backup failed."
    exit 1
fi
```