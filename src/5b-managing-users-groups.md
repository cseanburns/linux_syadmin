# Managing Users and Groups

1. **Understanding User and Group Management**: Learn how to create, modify, and delete user accounts and groups on a Linux system using essential tools like `useradd`, `userdel`, `groupadd`, and `groupdel`.
1. **Working with System Files**: Gain familiarity with critical system files like **/etc/passwd**, **/etc/shadow**, and **/etc/group**, and understand how user, password, and group information is stored and managed.
1. **Customizing User Account Settings**: Modify default settings for new user accounts by editing configuration files such as **/etc/skel** and **/etc/adduser.conf**, allowing for customization of new user environments.
1. **Password and Account Security**: Develop skills in securing user accounts by setting password expiration policies and managing password lifetimes through the **/etc/shadow** file and commands like `passwd` and `chage`.
1. **Group-Based Permissions and Shared Resources**: Learn to create and manage groups, assign users to groups, and configure shared directories with appropriate permissions to facilitate collaborative work among users.
1. **File Permissions and Directory Management**: Understand how to change ownership and permissions of directories using tools like `chmod` and `chgrp` to control access based on user and group roles.
1. **Practical User Management Tools**: Apply hands-on experience using
utilities such as `gpasswd`, `su`, `sudo`, and `nano` to manage users and
groups, edit system files, and adjust account settings on a Linux system.

## Getting Started

In some cases we'll want to provide user accounts on the servers we administrate, or we'll want to set up servers for others to use.
In doing so, we have to setup some basic permissions to keep various home directories protected from the prying eyes of other users on the system.

The process of creating accounts is fairly straightforward, but there are a few things to know about how user accounts work.

## The passwd file

The **/etc/passwd** file contains information about the users on your system.
There is a `man` page that describes the file.
`man` pages are divided into sections (see ``man man``), and the man page for the ``passwd`` file is in section 5.
Therefore in order to read the man page for the **/etc/passwd** file, we run the following command:

```
man 5 passwd
```

Before we proceed, let's take a look at a single line of the file.
Below I'll show the output for a made up user account:

```
grep "peter" /etc/passwd
peter:x:1000:1000:peter,,,:/home/peter:/bin/bash
```

The line starting with **peter** is a colon separated line.
That means that the line is composed of multiple fields each separated by a colon.

``man 5 passwd`` tells us what each field indicates.
The first field is the login name, which in this case is **peter**.
The second field, marked **x**, marks the password field.
This file does not contain the password, though.
The passwords, which are [hashed and salted][hashedSalted], for users are stored in the **/etc/shadow** file.
This file can only be read by the root user (or using the ``sudo`` command).

> Hashing a file or a string of text is a process of running a hashing algorithm on the file or text.
> If the file or string is copied exactly, byte for byte, then hashing the copy will return the same value.
> If anything has changed about the file or string, then the hash value will be different.
> By implication, this means that if two users on a system use the same password, then the hash of each will be equivalent.
> Salting a hashed file (or file name) or string of text is a process of adding random data to the file or string.
> Each password will have a unique and mostly random salt added to it.
> This means that even if two users on a system use the same password, salting their passwords will result in unique values.

The third column indicates the user's numerical ID, and the fourth column indicates the users' group ID.
The fifth column repeats the login name, but could also serve as a comment field.
Comments are added using certain commands (discussed later).
The fifth field identifies the user's home directory, which is **/home/peter**.
The sixth field identifies the user's default shell, which is ``/bin/bash``.

The **user name or comment** field merely repeats the login name here, but it can hold specific types of information.
We can add comments using the ``chfn`` command.
Comments include the user's full name, their home and work phone numbers, their office or room number, and so forth.
To add a full name to user **peter**'s account, we use the **-f** option:

```
sudo chfn -f "Peter Parker" peter
```

The **/etc/passwd** file is a standard Linux file, but some things will change depending on the Linux distribution.
For example, the user and group IDs above start at 1000 because **peter** is the first human account on the system.
This is a common starting numerical ID nowadays, but it could be different on other Linux or Unix-like distributions.
The home directory could be different on other systems, too;
for example, the default could be located at **/usr/home/peter**.
Also, other shells exist besides ``bash``, like [zsh][zsh], which is now the default shell on macOS;
so other systems may default to different shell environments.

