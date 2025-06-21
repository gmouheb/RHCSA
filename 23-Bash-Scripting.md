# Bash Scripting in Linux

This document covers Bash scripting concepts and techniques for Linux system administration.

## Introduction to Bash Scripting

Bash (Bourne Again SHell) is the default shell in most Linux distributions. Bash scripts allow you to:

- Automate repetitive tasks
- Create custom commands
- Manage system configurations
- Process text and files
- Schedule and run system maintenance

## Creating Your First Bash Script

### Basic Script Structure

```bash
#!/bin/bash
# This is a comment
echo "Hello, World!"
```

### Making Scripts Executable

```bash
# Create a script
vi myscript.sh

# Make it executable
chmod +x myscript.sh

# Run the script
./myscript.sh
```

## Bash Script Components

### Shebang Line

The first line of a script specifies the interpreter:

```bash
#!/bin/bash
```

Alternative shebangs:
```bash
#!/usr/bin/env bash  # More portable across systems
#!/bin/sh            # POSIX-compliant shell
```

### Variables

```bash
# Assigning variables
name="John"
age=30
current_date=$(date +%Y-%m-%d)

# Using variables
echo "Hello, $name!"
echo "You are $age years old."
echo "Today is $current_date"

# Command substitution
user_count=$(who | wc -l)
echo "Number of logged-in users: $user_count"

# Arithmetic operations
result=$((10 + 5))
echo "10 + 5 = $result"
```

### Special Variables

```bash
$0    # Script name
$1    # First argument
$2    # Second argument
$@    # All arguments as separate words
$*    # All arguments as a single word
$#    # Number of arguments
$$    # Process ID of the script
$?    # Exit status of the last command
```

### User Input

```bash
# Read user input
echo "Enter your name:"
read name

# Read with prompt
read -p "Enter your age: " age

# Read password (hidden input)
read -sp "Enter password: " password
echo # New line after password input

# Read with timeout
read -t 5 -p "You have 5 seconds to answer: " answer
```

## Control Structures

### Conditional Statements

#### If-Else Statements

```bash
# Basic if statement
if [ "$age" -ge 18 ]; then
    echo "You are an adult."
fi

# If-else statement
if [ "$age" -ge 18 ]; then
    echo "You are an adult."
else
    echo "You are a minor."
fi

# If-elif-else statement
if [ "$age" -lt 13 ]; then
    echo "You are a child."
elif [ "$age" -lt 18 ]; then
    echo "You are a teenager."
else
    echo "You are an adult."
fi
```

#### Case Statements

```bash
echo "Enter a fruit name:"
read fruit

case "$fruit" in
    "apple")
        echo "Red fruit"
        ;;
    "banana")
        echo "Yellow fruit"
        ;;
    "orange"|"tangerine")
        echo "Orange fruit"
        ;;
    *)
        echo "Unknown fruit"
        ;;
esac
```

### Loops

#### For Loops

```bash
# Loop through a list
for name in John Mary Steve; do
    echo "Hello, $name!"
done

# Loop through command output
for file in $(ls *.txt); do
    echo "Processing $file"
done

# C-style for loop
for ((i=1; i<=5; i++)); do
    echo "Count: $i"
done
```

#### While Loops

```bash
# Basic while loop
count=1
while [ $count -le 5 ]; do
    echo "Count: $count"
    count=$((count + 1))
done

# Read file line by line
while read line; do
    echo "Line: $line"
done < input.txt
```

#### Until Loops

```bash
# Until loop (runs until condition becomes true)
count=1
until [ $count -gt 5 ]; do
    echo "Count: $count"
    count=$((count + 1))
done
```

### Loop Control

```bash
# Break statement
for i in {1..10}; do
    if [ $i -eq 5 ]; then
        break
    fi
    echo $i
done

# Continue statement
for i in {1..10}; do
    if [ $i -eq 5 ]; then
        continue
    fi
    echo $i
done
```

