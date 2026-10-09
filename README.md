# RHCSA Practice — Standard Linux Permissions
Today’s objective: Read, set, and verify standard user/group/other permissions using symbolic and numeric notation.
Estimated time: 45 minutes
Lab system: Your RHEL 9 VM is suitable for this practice.
Safety: This lab works entirely inside /tmp/rhcsa-permissions-lab. It does not change networking, storage, boot settings, authentication, SELinux, the firewall, or system availability.
Red Hat’s current EX200 objectives include listing and changing standard ugo/rwx permissions, managing default permissions, and diagnosing file-permission problems. The current published exam is based on RHEL 10, but the commands practiced here work the same way on RHEL 9. The exam is performance-based, so the important skill is performing and verifying the task without assistance. Official Red Hat EX200 objectives
1. The concept in plain language — 10 minutes
Every standard Linux file or directory has permissions for three identities:
- u — user who owns the item
- g — group that owns the item
- o — others, meaning everyone else
The permissions are:
- r — read
- w — write
- x — execute or access
A long listing might show:
-rw-r-----

Break it into sections:
- | rw- | r-- | ---
    user  group others

That means:
- Owner can read and modify the file.
- Group members can read it.
- Everyone else has no access.
For directories, permissions behave differently:
- r lists the directory’s filenames.
- w creates, removes, or renames entries.
- x enters or traverses the directory.
A user normally needs both r and x to browse a directory comfortably.
Numeric permissions
Each permission has a value:
r = 4
w = 2
x = 1

Add the values for each identity:
Number	Permission	Meaning
7	rwx	Read, write, execute
6	rw-	Read and write
5	r-x	Read and execute
4	r--	Read only
0	---	No permissions


Therefore:
640 = rw-r-----
750 = rwxr-x---

2. Guided practice — 20 minutes
This portion is intentionally guided, so the commands and explanations are provided together.
Starting state
Open a terminal on your RHEL VM. You may use your normal account or the root account.
Confirm your identity:
whoami
id

- whoami displays the current username.
- id shows your user ID, primary group, and supplementary groups.
- This matters because Linux uses these identities when deciding whether access is allowed.
Create a clean practice area:
rm -rf /tmp/rhcsa-permissions-lab
mkdir -p /tmp/rhcsa-permissions-lab/application/{config,logs,scripts}
cd /tmp/rhcsa-permissions-lab/application

- rm -rf removes only the named temporary lab directory if an earlier copy exists.
- mkdir -p creates the application directory and its three subdirectories.
- The braces expand into config, logs, and scripts.
- cd moves into the new application directory.
Create sample application files:
printf '%s\n' 'database_server=db01.example.test' > config/application.conf
printf '%s\n' 'Application started successfully' > logs/application.log
printf '%s\n' '#!/bin/bash' 'echo "Application health check passed"' > scripts/healthcheck.sh

These commands create:
- A configuration file
- An application log
- A small health-check script
Inspect the starting permissions:
ls -ld config logs scripts
ls -l config/application.conf logs/application.log scripts/healthcheck.sh

ls -ld displays the directory itself instead of listing its contents. ls -l displays file types, permissions, ownership, size, and timestamps.
Configure the directories
The application owner needs full access. Members of the owning group should be able to enter and read the directories. Everyone else should receive no access.
chmod 750 config logs scripts

750 means:
Owner: rwx = 7
Group: r-x = 5
Other: --- = 0

Verify:
stat -c '%A %a %U %G %n' config logs scripts

Expected permission portion:
drwxr-x--- 750

Your displayed owner and group depend on the account running the lab.
Protect the configuration file
The owner should be able to read and modify it. The group should only be able to read it. Others should have no access.
Use symbolic notation:
chmod u=rw,g=r,o= config/application.conf

- u=rw assigns read and write to the owner.
- g=r assigns read-only access to the group.
- o= removes all permissions from others.
Verify:
stat -c '%A %a %U %G %n' config/application.conf