## The shadow file

The **/etc/passwd** file does not contain any passwords but a simple **x** to mark the password field.
Passwords on Linux are stored in **/etc/shadow** and are hashed with **sha512**, which is indicated by **$6$**.
You need to be root to examine the shadow file or use ``sudo``:

The fields are (see ``man 5 shadow``):

* login name (username)
* encrypted password
* days since 1/1/1970 since password was last changed
* days after which password must be changed
* minimum password age
* maximum password age
* password warning period
* password inactivity period
* account expiration date
* a reserved field

The **/etc/shadow** file should not be directly edited.
To set, for example, a warning that a user's password will expire, we would use the ``passwd`` command (see ``man passwd`` for options).
The following command would make it so the user **peter** is warned that their password will expire in 14 days: 

```
passwd -w 14 peter
```

## The group file

The **/etc/group** file holds group information about the entire system (see `man 5 group`).
By default the file can be viewed by anyone on a system, but there is also a `groups` command that will return the groups for a user.
See: `man groups`
Running the `groups` command by itself will return your own memberships.

## Management Tools

There are different ways to create new users and groups, and the following list includes most of the utilities to help with this.
Note that, based on the names of the utilities, some of them are repetitive.

* useradd (8) - create a new user or update default new user information
* usermod (8) - modify a user account
* userdel (8) - delete a user account and related files
* groupadd (8) - create a new group
* groupdel (8) - delete a group
* groupmod (8) - modify a group definition on the system
* gpasswd (1) - administer /etc/group and /etc/gshadow
* adduser.conf (5) - configuration file for adduser(8) and addgroup(8) .
* adduser (8) - add a user or group to the system
* deluser (8) - remove a user or group from the system
* delgroup (8) - remove a user or group from the system
* chgrp (1) - change group ownership

The numbers within parentheses above indicate the ``man`` section.
Therefore, to view the man page for the ``userdel`` command:

```
man 8 userdel
```

## Practice

### Modify default new user settings

Let's modify some default user account settings for new users, and then we'll create a new user account.

Before we proceed, let's review several important configuration files that establish some default settings:

- **/etc/skel**
- **/etc/adduser.conf**

The **/etc/skel** directory defines the home directory for new users.
Whatever files or directories exist in this directory at the time a new user account is created
will result in those files and directories being created in the new user's home directory.
We can view what those are using the following command:

```
ls -a /etc/skel/
```

