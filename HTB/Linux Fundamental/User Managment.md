# User Management

#### Execution as a user

```shell
DixLan@htb[/htb]$ cat /etc/shadow

cat: /etc/shadow: Permission denied
```

#### Execution as root           

```shell
DixLan@htb[/htb]$ sudo cat /etc/shadow

root:<SNIP>:18395:0:99999:7:::
daemon:*:17737:0:99999:7:::
bin:*:17737:0:99999:7:::
<SNIP>
```

Here is a list that will help us to better understand and deal with user management.

| **Command** | **Description**                                              |
| ----------- | ------------------------------------------------------------ |
| `sudo`      | Execute command as a different user.                         |
| `su`        | The `su` utility requests appropriate user credentials  via PAM and switches to that user ID (the default user is the  superuser). A shell is then executed. |
| `useradd`   | Creates a new user or update default new user information.   |
| `userdel`   | Deletes a user account and related files.                    |
| `usermod`   | Modifies a user account.                                     |
| `addgroup`  | Adds a group to the system.                                  |
| `delgroup`  | Removes a group from the system.                             |
| `passwd`    | Changes user password.                                       |

## Questions

 Which option needs to be set to create a home directory for a new user using "useradd" command? `-m`

Which option needs to be set to lock a user account using the "usermod" command? (long version of the option) `--lock`

 Which option needs to be set to execute a command as a different user using the "su" command? (long version of the option) `--command`