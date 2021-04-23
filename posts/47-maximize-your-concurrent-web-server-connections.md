---
Title: Maximize your concurrent web server connections
Date: 2021-04-23
Image: https://wakatime.com/static/img/covers/rocket.jpg
Description: A guide to increasing your max open file descriptors (ulimit -n) on Linux machines.
Author: Alan Hamlett
AuthorUrl: https://wakatime.com/@alan
AuthorGravatar: https://wakatime.com/gravatar/@alan
Category: Engineering
Tags: haproxy, nginx, devops
---

<img src="https://wakatime.com/static/img/covers/rocket.jpg" alt="rocket" style="width:100%" />

For production web servers, most people assume scaling means needing faster (and more expensive) hardware.
Before spending more money on servers, first make sure your web server process is using the maximum available connections supported by your Linux kernel.

### Max open file descriptors (Kernel file-max)

Each Linux kernel is compiled to allow a certain maximum open files (or sockets) per process.
From now on, we’ll call that `file-max`.
[WakaTime][wakatime] uses [DigitalOcean][digitalocean] servers, and depending on the instance size each comes with a different file-max.
To check the kernel’s file-max, run `cat /proc/sys/fs/file-max`.

For example, the $320/mo DigitalOcean instances have a file-max of `6,579,127`.

    $ cat /proc/sys/fs/file-max
    6579127

<img src="https://wakatime.com/static/img/blog/digitalocean-medium-droplet.png" class="img-thumbnail" alt="DigitalOcean medium droplet size" style="width:90%" />

While the $5/mo DigitalOcean instances only have a file-max of `94,198`:

    $ cat /proc/sys/fs/file-max
    94198

<img src="https://wakatime.com/static/img/blog/digitalocean-small-droplet.png" class="img-thumbnail" alt="DigitalOcean small droplet size" style="width:90%" />

That’s 94k concurrent connections per process, which is still plenty enough for most production web servers.
If you’re using Node.js that’s your max, but if your web server runs multiple worker processes then multiply 94k by the number of processes to get your max connections.
However, to actually use those 94k concurrent connections your web server process needs it’s ulimit increased.

### Ulimit (Process file-max)

Your Linux kernel has a file-max of 94k, but let’s check your web server’s process file-max:

    $ ps -aux | grep -m 1 nginx
    nginx    12785  0.1  0.1  62508 24372   nginx: worker process
    $ cat /proc/12785/limits | grep "open files"
    1024

Or in a one-liner:

    $ cat /proc/`ps -aux | grep -m 1 nginx | awk -F ' ' '{print $2}'`/limits | grep "open files" | awk -F ' ' '{print $4}'
    1024

What’s that?
Nginx can only handle 1k concurrent connections when the kernel supports 94k?

That’s because we need to increase the file-max setting for the `nginx` user.
To do that, add these lines to your `/etc/security/limits.conf` file:

```
* soft nofile 94000
* hard nofile 94000
```

The `*` is for all users except the `root` user, and you can specify a username like `nginx soft nofile 94000`.
Now your Nginx user has permission to open 94k connections, but it still won’t use those 94k connections.
That’s because you also need to edit your systemd unit file for Nginx.

### Systemd file-max (LimitNOFILE)

When running Nginx with [systemd][systemd], you’ll notice the Nginx processes aren’t showing the 94k file-max limit available.
To fix that, add `LimitNOFILE=94000` to your `/etc/systemd/system/nginx.service` file under the `[Service]` block:

```
[Unit]
Description=A high performance web server and a reverse proxy server
After=network.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t -q -g 'daemon on; master_process on;'
ExecStart=/usr/sbin/nginx -g 'daemon on; master_process on;'
ExecReload=/usr/sbin/nginx -g 'daemon on; master_process on;' -s reload
ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx.pid
TimeoutStopSec=5
KillMode=mixed
LimitNOFILE=94000

[Install]
WantedBy=multi-user.target
```

Run `systemctl daemon-reload` to apply the changes, and now your Nginx processes show 94k when you check their file-max limit:

    $ cat /proc/`ps -aux | grep -m 1 nginx | awk -F ' ' '{print $2}'`/limits | grep "open files" | awk -F ' ' '{print $4}'
    94000

### Haproxy

Other software, such as [haproxy][haproxy], set their own file-max ulimit.
If you’ve increased the above limits and your haproxy child process still isn’t showing 94k then try adding `maxconn 94000` to your `haproxy.cfg` file.
We also switched from init.d to systemd for managing haproxy, since setting `LimitNOFILE` is super easy with systemd:

```
[Unit]
Description=HAProxy Service

[Service]
ExecStart=/usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg
ExecReload=/bin/kill -USR2 $MAINPID
Restart=always
KillSignal=SIGTERM
TimeoutStopSec=5
LimitNOFILE=94000

[Install]
WantedBy=multi-user.target
```

### Postgres

This isn’t just for web servers.
Databases like Postgres use sockets too, when your web servers connect to your database.
By default, Postgres uses 1k file-max per process.
Sometimes that's over your default ulimit, and you’ll start seeing the `Too many Open files` error in your Postgres logs.
Instead of lowering the default Postgres `max_files_per_process` config, just edit your `/etc/security/limits.conf` file adding:

```
postgres soft nofile 1000
postgres hard nofile 1000
```

Then restart Postgres and check its file-max:

    $ cat /proc/`ps -aux | grep -m 1 postgres | awk -F ' ' '{print $2}'`/limits | grep "open files" | awk -F ' ' '{print $4}'
    1000

### Sysctl

It’s also worth tweaking your kernel’s networking settings with sysctl, such as your max backlogged connections and tcp buffers.
Check out Peter Mescalchin’s [collection of sysctl notes][sysctl notes] and the [sysctl docs][sysctl docs].
Use `sysctl -a` to list all supported configs on your machine, and see their default values.
Add any changes to your `/etc/sysctl.conf` file then run `sysctl -p` to apply them without needing a reboot.

### Conclusion

Increasing your file-max limit unlocks the full potential of your web server.
If you liked this post, also check out our previous post about a [disk-backed Redis compatible server called SSDB][blog post 45].
It removes the main limitation of Redis: your data set having to fit in available RAM.


[wakatime]: https://wakatime.com
[digitalocean]: https://www.digitalocean.com/
[systemd]: https://www.freedesktop.org/software/systemd/man/systemd.service.html
[systemd nofile]: https://www.freedesktop.org/software/systemd/man/systemd.exec.html#LimitCPU=:~:text=LimitNOFILE%3D,-ulimit
[haproxy]: https://cbonte.github.io/haproxy-dconv/2.0/configuration.html#maxconn
[sysctl notes]: https://bl.ocks.org/magnetikonline/2760f98f6bf654d5ad79
[sysctl docs]: https://www.kernel.org/doc/Documentation/sysctl/net.txt
[blog post 45]: https://wakatime.com/blog/45-using-a-disk-based-redis-clone-to-reduce-aws-s3-bill