The **/etc/adduser.conf** file defines the default parameters for new users.
It's in this file where the default starting user and group IDs are set,
where the default home directory is located (e.g., in **/home/**),
where the default shell is defined (e.g., ``/bin/bash``),
where the default permissions are set for new home user directories (e.g., **0755**) and more.

Let's change some defaults for **/etc/skel**.
We need to use ``sudo [command]`` or use ``su`` to become the root user.
I prefer to use ``sudo [command]`` since this is a bit safer than becoming root.
Let's edit the default **.bashrc** file:

```
sudo nano /etc/skel/.bashrc
```

We want to add these lines at the end of the file.
This file is a configuration file for ``/bin/bash``, and will be interpreted by Bash.
Therefore, lines starting with a hash mark are comments:

```
# Dear New User,
#
# I have made the following settings
# to make your life a bit easier:
#
# make "c" a shortcut for "clear"
alias c='clear'
```

Use ``nano`` again to create a README file.
This file will be added to the home directories of all new users.
Add any welcome message you want to add, plus any guidelines for using the system.

```
sudo nano /etc/skel/README
```

### Add new user account

After writing (saving) and exiting ``nano``, we can go ahead and create a new user named **linus**.

```
sudo adduser linus
```

We'll be prompted to enter a password for the new user, plus comments (full name, phone number, etc).
Any of these can be skipped by pressing enter.
You can see from the output of the ``grep`` command below that I added some extra information:

```
grep "linus" /etc/passwd
linus:x:1003:1004:Linus Torvalds,333,555-123-4567,:/home/linus:/bin/bash
```

We may want to set up some password conditions to help keep the new user account secure.
To do that, we can modify the minimum days before the password can be changed, the maximum days before the password expires,
the number of days before the user gets a warning to change their password, and the number of days of inactivity when the password is locked:

```
sudo chage -m 7 -M 90 -W 14 -I 14 linux
```

See `man chage` for details, but:

- `-m 7` sets the minimum password age to 7 days before the user can change their password.
- `-M 90` sets the maximum age of the password to 90 days.
- `-W 14` provides a 14 day warning to the user that the password will expire.
- `-I 14` locks the account after 14 days of inactivity.

You can see these values by grepping the shadow file:

```
sudo grep "linus" /etc/shadow
```

To log in as the new user, use the ``su`` command:

```
su linus
```

To exit the new user's account, use the ``exit`` command:

```
exit
```

As a sysadmin, you will want to regularly review and audit the `/etc/passwd` and the `/etc/shadow` files to ensure only
authorized users have access to the system.

### Add users to a new group

Because of the default configuration defined in **/etc/adduser.conf**, the **linus** user only belongs to a group of the same name.
Let's create a new group that both **linus** and **peter** belong to.
For that, we'll use the **-a** option for the ``gpasswd`` command.
We'll also make the user **peter** the group administrator using the **-A** option (see ``man gpasswd`` for more details).

```
sudo groupadd developers
sudo gpasswd -a peter developers
sudo gpasswd -A peter developers
sudo gpasswd -a linus developers
grep "developers" /etc/group
```

> Note: if a user is logged in when you add them to a group, they need to logout and log back in before the group membership goes into effect.

### Create a shared directory

One of the benefits of group membership is that members can work in a shared directory.

Let's make the **/srv/developers** a shared directory.
The **/srv** directory already exists, so we only need to create the **developers** subdirectory:

```
sudo mkdir /srv/developers
```

Now we change ownership of the directory so that it's group owned by the **developers** group that we created:

```
sudo chgrp developers /srv/developers
```

The directory ownership should now reflect that it's owned by the **developers** group:

```
ls -ld /srv/developers
```

We'll have to change the default permissions, which are currently set to **0755**:

```
ls -ld /srv
ls -ld /srv/developers
```

To allow group members to read and write to the above directory, we need to use the ``chmod`` command in a way we haven't yet.
Specifically, we add a leading **2** that sets the group identity.
The **770** indicates that the user and group owners of the directory have read, write, and execute permissions for the directory:

```
sudo chmod 2770 /srv/developers
```

Now either **linus** or **peter** can add, modify, and delete files in the **/srv/developers** directory.

### User account and group deletion

You can keep the additional user and group on your system, but know that you can also remove them.
The ``deluser`` and ``delgroup`` commands offer great options and may be preferable to the others utilities (see ``man deluser`` or ``man delgroup``).

If we want to delete the new user's account and the new group, these are the commands to use.
The first command will create an archival backup of **linus**' home directory and also remove the home directory and any files in it.

```
deluser --backup --remove-home linus
delgroup developers
```

## Conclusion

Knowing how to manage user accounts and manage passwords are key sysadmin skills needed to provide collaborative environments and keep our systems secure.
While the method to manage these things vary by operating system, the basic theory is the same across OSes.

In this section, we learned about important user management files like `/etc/passwd`, `/etc/shadow`, `/etc/group`, `/etc/skel`, and `/etc/adduser.conf`.
We continued to use `nano` to edit new configuration files, specifically `/etc/skel` and `/etc/adduser.conf`.

We covered the following new commands:

- `adduser`: add a user or group to the system
- `chage`: change user password expiry information
- `chgrp`: change group ownership
- `delgroup`: remove a user or group from the system
- `deluser`: remove a user or group from the system
- `gpasswd`: administer `/etc/group` and `/etc/gshadow`
- `groupadd`: create a new group
- `passwd`: the password file
- `su`: run a command with substitute user and group ID

[hashedSalted]:https://auth0.com/blog/adding-salt-to-hashing-a-better-way-to-store-passwords/
[zsh]:https://www.zsh.org/