## Functions

### Defining and Calling Functions

```bash
# Define a function
hello() {
    echo "Hello, World!"
}

# Call the function
hello

# Function with parameters
greet() {
    echo "Hello, $1! You are $2 years old."
}

# Call with arguments
greet "John" 30
```

### Return Values

```bash
# Return a status code
is_even() {
    if [ $(($1 % 2)) -eq 0 ]; then
        return 0  # Success (true)
    else
        return 1  # Failure (false)
    fi
}

# Use the return value
if is_even 4; then
    echo "4 is even"
else
    echo "4 is odd"
fi

# Return a value via echo
get_sum() {
    echo $(($1 + $2))
}

# Capture the output
sum=$(get_sum 5 3)
echo "Sum: $sum"
```

### Local Variables

```bash
# Function with local variables
calculate() {
    local x=10
    local y=5
    echo "Sum: $((x + y))"
}

calculate
# x and y are not accessible here
```

## File Operations

### Reading and Writing Files

```bash
# Write to a file
echo "Hello, World!" > output.txt
echo "Appended line" >> output.txt

# Read from a file
cat input.txt

# Read file line by line
while read line; do
    echo "Line: $line"
done < input.txt
```

### File Tests

```bash
file="example.txt"

# Check if file exists
if [ -e "$file" ]; then
    echo "File exists"
fi

# Check if it's a regular file
if [ -f "$file" ]; then
    echo "Regular file"
fi

# Check if it's a directory
if [ -d "$file" ]; then
    echo "Directory"
fi

# Check if file is readable
if [ -r "$file" ]; then
    echo "File is readable"
fi

# Check if file is writable
if [ -w "$file" ]; then
    echo "File is writable"
fi

# Check if file is executable
if [ -x "$file" ]; then
    echo "File is executable"
fi

# Check if file is not empty
if [ -s "$file" ]; then
    echo "File is not empty"
fi
```

## String Operations

### String Comparison

```bash
str1="hello"
str2="world"

# Equality
if [ "$str1" = "$str2" ]; then
    echo "Strings are equal"
fi

# Inequality
if [ "$str1" != "$str2" ]; then
    echo "Strings are not equal"
fi

# String is empty
if [ -z "$str1" ]; then
    echo "String is empty"
fi

# String is not empty
if [ -n "$str1" ]; then
    echo "String is not empty"
fi
```

### String Manipulation

```bash
string="Hello, World!"

# String length
echo "Length: ${#string}"

# Substring
echo "Substring: ${string:7:5}"  # Starts at position 7, length 5

# Replace
echo "Replace: ${string/World/Linux}"

# Replace all occurrences
echo "Replace all: ${string//l/L}"

# Remove prefix
echo "Remove prefix: ${string#Hello, }"

# Remove suffix
echo "Remove suffix: ${string%World!}"

# Convert to uppercase
echo "Uppercase: ${string^^}"

# Convert to lowercase
echo "Lowercase: ${string,,}"
```

## Arrays

### Array Operations

```bash
# Declare an array
fruits=("apple" "banana" "orange")

# Access elements
echo "First fruit: ${fruits[0]}"
echo "Second fruit: ${fruits[1]}"

# All elements
echo "All fruits: ${fruits[@]}"

# Array length
echo "Number of fruits: ${#fruits[@]}"

# Add element
fruits+=("grape")

# Iterate through array
for fruit in "${fruits[@]}"; do
    echo "Fruit: $fruit"
done

# Associative arrays (dictionaries)
declare -A user
user[name]="John"
user[age]=30
user[city]="New York"

echo "Name: ${user[name]}"
echo "Age: ${user[age]}"

# Iterate through keys
for key in "${!user[@]}"; do
    echo "$key: ${user[$key]}"
done
```

## Error Handling

### Exit Status

