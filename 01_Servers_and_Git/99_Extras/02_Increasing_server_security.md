# Summary
In the following instructions, we'll look at the procedures for &ldquo;hardening&rdquo; your DigitalOcean Droplet, making it more difficult for malicious hackers to compromise your server.

Here are the steps we will cover:

1. Create a new, alternative user to `root`.
2. Grant this new user administrative privileges.
3. Disable SSH access from the `root` user.
4. Cover how to run commands as an administrator as your new user.
5. Install fail2ban to block repeated, failed, login attempts.


## Create a new user
The `root` user on your account has the highest priveleges; because of that, if a hacker gains access to your server as the `root` user, they have a lot of power to do a lot of harm.

Because of this, we're going to disable `root` from being able to SSH in to your server.

Before we do that, though, we need to create a new, alternative user that *can* SSH into your server.

Begin by SSH'ing into your Droplet as `root`:

```bash
$ ssh root@your-droplet-ip-address
```

Next we'll use the command `adduser` followed by a username of your choice.

During this procedure, you'll be prompted to create a password for this new user. Strong passwords are essential to prevent brute force attacks (where a machine is used to try and guess your password). If you're not confident in your ability to generate a secure password, use a [password generator tool](http://correcthorsebatterystaple.net/).

With your password chosen, initiate the new user (replace `newuser` with a username of your choosing):

```bash
$ adduser susanbuck
```

The process will look like this:
```
Adding user 'susanbuck' ...
Adding new group 'susanbuck' (1000) ...
Adding new user 'susanbuck' (1000) with group 'susanbuck' ...
Creating home directory '/home/susanbuck' ...
Copying files from '/etc/skel' ...
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Changing the user information susanbuck
Enter the new value, or press ENTER the default
	Full Name []: Susan Buck
	Room Number []:  
	Work Phone []:
	Home Phone []:
	Other []:
Is the information correct? [Y/n] y
```


## Give your new user admin privileges
Next, we want to give your new user administrator privileges.

While still signed in as the root user, run:

```bash
$ visudo
```

This command uses nano to open the files that controls your server's `sudoers` (more on `sudo` later.)

Search for the line that looks like this:

```bash
root    ALL=(ALL:ALL) ALL
```

Below that line, add *new* line that looks like the following (replace `susanbuck` with your username):

```bash
susanbuck ALL=(ALL:ALL) ALL
```

Save your changes (`ctrl` + `x`, then `y`, then *Enter*).


## SSH access for your new user
When you first set up DigitalOcean, you created a SSH key connection between your computer and your server. This allowed you to SSH into your Droplet as root, without having to enter a password.

You'll want the same convenience/security that SSH keys provide for your new user as well, so lets set that up.

(In the following steps, it's assumed that you've already created a public key, `id_rsa.pub`, when you first followed the [Setup Git notes](https://github.com/susanBuck/dwa15-fall2015-notes/blob/master/01_Servers_and_Git/07_Setup_Github.md).)

Open a *new* Terminal/Cmder window, so we can grab your public key from your local machine, without losing your connection to your Droplet.

In the new window, on your local computer, get the contents of your `id_rsa.pub` file using the cat command.

```bash
$ cat ~/.ssh/id_rsa.pub
```

The output will look something like this:

```bash
ssh-rsa [LONG STRING OF RANDOM CHARACTERS] your@email.com
```

Select the entire public key, and copy it.

Switch back to the Terminal/Cmder window that's SSH'd as root into your Droplet and run the following command to switch to your new user (replace `susanbuck` with your username):

```bash
$ su - susanbuck
```

Your new user does not have any SSH settings yet, so let's start by creating a new `.ssh` directory:

```bash
$ mkdir ~/.ssh
```

And, for security purposes, restrict the permissions on this new directory:

```bash
$ chmod 700 .ssh
```

Now we'll open a file called `authorized_keys` using nano:

```bash
$ nano ~/.ssh/authorized_keys
```

Now insert your public key (which should be in your clipboard).

Save and exit (`ctrl` + `x`, then `y`, then *Enter*).

And once again, we'll secure things up with permissions:

```bash
$ chmod 600 ~/.ssh/authorized_keys
```

To confirm this worked, log out of your Droplet by running the `exit` command:

```bash
susanbuck@Sept-17-2015:~$ exit
logout
Connection to 45.55.221.4 closed.
```

Then try to SSH back in as your new user:
```
$ ssh susan@45.55.221.4
```

If everything went well, you should be logged in again.


## Change ownership of /var/www/html
If you try to edit files in your `/var/www/html` as your new user, you'll run into permission issues because that directory is owned by `root`. You can get around permission issues using the `sudo` command, but to make things easier, you can instead change the ownership of that directory to belong to your new user, with the following command.

Replace `susanbuck` with your username.

```bash
$ chown -R susanbuck /var/www/html
```


## Disable root access
Now that you've confirmed you can SSH in as your new user, it's safe to disable `root` from SSH'ing into the server.

This is done via the config file, `/etc/ssh/sshd_config`.

Typically, to edit this file, we'd use nano, make some changes, save, and be done.

However, if you were to try to do that now, you'd receive the following error when trying to save:

```
Error writing /etc/ssh/sshd_config: Permission denied
```

This doesn't seem to make sense... Didn't we just give our new user administrative privileges in the steps above?

We did! *However*, in order to utilize these privileges, we need to prefix commands with `sudo`.

> sudo (/ˈsuːduː/ or /ˈsuːdoʊ/) is a program for Unix-like computer operating systems that allows users to run programs with the security privileges of another user, by default the superuser. The name is a contraction of "do as su" where "su" is an abbreviation for "super user." -[ref](https://en.wikipedia.org/wiki/Sudo)

Moving forward, if you ever try a command and are told you don't have permissions, try the command again but prefix it with `sudo`.

So, to open `sshd_config` with admin privileges, run this command:

```
$ sudo nano /etc/ssh/sshd_config
```
You will be prompted to enter the password for your user.

Once the file is opened, look for the line `PermitRootLogin yes` and toggle `yes` to `no`.

When done, save and exit (`ctrl` + `x`, then `y`, then *Enter*).

To make this change take effect, run the following command to restart your SSH services:

```bash
$ sudo service ssh restart
```

Note how that command was again prefixed with `sudo` since restarting SSH is something only an administrator can do.





## Confirm root user no longer has SSH access
Let's just confirm the above changes worked and the `root` user can no longer SSH into your Droplet.

In a new Terminal/Cmder window, try to SSH in as root:

```
$ ssh root@your-server-ip-address
```

You'll note that it will now continually ask for a password, instead of just letting you in as it previously did with your SSH keys set up. Even if you had the password, it would still not let you in.

You might look at this whole procedure and think we're chasing our tails. We prevented the `root` user from logging in because it's dangerous because `root` has administrative privileges.

*But*, we also created a *new* user that also had administrative privileges.

Isn't this just as bad?

It's not, and the reason is because hackers target `root`. They know many, many servers start with a default admin login with the username `root`, and not all server admins will go to the trouble to disable `root`. Because of this, they target their attacks on `root` since it provides them the greatest payoff.

By using a non-default username, you're greatly decreasing the chances a hacker will brute force enter your server.


## Ban failed login attempts

There's a file on your server, `/var/log/auth.log`, which logs authorization requests on your server.

Take a look at its contents:

```bash
$ sudo cat /var/log/auth.log
```

If you look at this file that's been running on a server for even a short time, you'd be amazed by how many outside login attempts are being thrown at your server by malicious hackers you don't know.

You'll likely see lots of login attempts for `root`, and other random user names as well. If you look up the IP address for these login attempts, you'll also see they're coming from all over the world.

By disabling `root` access and using SSH keys, we've reduced the chances these malicious login attempts will work, however, we can take it one step further by banning the IP addresses of machines that have repeated, failed, login attempts.

This is done using a software module called *Fail2Ban*. DigitalOcean has a [full write up on how to use Fail2Ban](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04), but below are the express instructions.

Tell apt-get to get the latest updates:
```bash
$ sudo apt-get update
```

Install fail2ban:
```bash
$ sudo apt-get install fail2ban
```

Make a copy of the default configs so you can make any needed updates:
```bash
$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
````

Restart fail2ban:
```bash
$ sudo service fail2ban restart
```

Note that with fail2ban implemented, you do stand the chance of accidentally banning yourself from your own server if you have too many failed login attempts.

The default threshold for getting banned is 3 failed login attempts.

If that happens, you will not be able to login from your banned IP address for the default time of 1 hour.

Note that these default settings can all be adjusted in your `/etc/fail2ban/jail.local` config file. In that file, you can also &ldquo;whitelist&rdquo; your IP address so it will not get banned. If your network provider uses dynamic IPs, however, this change will not be permanent.



## Take a Snapshot of your Current Configuration
You've now put a fair amount of work in to configuring your Droplet. Given that, you should take advantage of [DigitalOcean's *Snapshot* feature](https://github.com/susanBuck/dwa15-fall2015-notes/blob/master/01_Servers_and_Git/99_Extras/03_Digital_Ocean_Snapshots.md).


## Tips

### whoami
The `whoami` command can be used now (or anytime) to tell you who you're logged in as. This can be useful if you're switching between root and your new user


### passwd
To update your user password, use the `passwd` command.


### What users exist?
To see what users exist on your server, run `cat /etc/passwd`

Note that in addition to the new user you created above, and `root`, there are many other default users on your server. These users do not have full administrator privileges like you and `root` do.


## Reference:
+ [How To Add and Delete Users on an Ubuntu 14.04 VPS](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps)
+ [Initial Server Setup with Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04)
