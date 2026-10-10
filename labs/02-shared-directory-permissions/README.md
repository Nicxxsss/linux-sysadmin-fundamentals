# Shared directory permissions

In this lab, I configured a directory that can be used by members of a specific Linux group while remaining inaccessible to other local users.

The work was performed on the Debian 13 VM created for this project.

## Creating the users and group

I created two users to represent people working with security reports:

```bash
sudo adduser analyst
sudo adduser developer
sudo groupadd security
sudo usermod -aG security analyst
sudo usermod -aG security developer
```

The `-aG` options add a supplementary group without replacing the user's existing group memberships.

The resulting group membership is available in [`group-membership.txt`](../../evidence/02-shared-directory-permissions/group-membership.txt).

## Configuring the shared directory

I created `/srv/security-reports` and assigned it to `root` and the `security` group:

```bash
sudo mkdir /srv/security-reports
sudo chown root:security /srv/security-reports
sudo chmod 2770 /srv/security-reports
```

The owner and group have full access, while other users have no permissions. The leading `2` enables `setgid`, so files created inside the directory inherit the `security` group.

## File permission problem

The first file created by `analyst` inherited the correct group, but `developer` could not modify it. The file was created without group write permission.

I corrected the existing file with:

```bash
sudo chmod g+w /srv/security-reports/incident-001.txt
```

This fixed that file, but it did not solve the same problem for files created in the future. The `setgid` bit controls group ownership; it does not guarantee group write permission.

## Default ACL

I installed the ACL utilities and added a default ACL to the directory:

```bash
sudo apt install acl -y
sudo setfacl -m d:u::rwx,d:g::rwx,d:o::---,d:m::rwx /srv/security-reports
```

New files now inherit permissions that allow members of `security` to work on them. Users outside the group remain blocked.

The directory permissions and complete ACL are recorded in [`default-acl.txt`](../../evidence/02-shared-directory-permissions/default-acl.txt).

## Access test

For the final test:

- `analyst` created a file;
- `developer` modified and read the same file;
- `contractor`, which is not a member of `security`, received `Permission denied`.

The command output and exit statuses are recorded in [`access-tests.txt`](../../evidence/02-shared-directory-permissions/access-tests.txt).

This test also showed that having `sudo` available does not automatically bypass permissions during normal commands. My regular `user1` account could not read the file without explicitly running the command through an authorized user or elevated privileges. 
