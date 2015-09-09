## Domain setup

Right now you can access your DigitalOcean droplet by typing its IP address into your browser. This works but is hard to remember, and it'd also be nice to run several web applications on your one Droplet, each with a unique subdomain.

To set this up, we're going to configure a domain and subdomains for use with DigitalOcean.

If you have an existing domain you'd like to use, that's fine. If not, you can create a new domain name via a service like **[Namecheap](http://www.namecheap.com/?aff=61057)**. As of this writing, Namecheap is offering free domain names for students (with a `.edu` email address) via <https://nc.me/>.

After your create your domain, find your **DNS settings** within your domain companies' control panel. In Namecheap, these settings are found under **All Host Records**

In your DNS settings, you'll set both your `@` and `www` hostname to your DigitalOcean IP address.

Also, while you're there, add a subdomain called `helloworld` which also points to the same IP.

<img src='http://making-the-internet.s3.amazonaws.com/version-control-namecheap-dns@2x.png' style='max-width:1161px; width:100%' alt='Namecheap DNS'>

Give the above settings a few minutes to take effect, then test out your domain. You should see the same results you saw above when you tested your IP address, but this time it's loaded via your domain name:

<img src='http://making-the-internet.s3.amazonaws.com/vc-namecheap-domain-first-working@2x.png' style='max-width:958px; width:100%' alt='First domain working'>

If you don't yet see the above, try the following:

1. Clear your browser cache.
2. Do a [DNS Cache Flush](http://docs.cpanel.net/twiki/bin/view/AllDocumentation/ClearingBrowserCache).

Try again. Still not loading?

3. Try a different browser.
4. Try accessing your URL via [this proxy](http://www.megaproxy.com/freesurf/). Does it load there? If it loads there, it just means your computer is still caching old settings.




## Virtual Host / Subdomain setup

Your primary domain is working, now lets get a subdomain working (`http://helloworld.domain.com`).

In the above step, you already set up your DNS for the subdomain, making it so that traffic hitting `http://helloworld.domain.com` will point to your IP address.

Now we need to give the server instructions on what to do with this traffic.

This is done via a Virtual Host configuration; **Virtual Hosts allow Apache (your web server software) to be configured for multiple sites that have different configurations**.

In DigitalOcean, Virtual Hosts can be specified in the following file: `/etc/apache2/sites-enabled/000-default.conf`.

Given that, navigate to `/etc/apache2/sites-enabled/` and open `000-default.conf` for editing using `nano.`

(If need a refresher on editing files via nano, [go here](https://github.com/susanBuck/dwa15-fall2015-notes/blob/master/00_Command_Line/06_Nano.md).)

At the *bottom* of `000-default.conf`, add this VirtualHost block:

```
<VirtualHost *:80>
  ServerName helloworld.dwa15-practice.biz
  DocumentRoot "/var/www/html/hello-world"
  <Directory "/var/www/html/hello-world">
    AllowOverride All
  </Directory>
</VirtualHost>
```

Be sure to change the `ServerName` value to match *your* subdomain. In this example, it's set to `helloworld.dwa15-practice.biz` but you'll need to change it to match your domain.

Also, if necessary, adjust the `DocumentRoot` and `Directory` to point to the location of your `hello-world` directory.

In plain english, the above configuration says: *when traffic comes in via `helloworld.dwa15-practice.biz`, point to the `/var/www/html/hello-world` directory*.

When you're done, save your changes to `000-defualt.conf`.

(If for some reason you make a mistake in `000-default.conf`, [here's a copy of the original](https://gist.github.com/susanBuck/790ea5a0d1ad7d02e586).)

To make your VirtualHost changes take effect, in DigitalOcean while looking at your Droplet, go to the *Power* section and find the button to **Power Cycle**

<img src='http://making-the-internet.s3.amazonaws.com/vc-do-power-cycle@2x.png' class='' style='max-width:968px; width:100%' alt='Digital Ocean Power Cycle'>

Once the restart is complete, test out your subdomain `helloworld.yourdomain.com`.

<img src='http://making-the-internet.s3.amazonaws.com/version-control-subdomain-good@2x.png' class='' style='max-width:588px; width:75%' alt=''>

Keep these notes handy for when you set up projects, because each project should have it's own subdomain:

+ `http://p1.yourdomain.com`
+ `http://p2.yourdomain.com`
+ `http://p3.yourdomain.com`
+ `http://p4.yourdomain.com`




## Tips / Notes

Find out the IP address of a domain:

```bash
$ ping domain.com
```

Find out the nameserver of a domain:

```bash
$ dig +short NS domain.com
```
