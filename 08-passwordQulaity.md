# Password Quality Management in Linux

This guide explains how to configure and enforce password quality requirements in Linux systems to enhance security by preventing weak passwords.

## The Password Quality Configuration File

The `/etc/security/pwquality.conf` file in Linux systems is used to configure password quality requirements for user accounts. This file is typically part of the `libpwquality` library, which provides mechanisms for enforcing strong password policies.

## Purpose and Function

The main function of this configuration file is to enforce password complexity and strength policies. These policies help:

- Prevent users from setting easily guessable passwords
- Reduce the risk of successful brute force attacks
- Ensure compliance with security standards and best practices
- Maintain a consistent password policy across the system

## Common Parameters

The `/etc/security/pwquality.conf` file supports various parameters to control different aspects of password quality:

| Parameter | Description |
|-----------|-------------|
| `minlen` | Specifies the minimum length of passwords |
| `maxrepeat` | Sets the maximum number of allowed consecutive identical characters in a password |
| `maxclassrepeat` | Defines the maximum number of allowed consecutive characters from the same character class (lowercase, uppercase, digits, symbols) |
| `dcredit` | Controls the number of digits required in a password |
| `ucredit` | Specifies the number of uppercase letters required |
| `lcredit` | Defines the number of lowercase letters needed |
| `ocredit` | Sets the number of other characters (such as symbols) required |
| `difok` | Specifies the number of characters that must be different from the old password |
| `enforce_for_root` | Determines whether the password quality checks apply to the root user |
| `dictcheck` | Enables or disables dictionary checking |
| `dictpath` | Specifies the path to the dictionary file for dictionary checks |

## Understanding Credit Values

For the credit parameters (`dcredit`, `ucredit`, `lcredit`, `ocredit`):

- **Negative values**: Require at least that many characters of that class
  - Example: `dcredit = -2` means at least 2 digits are required
  
- **Positive values**: Provide a maximum credit for having characters of that class
  - Example: `dcredit = 1` means having digits will count as a credit of up to 1 toward the minimum length

## Example Configuration

Here's an example of a strong password policy configuration:

```ini
# Require at least 12 characters
minlen = 12

# Require at least 1 digit
dcredit = -1

# Require at least 1 uppercase letter
ucredit = -1

# Require at least 1 lowercase letter
lcredit = -1

# Require at least 1 special character
ocredit = -1

# No more than 3 consecutive identical characters
maxrepeat = 3

# No more than 2 consecutive characters from the same class
maxclassrepeat = 2

# At least 4 characters must be different from the old password
difok = 4

# Apply these rules to the root user as well
enforce_for_root = 1
```

## Testing Password Quality

You can test if a password meets the configured requirements using the `pwscore` command:

```bash
echo "MyNewPassword123!" | pwscore
```

The command returns a score from 0 to 100, with higher scores indicating stronger passwords. A score below 50 typically means the password doesn't meet the requirements.

## Implementing Password Quality Checks

The password quality settings are enforced through PAM (Pluggable Authentication Modules). To ensure the settings are applied, check that the following line exists in `/etc/pam.d/system-auth` and `/etc/pam.d/password-auth`:

```
password    requisite     pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type=
```

## Additional Password Security Measures

Beyond the basic password quality settings, consider implementing:

1. **Password History**: Prevent users from reusing recent passwords
   ```
   password    sufficient    pam_unix.so sha512 shadow nullok try_first_pass use_authtok remember=5
   ```

2. **Account Lockout**: Lock accounts after failed login attempts
   ```
   auth        required      pam_faillock.so preauth silent audit deny=5 unlock_time=1800
   ```

3. **Password Aging**: Force regular password changes (configured in `/etc/login.defs`)
   ```
   PASS_MAX_DAYS   90
   PASS_MIN_DAYS   1
   PASS_WARN_AGE   7
   ```

## Best Practices for Password Policies

1. **Balance security and usability**: Overly complex requirements may lead users to write down passwords
2. **Educate users** about creating strong, memorable passwords
3. **Consider using passphrases** instead of complex passwords
4. **Implement multi-factor authentication** when possible
5. **Regularly audit password policies** to ensure they meet current security standards
6. **Test password strength** against common attack methods

By properly configuring password quality requirements, you can significantly enhance the security posture of your Linux systems while maintaining a balance with usability.