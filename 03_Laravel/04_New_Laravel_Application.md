## Preflight check

The Laravel framework has these system requirements:

+ PHP >= 5.5.9
+ OpenSSL PHP Extension
+ PDO PHP Extension
+ Mbstring PHP Extension
+ Tokenizer PHP Extension

If you've followed our lead and set your PHP executable to be the same as the PHP that ships with MAMP/XAMPP, your local server should meet all these requirements.




## Create a new Laravel applications

There are a couple different ways to spawn a new Laravel application but we'll be using Composer.

Change directories into your local Document Root where your new app will live.

Mac:
```bash
$ cd /Applications/MAMP/htdocs
```

Windows:
```bash
$ cd c:\xampp\htdocs
```

Next, use this `composer` command to generate a new Laravel project. In this command, the name of our project is `foobooks` which the name we're giving an example application we'll be building during lectures.

```bash
$ composer create-project laravel/laravel foobooks --prefer-dist
```

When Composer is done working, move into the newly created project directory:

```
$ cd foobooks
```

Once in this directory, run the `ls -la` command; you should see all the Laravel related files (including hidden ones).

<img src='http://making-the-internet.s3.amazonaws.com/laravel-fresh-install@2x.png' style='max-width:668px; width:100%' alt='Laravel fresh install'>




## Permissions

At this point in the installation procedure, the Laravel documentation (and most Laravel tutorials) would have you run the following two commands:

```bash
$ chmod -R 777 storage
$ chmod -R 777 bootstrap/cache
```

Before you run those commands, though, read on:

Laravel/your Apache web server needs *write* access for the `storage` and `bootstrap/cache` directories. This is because when you run your Laravel application it will write to these directories. For example, errors will be written to `storage/logs/laravel.log`, and cached configurations to speed up your app will be written to `bootstrap/cache`.

But for __Windows/XAMPP users__, you shouldn't need to adjust permissions because Apache already has all the permissions it needs because it runs under the LocalSystem account on Windows, which has extensive read/write access to local paths.

Likewise, most __Mac/MAMP users__ shouldn't need to run those commands because MAMP Apache is running as the user who launched it, i.e. you, and as owner of all the files and directories in your project, you already have write access.

The only instance where this might not work is if you're using a different local server setup that does *not* have the correct permissions. If that were the cause, you'd run the above `chmod` commands to grant the appropriate `write` access.

__Long story short:__ You can skip those commands for now. Down below when you try to first load the app, if you run into permission issues, you can come back here.

All this being said, when you get to the deploying your Laravel app to a live web server, you will definitely need those commands. We'll cover that when you get to the *Deploy to DigitalOcean* notes though, so you don't have to worry about that right now.

(If you want to learn more about the permissions used in the commands, [go here](https://github.com/susanBuck/dwa15-fall2015-notes/blob/master/00_Command_Line/07_Permissions.md)).




## Namespacing

Namespacing is a programming techniques that helps you avoid name conflicts between software programs that
are used in combination with one another.

Laravel5 uses namespacing, so one of the initial configurations you can make is to set a specific `name` for your application.

For example, for our &ldquo;Foobooks&rdquo; app, we may want to use the name `Foobooks`. This would be accomplished with this command:

```bash
$ php artisan app:name Foobooks
```

If you don't explicitly set a name for your application, it will default to `App`.

Choosing a name other than `App` for your application is suggested, but it's optional.

In the interest of making our examples more generic, we're *not* going to rename the Foobooks namespace and leave it as the default `App`.

We'll discuss namespacing more later, but in the mean time, you can learn more here: [CodeBright: Primer: Namespacing](http://daylerees.com/codebright-the-primers/).


## Point local server to your new app

The bare bones initial setup for your app is complete, and you should be ready to view it in the browser.

First, though, you'll need to point your localhost's Document Root to the `public/` directory within your new app.

The paths will look something like this (adjust accordingly to match your system):

Mac:
```
/applications/MAMP/htdocs/foobooks/public
```

Windows:
```
c:\xampp\htdocs\foobooks\public
```

In MAMP you can change the Document Root via the Apache settings:

<img src='http://making-the-internet.s3.amazonaws.com/laravel-app-setup-document-root@2x.png' class='' style='max-width:300px; width:100%'>

In XAMP you'll change the Document Root via Apache's config file, `httpd.conf`:

<img src='http://making-the-internet.s3.amazonaws.com/laravel-xamp-document-root@2x.png' class='' style='max-width:1087px; width:100%' alt=''>

__Save all changes and restart your server.__

If all went well, you should see Laravel's default welcome page when you hit `http://localhost` in your browser:

<img src='http://making-the-internet.s3.amazonaws.com/laravel-app-setup-success@2x.png' style='max-width:300px; width:100%'>

Whenever configuring a server to run Laravel, always remember **the Document Root has to point to the `public/` folder within your app directory**.




## Version Control your new app

In your project, initiate a new Git repository:

```bash
$ git init
```

### Github

Create a new repository at Github.com. When doing this, do *not* initialize the repository with a `README.md` file because you'll be working with a repository that has already been initialized.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-foobook-repo@2x.png' class='' style='max-width:735px; width:100%' alt='Foobooks repo setup in Github'>

Note the Git SSH URL, for example, `git@github.com:username/foobooks`

In your project directory, add a new remote origin called `origin`, for example:

```bash
$ git remote add origin git@github.com:username/foobooks.git
```

### First commit
Run git status to see all your untracked files:

```bash
$ git status
```

Add all your files for committing:

```bash
$ git add --all
```

Commit these changes:

```bash
$ git commit -m "First commit"
```

Push your project up to Github:

```bash
$ git push origin master
```

When you visit your repository on Github you should see all your changes there.

Your app is now set up locally and ready for development. In the next section, we'll cover the procedure for deploying your app to a live server.




## Tips

* When using the `composer create-project` command, we added the `--prefer-dist ` flag. You can read more about `--prefer-dist` and how it differs from `--prefer-source` [here](https://getcomposer.org/doc/03-cli.md#install).