```bash
# Check exit status
ls /nonexistent
if [ $? -ne 0 ]; then
    echo "Command failed"
fi

# Set exit status
exit 0  # Success
exit 1  # Failure
```

### Error Redirection

```bash
# Redirect stderr to stdout
command 2>&1

# Redirect stderr to a file
command 2> error.log

# Redirect both stdout and stderr to a file
command &> output.log

# Discard output
command > /dev/null 2>&1
```

### Error Trapping

```bash
# Set up error trap
trap 'echo "Error occurred at line $LINENO"; exit 1' ERR

# Clean up on exit
trap 'echo "Cleaning up..."; rm -f temp_file.txt' EXIT
```

## Command Line Arguments

### Parsing Arguments

```bash
# Basic argument handling
echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "All arguments: $@"
echo "Number of arguments: $#"

# Shift arguments
shift
echo "After shift, first argument: $1"
```

### Getopts for Option Parsing

```bash
# Parse options with getopts
while getopts "a:b:c" opt; do
    case $opt in
        a)
            echo "Option -a with value $OPTARG"
            ;;
        b)
            echo "Option -b with value $OPTARG"
            ;;
        c)
            echo "Option -c"
            ;;
        \?)
            echo "Invalid option: -$OPTARG"
            exit 1
            ;;
    esac
done

# Process remaining arguments
shift $((OPTIND - 1))
echo "Remaining arguments: $@"
```

## Debugging Bash Scripts

### Debugging Techniques

```bash
# Enable debug mode
set -x  # Print commands and arguments as they are executed
set -e  # Exit immediately if a command exits with non-zero status
set -u  # Treat unset variables as an error
set -o pipefail  # Return value of a pipeline is the value of the last command to exit with non-zero status

# Disable debug mode
set +x

# Run script in debug mode
bash -x script.sh
```

## Best Practices

1. **Always use shebang**: Start scripts with `#!/bin/bash`
2. **Use meaningful variable names**: `user_count` instead of `uc`
3. **Quote variables**: Use `"$variable"` to prevent word splitting and globbing
4. **Use functions** for reusable code blocks
5. **Comment your code** to explain complex logic
6. **Check for required tools** at the beginning of scripts
7. **Validate input** before processing
8. **Handle errors** and provide meaningful error messages
9. **Use exit codes** appropriately
10. **Follow a consistent style** for readability

## Example Scripts

### System Information Script

```bash
#!/bin/bash
# Script to display system information

echo "System Information"
echo "================="
echo "Hostname: $(hostname)"
echo "Kernel: $(uname -r)"
echo "Uptime: $(uptime -p)"
echo "CPU: $(grep 'model name' /proc/cpuinfo | head -1 | cut -d ':' -f 2 | sed 's/^[ \t]*//')"
echo "Memory: $(free -h | grep Mem | awk '{print $2}')"
echo "Disk Usage:"
df -h | grep '^/dev'
```

### Backup Script

```bash
#!/bin/bash
# Simple backup script

# Configuration
backup_dir="/backup"
source_dir="/home/user/documents"
date_format=$(date +%Y%m%d)
backup_file="backup_${date_format}.tar.gz"

# Check if backup directory exists
if [ ! -d "$backup_dir" ]; then
    mkdir -p "$backup_dir"
    if [ $? -ne 0 ]; then
        echo "Error: Failed to create backup directory"
        exit 1
    fi
fi

# Create backup
echo "Creating backup of $source_dir..."
tar -czf "${backup_dir}/${backup_file}" "$source_dir"

# Check if backup was successful
if [ $? -eq 0 ]; then
    echo "Backup created successfully: ${backup_dir}/${backup_file}"
    
    # Clean up old backups (keep last 5)
    echo "Cleaning up old backups..."
    ls -t "${backup_dir}"/backup_*.tar.gz | tail -n +6 | xargs -r rm
    echo "Backup process completed"
else
    echo "Error: Backup failed"
    exit 1
fi
```