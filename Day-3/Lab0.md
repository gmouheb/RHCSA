# Lab 0: Shared Group Environment Configuration

This lab focuses on setting up a shared group environment with specific directory permissions, group ownership inheritance, and file deletion restrictions.

## Requirements

1. Set up a shared group environment:
   - If you haven't created these directories in a previous exercise yet, create two directories: `/data/account` and `/data/sales`
   - Make the group `sales` the owner of the directory `/data/sales`
   - Make the group `account` the owner of the directory `/data/account`

2. Configure the permissions:
   - The user owner (which must be root) and group owner should have full access to the directory
   - There should be no permissions assigned to the others entity

3. Configure group ownership inheritance:
   - Ensure that all new files in both directories inherit the group owner of their respective directory
   - This means that all files created in `/data/sales` will be owned by the group `sales`
   - All files created in `/data/account` will be owned by the group `account`

4. Configure file deletion restrictions:
   - Ensure that users are only allowed to remove files of which they are the owner