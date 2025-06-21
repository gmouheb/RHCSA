# Lab 2: Process Management and System Tuning

This lab focuses on managing processes, adjusting process priorities, and configuring system performance profiles.

## Tasks

1. Launch the command `dd if=/dev/zero of=/dev/null` three times as a background job.

2. Increase the priority of one of these commands using the nice value -5. Change the priority of the same process again, but this time use the value -15. Observe the difference.

3. Kill all the dd processes you just started.

4. Ensure that tuned is installed and active, and set the throughput-performance profile.