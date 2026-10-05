# Lab 02: Troubleshooting File Access Permissions

## Objective
Reproduce and resolve two “Permission denied” errors on macOS,
using targeted permission changes and verifying normal user access.

## Environment
- macOS
- Tools: Terminal, chmod, cat, ls
- Sample file: ~/IT-Support-Lab/permissions-case/handoff.txt
- Controlled lab using a dedicated practice folder
- No sudo or recursive permission changes used

## Scenario
A user can see a project folder but cannot read its handoff notes.

## Working Baseline
The sample file contained:
“Internal project handoff: testing file access.”

Initial permissions:
- File: -rw-r--r--
- Parent folder: drwxr-xr-x

The current user owned both items, and cat displayed the file.

## Case A: Missing File Read Permission

### Introduce the fault
`chmod u-r ~/IT-Support-Lab/permissions-case/handoff.txt`

### Collect evidence
`cat ~/IT-Support-Lab/permissions-case/handoff.txt`
Returned: Permission denied.

`ls -l ~/IT-Support-Lab/permissions-case/handoff.txt`
Showed: --w-r--r--

### Diagnosis
The owner lacked read permission. Under the standard Unix
permission bits in this lab, the owner did not fall back to
the read permissions assigned to the group or others.

### Fix
`chmod u+r ~/IT-Support-Lab/permissions-case/handoff.txt`

### Verify
cat displayed the original text.
ls -l showed restored permissions: -rw-r--r--.

## Case B: Missing Directory Traversal Permission

### Introduce the fault
`chmod u-x ~/IT-Support-Lab/permissions-case`

### Collect evidence
`cat ~/IT-Support-Lab/permissions-case/handoff.txt`
Returned: Permission denied.

`ls -ld ~/IT-Support-Lab/permissions-case`
Showed: drw-r-xr-x.

### Diagnosis
The file's read permission had already been restored, but the
owner lacked traversal permission on its parent directory.

For directories, execute permission allows traversal and
access to entries inside. Read permission alone is insufficient
to open a file within the directory.

### Fix
`chmod u+x ~/IT-Support-Lab/permissions-case`

### Verify
cat displayed the original text.
ls -ld showed restored directory permissions: drwxr-xr-x.

## Outcome
Both faults were resolved by restoring only the missing
owner permission. Existing group and other permissions
were left unchanged.

## Lessons Learned
- The same error can have different underlying causes.
- Inspect both the file and its parent directory.
- File read permission and directory traversal are distinct.
- Avoid broad changes such as chmod 777.
- Verify the affected user's task after applying the fix.

## Scope
This lab covered standard Unix permission bits on a local Mac.
ACLs, network shares, and Windows NTFS permissions were not tested.
