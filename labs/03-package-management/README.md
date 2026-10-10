# Installing and removing a Debian package

For this lab, I used the `tree` utility to practice the basic lifecycle of a Debian package: checking whether it was installed, researching it, installing it, identifying its files and removing it.

## Checking the package

The initial check showed that the `tree` command was not available:

```bash
command -v tree
```

Before installing anything, I searched the local APT metadata:

```bash
apt-cache search "^tree$"
apt-cache policy tree
apt-cache show tree
```

The package was available from the Debian `trixie/main` repository, but its installed state was `(none)`.

The package description, candidate version and repository information are recorded in [`package-research.txt`](../../evidence/03-package-management/package-research.txt).

## Installation and verification

I installed the package through APT:

```bash
sudo apt install tree
```

I then checked the executable path, program version and installed package version:

```bash
command -v tree
tree --version
apt-cache policy tree
```

The installed executable was `/usr/bin/tree`, running version `2.2.1`. APT reported package version `2.2.1-1`.

I used `dpkg` to confirm which installed package owned the executable:

```bash
dpkg -S /usr/bin/tree
```

The result linked `/usr/bin/tree` back to the `tree` package. APT handled the repository and installation, while `dpkg` provided information about the package installed on the local system.

## Permission check while testing

My first functional test used the directory from the previous permissions lab:

```bash
tree -L 2 /srv/security-reports
```

It returned `error opening dir`. The program was installed correctly, but `user1` did not have permission to access that directory.

I repeated the test with `/etc/ssh`, which is readable by the regular user, and `tree` displayed its contents successfully. This confirmed that the failure came from filesystem permissions rather than the package installation.

The installation checks and both tests are available in [`installation-validation.txt`](../../evidence/03-package-management/installation-validation.txt).

## Removing the package

After testing, I removed the package:

```bash
sudo apt remove tree
```

A final check showed that the executable was no longer available. `apt-cache policy` returned `Installed: (none)`, while the same version remained available as a candidate in the Debian repository.

The APT history recorded both operations, including the package version and the user who requested them. The result is stored in [`removal-validation.txt`](../../evidence/03-package-management/removal-validation.txt).

`apt remove` removes the installed package but may leave package-managed configuration files. `apt purge` also removes those configuration files. The distinction had little practical effect here because `tree` is a small command-line utility without a service configuration in `/etc`.
