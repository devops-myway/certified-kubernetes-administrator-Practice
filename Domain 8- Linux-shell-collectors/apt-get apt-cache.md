

#### APT (Advanced Package Tool)

##### Apt-Get

apt-get is for installing, upgrading, and cleaning packages in Ubuntu-based Linux distribution.

#### Using apt-get commands
apt-get basically works on a database of available packages.  If you don’t update this database, the system won’t know if there are newer packages available or not. Once you have updated the package database, you can upgrade the installed packages.

``````sh
sudo apt-get update

sudo apt-get upgrade

sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install <package_name>
sudo apt-get install <package_name>=<version_number>
sudo apt-get remove <package_name>
sudo apt-get purge <package_name>
sudo apt-get clean

--- example install containerd package
sudo apt-get update && apt-get upgrade
sudo apt-get install containerd -y
``````

##### APT-CACHE 
while apt-cache command is used for finding new packages in Ubuntu-based Linux distribution.
The location of APT cache is /var/lib/apt/lists/ directory. Which repository metadata to cache depends on the repositories added in your source list in the
/etc/apt/sources.list file and additional repository files located in ls /etc/apt/sources.list.d directory.

will list the versions available from all your sources. madison is an apt-cache subcommand, man apt-cache

apt-cache madison is not emphasized because most of what it displays is also available through apt-cache showpkg and apt-cache policy

policy: shows priorities, either of all repositories, or of the packages given as arguments
showpkg shows forward and reverse dependencies.

``````sh
apt-cache search package_name
apt-cache showpkg package_name

sudo apt-cache madison package_name

sudo apt-cache showpkg google-chrome-stable
sudo apt-cache madison google-chrome-stable


``````


