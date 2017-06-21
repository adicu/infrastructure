# Getting Started

## Pingy and the Brain

From your laptop, open a terminal session and run `ping adicu.com`. You should
get output that looks like:

```
$ ping adicu.com
64 bytes from 162.243.116.41: icmp_seq=1 ttl=57 time=17.1 ms
^^            ^^^^^^^^^^^^^^                         ^^^^^^^
|             |__ IP Address of server               |__ time to receive packet
|__ Packet Size
```

`CTRL-C` the process to stop (otherwise it'll just keep going). `ping` is a
basic diagnostic tool to make sure you're connected to the network. If it isn't
working, then either you have a bad internet connection or our server is down.
Neither is a good sign.

If you are connected to the internet and someone's given you SSH access, go
ahead and `ssh root@adicu.com` to login.

> NOTE: I ***highly*** recommend learning how to use a terminal multiplexer
> like [tmux](https://github.com/tmux/tmux/wiki) or [GNU
> screen](http://www.gnu.org/software/screen/). It'll make navigating the
> terminal *much* easier.

## Poking the Net

Once you've ssh-ed in, you should see a banner displaying the server's
operating system (Ubuntu 14.04 Linux), various system statistics, and urgent
messages that looks like:

```
Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 3.13.0-32-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

  System information as of Tue Jun 20 19:22:41 EDT 2017

  System load:  0.06               Processes:              112
  Usage of /:   58.2% of 19.56GB   Users logged in:        0
  Memory usage: 64%                IP address for eth0:    162.243.116.41
  Swap usage:   0%                 IP address for docker0: 172.17.42.1

  Graph this data and manage this system at:
    https://landscape.canonical.com/

40 packages can be updated.
32 updates are security updates.

```

Now, we know this server somehow servers our website, but how? Luckily, like
other Unix-like operating systems, Linux comes with a wide variety of tools to
let us poke around.

The first tool we'll use a command called `netstat`, which prints out our
current network connections. If you type `man netstat`, you should see a
plethora of documentation about `netstat` and how to use it. For now, we need
the program information about about listening sockets that speak
[TCP](https://en.wikipedia.org/wiki/Transmission_Control_Protocol), the
protocol underlying the Internet. According to the `man` page, the flags for
this are `-p`, `-t`, and `-l` (short for `--program`, `--tcp`, and
`--listening`), so go ahead and type:

```
$ netstat -tlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address       Foreign Address  State   PID/Program name
tcp        0      0 localhost:smtp      *:*              LISTEN  1245/master     
tcp        0      0 *:https             *:*              LISTEN  1042/nginx      
tcp        0      0 localhost:27017     *:*              LISTEN  5726/mongod     
tcp        0      0 *:http              *:*              LISTEN  1042/nginx      
tcp        0      0 *:x11-1             *:*              LISTEN  4993/python2    
tcp        0      0 *:x11-2             *:*              LISTEN  11122/python    
tcp        0      0 *:8181              *:*              LISTEN  4942/python2    
tcp        0      0 *:ssh               *:*              LISTEN  992/sshd        
tcp6       0      0 ip6-localhost:smtp  [::]:*           LISTEN  1245/master     
tcp6       0      0 [::]:ssh            [::]:*           LISTEN  992/sshd   
```

If local addresses aren't familiar to you, you can also add the `-n` (or
`--numeric`) flag to display:

```
$ netstat -tnlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address       Foreign Address  State   PID/Program name
tcp        0      0 127.0.0.1:25        0.0.0.0:*        LISTEN  1245/master     
tcp        0      0 0.0.0.0:443         0.0.0.0:*        LISTEN  1042/nginx      
tcp        0      0 127.0.0.1:27017     0.0.0.0:*        LISTEN  5726/mongod     
tcp        0      0 0.0.0.0:80          0.0.0.0:*        LISTEN  1042/nginx      
tcp        0      0 0.0.0.0:6001        0.0.0.0:*        LISTEN  4993/python2    
tcp        0      0 0.0.0.0:6002        0.0.0.0:*        LISTEN  11122/python    
tcp        0      0 0.0.0.0:8181        0.0.0.0:*        LISTEN  4942/python2    
tcp        0      0 0.0.0.0:22          0.0.0.0:*        LISTEN  992/sshd        
tcp6       0      0 ::1:25              :::*             LISTEN  1245/master     
tcp6       0      0 :::22               :::*             LISTEN  992/sshd 
```

This tells us that all `http` and `https` connections go through a program
called `nginx`.

# Nginx

Nginx (pronounced "Engine X") is a high-performance web-server. Now, according
to the [nginx Beginner's
Guide](http://nginx.org/en/docs/beginners_guide.html#control), 

> By default, the configuration file is named `nginx.conf` and placed in the
> directory `/usr/local/nginx/conf`, `/etc/nginx`, or `/usr/local/etc/nginx`. 

On our server, nginx configuration is located in the `/etc/nginx/` directory.
If you go ahead and open the main `nginx.conf` file, you should see a file
that begins with

```nginx
user www-data;
worker_processes 4;
pid /run/nginx.pid;

events {
        worker_connections 768;
        # multi_accept on;
}
```

Unfortunately, nginx configuration files are written in their own special DSL
instead of a standard like `toml` or `json`. Still, the basic gist of the
language should be pretty clear.


If we look at the `http` config block, we'll see a line that says:

```nginx
        include /etc/nginx/sites-enabled/*;
```

That tells us that there are more configuration files tucked away at
`/etc/nginx/sites-enabled/`. Let's go to the directory and look inside:

```bash
$ cd /etc/nginx/sites-enabled
$ ls
adi-website  density  devfest  learn
```

> NOTE: Usually, everything in `/etc/nginx/sites-enabled` is actually just a
> symlink to a corresponding file in `/etc/nginx/sites-available` -- that way,
> you can easily enable and disable websites just by `rm`-ing things in
> `sites-enabled`. For our purposes, however, this shouldn't matter.

**Now** we're getting somewhere. Go ahead and open the `adi-website` file. You
should see a file starting with:

```nginx
server { 
        listen 443 ssl;
        server_name adicu.com;
        root /srv/adi-website/www;

        ssl_certificate /etc/letsencrypt/live/adicu.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/adicu.com/privkey.pem;

        # ssl_certificate /srv/adi-website/ssl/unified.crt;
        # ssl_certificate_key /srv/adi-website/ssl/adicu.com.key;

        access_log /srv/adi-website/logs/access.log;
        error_log /srv/adi-website/logs/error.log;

        location / {
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;
                proxy_redirect off;
                proxy_pass http://127.0.0.1:8181;
        }
}
```

There's a lot of information packed in there, but the important bit for us is
the:

```
                proxy_pass http://127.0.0.1:8181;
```

line. This tells us that all the traffic is actually "proxying" (or going
through) `127.0.0.1:8181`! Going back through our `netstat -tnlp`, we see that
this is a `python2` process.

## Docker, Docker

At this point, I'm going to cheat and just tell you that this Python process
actually lives in a Docker container. Unfortunately, because of how Docker
interacts with the rest of Linux, it's not obvious to me how to figure this
out from first principles (for example, you can check `/proc/PID/cgroup`, but
cgroups are pretty much only used for containers AFAIK).

If you type `docker` into the terminal, you should get a huge list of possible
subcommands. For now, the most interesting one is

```
$ docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED        STATUS       NAMES
aeaff3e00af1   density       "/usr/bin/tini -- ..."   2 weeks ago    Up 2 weeks   density
5b0fd1ec032c   devfest       "/bin/sh -c 'gunic..."   2 months ago   Up 6 weeks   devfest
c638381b01a3   adi-website   "/bin/sh -c 'sourc..."   2 months ago   Up 6 weeks   adi-website
```

`docker ps` displays all the running docker containers (similar to `ps`, which
displays all running processes). You should see three docker containers: one
for the ADI website, one for Density, and one for the 2015 Devfest website.

Another useful Docker subcommand is `docker logs`, which as the name and `man`
page implies, lets you get the logs of a container:

```
docker logs adi-website
```

A handy trick for debugging is to use `docker exec`, which lets you run
arbitrary programs inside the Docker container. In particular, you can open up
a shell inside the container so you can poke around yourself with:

```
docker exec -it adi-website /bin/sh
```

(`-i` stands for `--interactive` and `-t` stands for `--tty=true`).


## Deployment

So now we know that the ADI website is actually running in a docker container
-- but what created that Docker container?

If you go to the `/root` directory, you should notice a bunch of
`adi-website.git`. This is a *bare* git repository which consists of only
git-internal data (in a normal git repository, all of this information is
hidden in the `.git` directory).

Inside, you should find a `hooks` directory:

```
$ cd /root/adi-website.git/hooks
$ ls
applypatch-msg.sample  pre-applypatch.sample      pre-rebase.sample
commit-msg.sample      pre-commit.sample          update.sample
post-receive           prepare-commit-msg.sample
post-update.sample     pre-push.sample
```

[Git hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) are a
neat way to automatically run programs when git takes certain actions. In our
case, the most interesting hook is the `post-receive` hook, which runs
whenever someone pushes to this git directory. If you open the `post-receive`
hook, you should see something like:

```bash
#! /bin/bash

ADI_WWW=/srv/adi-website/www

echo   "=================================="
echo   "Deployment Starting"
echo   "----------------------------------"
echo   "Progress:"
printf "Deploying adicu.com          "

GIT_WORK_TREE=$ADI_WWW git checkout -f
if [[ -z "$(ps -edaf | grep "^mongo")" ]]; then
    mongod &  # only start mongo if it's not already running
fi
cd "$ADI_WWW"

./deploy.sh
printf "Done\n"

echo   "----------------------------------"
echo   "Status:"
printf "adicu.com                "
sleep 2
if [[ -z "$(curl -Is https://adicu.com | head -1 | grep 200)" ]]; then
    printf "Fail\n"
else
    printf "Live\n"
fi

exit
```

So it looks like we load the most recent `git` commit into `$ADI_WWW` (or
`/srv/adi-website/www`) and then run `/srv/adi-website/www/deploy.sh`. Thus,
`/srv/adi-website/www` should contain the currently running version of
`adi-website`'s code along with a `deploy.sh` script. Go ahead and make sure
that's true.

## Summary

In short:

- We use `nginx` to handle our subdomains, which we proxy to the right apps
- Our apps run in Docker containers and can be debugged with standard Docker
  commands.
- Our apps are deployed via git hooks whenever we `git push` to the right
  repositories.
- The currently deployed code lives in `/srv/APP_NAME/www`.
