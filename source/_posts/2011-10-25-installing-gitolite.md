---
layout: post
title:  Installing Gitolite
categories:
    - blog
---
Recently a few of us at [phpfreaks][PHPFreaks] decided revamp some projects relating to the site. We had previously been using [subversion][Subversion] with any development work, but Iv'e been using [git][Git] now for the last few years, and most of the guys agreed it was time to move on. We needed a central repo though, and while [github][GitHub] is awesome for open source stuff, we need to keep most of this stuff private. [gitolite][Gitolite] is ideal for this sort of setup, and it's a pretty straight forward install / configure.

I'm just going to jot this down for posterity.

Before I start, I'll just point out  a little tip. We run [SSH][openssh] on some random port. To make accessing the remote server easier, add an entry to your local ~/.ssh/config file:
<pre>
    Host remote
    HostName some-remote-server.com
    Port 1234
</pre>

This means we can now also use the nice shortcut _remote_ to access the machine.

<strong>Installation:</strong>

We run [debian][Debian]on our dev servers so installation is simple.

    $ sudo apt-get update && sudo apt-get install gitolite

Back on my local machine, I need to copy my public ssh key to the server. It's best to place this in the /tmp directory so that the gitolite user can access this easily. You'll also want to name the remote file the same as the user you are planning on accessing the server as.

    $ scp ~/.ssh/trq.pub remote:/tmp/trq.pub</pre>


[OpenSSH][openssh] back into the remote server and make sure the public key we just uploaded is readable.

Now we need to su to the [gitolite][Gitolite] user and import our initial users key into the configuration:

    $ sudo su gitolite
    $ gl-setup /tmp/trq.pub

That's it, were up and running. Go back to your local machine to configure your repositories.

<strong>Configure Your Repositories:</strong>

That's right! One of the great things about [gitolite][Gitolite] is the fact that it's configuration is controlled remotely via a [git][Git] repository. Check it out and take a look:

    $ git clone gitolite@remote:gitolite-admin
    $ cd gitolite-admin
    $ tree
    .
    ├── conf
    │   └── gitolite.conf
    └── keydir
        ├── trq.pub

The entire repo contains a configuration file in conf/gitolite.conf and my public ssh key (added by (gl-setup) in the _keydir_ directory. To add new users, simply place there key in the _keydir_ then edit the configuration file as needed.

I'm not going to go into to many of the nitty gritty details, but I'll describe how we wanted our repos configured, then just show you the config file. It should be pretty self explanatory.

I basically wanted lead devs to have read/write access to the master branch, all other devs to have read only access and to allow all devs to be able to share branches by allowing them to push to a namespace on the remote server. A basic config looks like this:

    $ cat conf/gitolite.conf
    @admins             = trq
    @leads              = trq
    @devs               = somedev someotherdev

    @site-repos         = site forums
    @server-repos       = gitolite-admin configs

    repo                @server-repos
        RW+             = @admins

    repo                @site-repos
        RW+             = @leads
        RW+             devs/USER/          = @devs @leads

This setup allows all devs access to read the master branch of two repositories, _site_ and _forums_. They can also write to branches within these repositories starting with devs/username/somebranch. So basically they get there own namespace within devs/username, they can also read each others branches from these locations which makes for easy sharing.

On top of that, people in the @admins group (me), can read and write to two extra repositories; gitolite-admin (the [gitolite][Gitolite] configuration repository) and configs (various server configs we have under version control).

Hopefully this will give you an idea of the flexibility of [gitolite][Gitolite]. I might post another entry describing some of the [git][Git] workflow we use with this setup.

[phpfreaks]:        http://phpfreaks.com
[subversion]:       http://subversion.tigris.org
[git]:              http://git-scm.com
[github]:           http://github.com
[gitolite]:         https://github.com/sitaramc/gitolite
[openssh]:          http://www.openssh.com/
[debian]:           http://debian.org