Expected result:
-rw-r----- 640

Configure the log file
Give the owner read/write access and the group read-only access:
chmod 640 logs/application.log

Verify both files together:
ls -l config/application.conf logs/application.log

Both should begin with:
-rw-r-----

Make the health-check script executable
First, try running it:
./scripts/healthcheck.sh

You should initially receive Permission denied because the execute permission has not been assigned.
Add execute permission only for the owner:
chmod u+x scripts/healthcheck.sh

This is symbolic notation:
- u selects the owner.
- + adds a permission without replacing the existing ones.
- x adds execute permission.
Verify and run it:
ls -l scripts/healthcheck.sh
./scripts/healthcheck.sh

Expected output:
Application health check passed

The permissions will probably appear as:
-rwxr--r--

Now protect it so only the owner and group can access it:
chmod 750 scripts/healthcheck.sh
stat -c '%A %a %n' scripts/healthcheck.sh

Expected result:
-rwxr-x--- 750 scripts/healthcheck.sh

Diagnose all permissions at once
find /tmp/rhcsa-permissions-lab -printf '%M %m %u %g %p\n'

This reports:
- %M — symbolic permissions
- %m — numeric permissions
- %u — owner
- %g — group
- %p — complete path
You should find these important results:
750 application/config
750 application/logs
750 application/scripts
640 application/config/application.conf
640 application/logs/application.log
750 application/scripts/healthcheck.sh

The parent directories may have different permissions because we did not change them.
3. Realistic troubleshooting task — 5 minutes
Suppose an administrator accidentally removes the owner’s read permission from the configuration file:
chmod u-r config/application.conf
ls -l config/application.conf

The owner’s permission section now lacks r.
Restore the required state:
chmod 640 config/application.conf

Verify:
test "$(stat -c '%a' config/application.conf)" = "640" \
  && echo "PASS: configuration permissions are correct" \
  || echo "FAIL: configuration permissions are incorrect"

Expected result:
PASS: configuration permissions are correct

This illustrates an exam habit I strongly recommend: never assume a command worked. Verify the final state explicitly.
4. Independent challenge — 7 minutes
Do this without looking back at the guided commands.
Create the following structure inside the current application directory:
reports/
└── daily-report.txt

Requirements:
1. The reports directory must allow:
   - Owner: read, write, and enter
   - Group: read and enter
   - Others: no access
2. daily-report.txt must contain:
Daily application report
Status: Healthy

3. The report must allow:
   - Owner: read and write
   - Group: read only
   - Others: no access
4. Verify both symbolic and numeric permissions.
5. Display the file contents.
6. Do not use sudo for this challenge.
Your finished permissions should be:
reports                 750
reports/daily-report.txt 640

5. Skills check — 3 minutes
Answer these without running commands first:
1. What does chmod 640 filename permit the owner to do?
2. Why does a directory commonly need x in addition to r?
3. What is the numeric equivalent of rwxr-x---?
4. What command adds execute permission for the owner without changing other permissions?
5. What is the difference between these commands?
chmod u+x healthcheck.sh
chmod u=x healthcheck.sh

6. Which command can display both numeric and symbolic permissions?
7. If a file is correctly configured but its parent directory lacks x, can a user normally access that file?
Confidence standard
I would consider today’s objective mastered when you can independently:
- Interpret a permission string such as -rw-r-----
- Convert between rwx and numeric notation
- Use both symbolic and numeric chmod
- Explain directory execute permission
- Verify the result with ls, stat, or find
- Correct a deliberately misconfigured permission
Safe cleanup
When you finish the independent challenge, remove only this temporary lab:
cd /tmp
rm -rf /tmp/rhcsa-permissions-lab

Verify that it is gone:
test ! -e /tmp/rhcsa-permissions-lab \
  && echo "Cleanup complete" \
  || echo "Lab directory still exists"

Expected result:
Cleanup complete
