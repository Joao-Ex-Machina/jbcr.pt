+++
title = 'Self-Hosting part I: An Unexpected Journey to my own Website'
date = 2024-11-04T11:21:23Z
tags = ['tech', 'selfhosting','website']
+++

This post details the story of this website. Why and how I created it, who inspired me to it and how I am managing it.

If you want to pick some bits of my source code you can find it [here](https://github.com/Joao-Ex-Machina/jbcr.pt) so that you can hack together your own, **yes your very own**,  website.


This post is of course more than inspired on José Lopes' [post on self-hosting](https://joselopes.dev/self-hosting), go check it out!


# Why?

Well if you know me you know I am not really a software type of guy, I prefer to dabble in hardware design and electronics. Therefore I rarely entertained the idea of having me own webpage in last 4-5 years.

However everything changed this summer, when my good friend (and gym partner) José Duarte Lopes showed me his [website](https://joselopes.dev)

He (an ECE laddie like me) had made what no Computer Science guy could have ever done...He peaked my interest.

How delightful it was, a **clean and slick website**, with almost no JavaScript, while running on a **jerry-rigged smartphone/server**. It was no short of a feat of wizardry. That very same day I acquired the [jbcr.pt](https://jbcr.pt) domain from [amen.pt](https://amen.pt), for the low-low price of **FREE**!

And so like Gandalf with Bilbo, he had convinced me to go onto an **unexpected journey** ... Some months later of course, since the idea needed time to ferment - like a nicely brewed golden mead, but instead of honey, yeast and water my ingredients would be hugo, my router and a Thinkpad T410. 

# An Infallible Plan with Infallible Guidelines

I then devised a set of guidelines for this project:

- Every piece of software I used had to be **free and open source**, excluding GitHub
- The server setup should be easily **extensible**, and eventually **able to self-host more services**
- Editing the website should be as easy as editing a text file
    - I would **not use any JS** for my website, since I dislike it (possibly for no apparent reason)
- Ideally **the cost** to maintain the site **should be minimal**, no more than 5 coffees/month
- Finally the most important clause, arguably: **It had to be at least as good as José's**

# The Setup

![Tux Server?](/tux-server.jpg)

### Hugo

(Possibly I started from the opposite direction from where I should've started)

**Hugo** seemed a good starting point for my framework. It takes some .toml and .md files, applies some forbidden golang magic and bada-bing bada-boom I have a ton of htmls **ready to deploy with minimal effort**.

I started working on my website while on the bus back from [Festa do Software Livre](festa2024.softwarelivre.eu/), experimenting with some templates, none of which I liked, until I found the Holy Grail - [lugo](https://github.com/LukeSmithxyz/lugo) by Luke Smith. The template seemed fairly modular, it had some videos that explained it quite well, and targeted what I wanted - **a clean, extensible and easy to manage webpage with no JS**.

Sure I had to learn a bit of HTML, and I required one or another tip but other than that was point and shoot! We were already starting good!

I've been continously tweaking the style to fit my specific enjoyment for the **late 90's and early 00's web**, therefore I can always argue that my website is how the "internet should've been" and how it is single-handledly on a quest to "desmerdificar" (lit. unshittify) the internet.

### Hardware

Like José I pondered to use an old smartphone. Mine would have been a Huawei P Smart 2020, with a smashed screen. But then I remembered my stashed **Thinkpad T410** that was found on a **garden bench**.


<center>

{{<img caption="My Thinkpad T410 server after the full setup, running Wireshark " src=/t410-server.jpeg >}}

</center>


I already had a spare SSD and I had bought some broken keyboards to fix the missing keys (**Thank you Margarida Gralha** for picking them up). I assembled it, flashed Ubuntu into that machine, named it Erebor and called it a day.

Our hardware was ready to roll!

### Nginx

To setup nginx you have this file in [/etc/nginx/nginx.conf](). For now your config should kinda look like this:

```nginx
http {
  (...)
    server {
        listen  $PORT default_server; 
        #use whatever port you want.
        #80 is the default for HTTP
        server_name  www.yourdomain.pt ;
        location / {
            root  /path/to/website/root;
            index  index.html index.htm; 
            #nginx will look for these files on the root
        }
        (...)
    }
}

```
Do not forget to alter the file in:

[/etc/nginx/sites-enabled/default]()

**if you are serving on port 80, you should remove the default_server property from it.**

In the server I opened the desired port with:

```ps
> sudo ufw allow $PORT
```

Then you can enable and start nginx:

```ps
> sudo systemctl enable --now nginx
```

You can test it with another machine inside your LAN by acessing [local-server-ip:$PORT](192.168.1.254:80)

If you are using the port 80 you can drop [$PORT](:80), as it is the [HTTP](HTTP) default

### Router configuration

In your router configuration you simply have to **open a router port** and **redirect all traffic to your server port**.

Normally this should not be much of a hassle and must routers already have defaults defined for certain protocols like HTTP (port 80), HTTPS (port 443) and SSH (port 22).

Now you can test it by acessing [public-router-ip:$PORT](https:176.78.59.62:443)

### DynIP

Most routers **do not have a static IP**. Normally you have to pay for this feature.

In Portugal MEO provides a **Dynamic IP solution** for free for their clients, which basically wraps a ugly domain URL around your router IP dinamically. I used it just in case.

### Redirecting to my domain

This should be a hassle-free step (again). Just go into your Registrar and redirect your domain to your IP/DynIP. You may need to fix some of the DNS configs, but it will depend on the registrar, CloudFlare and NameCheap appear to be the most friendly.

You can now test it by acessing [yourdomain.pt](https://jbcr.pt) Hooray!

### SSL Certification

As my registrar does not provide free SSL Certification, and I really wanted/needed to be certified (I was in Firefox and some links on the site were getting blocked due to the lack of a certicate) I had to find another solution.

It appear in the way of [EFF's Lets Encrypt](https://letsencrypt.org/getting-started/). 

I just downloaded the [certbot](https://certbot.eff.org/) script from snap and ran on the server:

```ps
> sudo certbot --nginx

```

It'll ask you for some basic stuff in the CLI such as an e-mail and the target domain.

After it is finished your [/etc/nginx/nginx.conf]() will have the **ssl_certificate**, **ssl_certificate_key** and a **listen 443** and some other similiar parameters added to it. Like this:

```nginx
http {
    (...)
    server {
        (...)
        listen [::]:443 ssl ipv6only=on;
        # managed by Certbot
        listen 443 ssl; # managed by Certbot
        ssl_certificate /path/to/fullchain.pem
        ssl_certificate_key /path/to/privkey.pem;
        # managed by Certbot
        include /etc/letsencrypt/options-ssl-nginx.conf;
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
        # managed by Certbot

    }

	server {
        if ($host = jbcr.pt) {
            return 301 https://$host$request_uri;
        } # managed by Certbot


		listen 80 default_server;
		listen [::]:80;
		server_name  jbcr.pt;
        return 404; # managed by Certbot
    }
}
```


If your website is correctly serving on the domain there should be **no fish bones**!

# The End (?)

This is not the end, **for sure**. Soon I will change from ADSL to Fibre, which will deprecate my router config. I will also be continously working on this website, adding new content and making the framework better, I promise!

Overall it has been a rewarding experience. It almost feels like I am not procrastinating, since I am building something that is actually useful!


**PS:** Thank you Eduardo and José for the suggestions and peer-review of this post!
