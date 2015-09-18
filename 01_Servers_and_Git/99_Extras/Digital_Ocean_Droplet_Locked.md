# DigitalOcean Droplet Locked / DDoS

Below are two ominous messages we've occasionally seen students face in regards to their DigitalOcean Droplets:

<img src='http://making-the-internet.s3.amazonaws.com/vc-message-ddos-on-digital-ocean@2x.png' style='max-width:100%; width:1000px' alt='Message notifying you of a DDoS attack on DigitalOcean'>

When this happens, it means DigitalOcean has locked down your droplet because, as the message states, it detected a &ldquo;large flood of traffic from one or more of your servers.&rdquo;

When this happens, it likely means a malicious hacker gained access to your Droplet and used it to perform DDoS attacks, which caused that large flood of outgoing traffic.

DDoS stands for *Distributed Denial of Service* and is a attack in which hackers use &ldquo;innocent&lrdquo; servers to do things like send spam mail, or try to overwhelm some other target server.

## What happened
So how did this happen? Well, it can be hard to track down exactly where the problem started.

One possible cause is a hacker gained access to your root password, allowing them to access your server, create themselves an account, and then install their DDoS program to start sending traffic out.

Because of this, it's always preferable to *not* use a password when logging into your server, which is one of the reasons we use the more secure alternative of SSH keys.

If you're not using a root password, another possible cause could have come from outdated or insecure software on your server.

DDoS is one of the various plagues web developers and system administrators face in the real world, across all hosting providers.

In these notes we're going to describe what to do to get back on track if your DigitalOcean Droplet has been suspended.

## What to do next

If you read the support ticket DigitalOcean creates for you after they shut you down, you'll see they link to this article: [How To Recover from a Compromised Droplet Sending an Outgoing Flood or DDoS](https://www.digitalocean.com/community/tutorials/how-to-recover-from-a-compromised-droplet-sending-an-outgoing-flood-or-ddos).

The long and short of their plan for recovery involves creating a new Droplet, and &ldquo;hardening&rdquo; that Droplet to try and prevent further DDoS attacks from happening. The article also outlines how to recover your data from your compromised Droplet, so that you can move it to your new Droplet.

Following in DigitalOcean's lead, we're going to outline similar steps, but under the context of our class setup.

Fortunately, in our cases, recovery is simple for the following reasons:

1. All our code is backed up in Git.
2. All our databases are built with Laravel Migrations, making the structure easy to create.
3. We're not dealing with real-world applications, so our data (for the majority of students), is still disposable.

That being said, here's what you should do to get back on track...

## Step 1) Create a new droplet

Create a new Droplet.

As a reminder on what settings you need for the droplet, [here's the flowchart](http://making-the-internet.s3.amazonaws.com/vc-digital-ocean-new-droplet@2x.png) taken from the notes, [Deploy to Digital Ocean](https://github.com/susanBuck/dwa15-fall2015-notes/blob/master/01_Servers_and_Git/10_Deploy_to_Digital_Ocean.md#new-droplet).

Once your Droplet is created, note the IP address, and then log in via SSH:

```bash
$ ssh root@your.new.ip.address
```

## Step 2) Secure your new Droplet.

The `root` user on your account has the highest priveleges; because of that, if a hacker gains access to your server as the `root` user, they have a lot of power to do a lot of harm.

Because of this, we're going to disable `root` from being able to SSH in to your server.

Before we do that, though, we need to create a new, alternative user that *can* log into your server.

So, begin by SSH'ing into your server as `root`:

```bash
$ ssh root@your-server-ip-address
```

Now, use the command `adduser` followed by a username of your choice, like the example below.

During this process, you'll be prompted to create a password for this new user. Strong passwords are essential to prevent brute force attacks (where a machine is used to try and guess your password). If you're not confident in your ability to generate a secure password, use a [password generator tool](http://correcthorsebatterystaple.net/).

```bash
root@Sept-17-2015:~# adduser susanbuck
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

While still signed in as the root user, run this command:

```bash
$ visudo
```

Search for the line that looks like this:

```bash
root    ALL=(ALL:ALL) ALL
```

Below that line, add *new* line that looks like the following (replace `newuser` with your username):

```bash
newuser ALL=(ALL:ALL) ALL
```

Save your changes (`ctrl` + `x`, then `y`, then *Enter*).


## SSH access for your new user

When you first set up DigitalOcean, you created an SSH key connection between your computer and your server. This allowed you to SSH (as root), without having to enter a password.

You'll want the same conveience/security that SSH keys provide for your new user as well, so lets set that up.

Open a *new* Terminal/Cmder window, so we can grab your public key from your local machine, without loosing your connection to your Droplet.

In the new window, on your local computer, get the contents of your `id_rsa.pub` file using the cat command.

```bash
cat ~/.ssh/id_rsa.pub
```

The output will look something like this:

```bash
ssh-rsa [LONG STRING OF RANDOM CHARACTERS] your@email.com
```

Select the entire public key, and copy it.

Switch back to the Terminal/Cmder window that's SSH'd into your Droplet and run the following command to switch to your new user (replace `username` with your username):

```bash
$ su - username
```

Tip: The `whoami` command can be used now (or anytime) to tell you who you're logged in as.

Your new user does not have any SSH settings yet, so let's start by creating a new .ssh directory:

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

Now insert your public key (which should be in your clipboard). Save and exit (`ctrl` + `x`, then `y`, then *Enter*).

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

## Disable root access

Now that you've confirmed you can SSH in as your new user, it's safe to now disable `root` from SSH'ing into the server.

This is done via the config file, `/etc/ssh/sshd_config`.

Typically, to edit this file, we'd use nano, make some changes, save, and be done.

However, if you were to try to do that now, you'd receive the following error when trying to save:

```
Error writing /etc/ssh/sshd_config: Permission denied
```

This doesn't seem to make sense... Didn't we just give our new user administrative privileges in the steps above?

We did! *However*, in order to utilize these privileges, we need to prefix commands with `sudo`.

> sudo (/ˈsuːduː/ or /ˈsuːdoʊ/) is a program for Unix-like computer operating systems that allows users to run programs with the security privileges of another user, by default the superuser. The name is a contraction of "do as su" where "su" is an abbreviation for "super user." -[ref](https://en.wikipedia.org/wiki/Sudo)

So, moving forward, if you ever try a command are told you don't have permissions, prefix the command with `sudo`.

Given that, to open `sshd_config` with admin privileges, run this command:

```
$ sudo nano /etc/ssh/sshd_config
```

In the file, look for the line `PermitRootLogin yes` and toggle `yes` to `no`.

When done, save and exit (`ctrl` + `x`, then `y`, then *Enter*).

To make this change take effect, run the following command to restart your SSH services:

```bash
$ sudo service ssh restart
```

(Note how that command was again prefixed with `sudo` since restarting SSH is something only an administrator can do).

## test
Let's just confirm the above changes worked and the `root` user can no longer SSH into your Droplet.

In a new Terminal/Cmder window, try to SSH in as root:

```
$ ssh root@your-server-ip-address
```

You'll note that it will now continually ask for a password, instead of just letting you in as it previously did with your SSH keys set up. Even if you had the password, it would still not let you in.

Now you might look at this whole procedure and think we're chasing our tails. We prevented the `root` user from logging in because it's dangerous because `root` has administrative privileges.

*But*, we also created a *new* user that also had administrative privileges.

Isn't this just as bad?

It's not, and the reason is because hackers target `root`. They know many, many servers start with a default admin login with the username `root`, and not all server admins will go to the trouble to disable `root`. Because of this, they target their attacks on `root` since it provides them the greatest payoff.

But using a non-default username, you're greatly decreasing the chances a hacker will brute force enter your server.




## Tips

If you want to see a history of authorization attempts made on your server, view the contents of `auth.log`:

```bash
$ sudo cat /var/log/auth.log
```

It's not uncommon to open that file and see lots of root login attempts; those are the hackers trying to get into your account!





Disable the root user
Use an SSH key
Keep your software up to date
Use strong passwords
Keep your firewall settings as strict as possible


## Reference:
+ https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps

+ https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04







---

sudo cat /var/log/auth.log








https://www.digitalocean.com/community/tutorials/how-to-recover-from-a-compromised-droplet-sending-an-outgoing-flood-or-ddos
