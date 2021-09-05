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

![rocket](https://wakatime.com/static/img/covers/rocket.jpg)
*Image Credit: [Bill Jelen](https://unsplash.com/photos/NVWyN8GamCk)*

For production web servers, most people assume scaling means needing faster (and more expensive) hardware.
Before spending more money on servers, first make sure your web server process is using the maximum available connections supported by your Linux kernel.
There are 3 layers you need to check and configure:

1. Kernel file-max
2. Process file-max (ulimit)
3. Systemd/Server config file-max (LimitNOFILE)

Any one of these layers can limit the number of max connections your web server process is allowed.

This becomes really useful when your web server isn’t bottlenecked by RAM or CPU.
For ex, when using [HAProxy to load balance Nginx web servers][load balancing blog post].
HAProxy isn’t processing the request and uses almost no CPU, but it needs to handle a large amount of concurrent connections.
Increasing the file-max is also beneficial for production Nginx, Apache, and even your Redis and Postgres database servers.

### Max open file descriptors (Kernel file-max)

Each Linux kernel supports a certain maximum number of open files (or sockets) per process.
From now on, we’ll call that `file-max`.
To check the kernel’s current file-max, run:

    cat /proc/sys/fs/file-max

For example, the $5/mo DigitalOcean instances have a file-max of 9 Trillion:

    $ cat /proc/sys/fs/file-max
    9223372036854775807

<img src="https://wakatime.com/static/img/blog/digitalocean-small-droplet.png" class="img-thumbnail" alt="DigitalOcean small droplet size" style="width:90%" />

You won’t ever be able to use those 9T concurrent connections.
That’s because each open connection [uses around 1k RAM][file-max ram], and DigitalOcean doesn’t offer any instance with that much RAM.

If your default file-max is too low, increase it by adding this line to your `/etc/sysctl.conf` file:

    fs.file-max = 1000000

Load the new file-max by running `sysctl -p` as root.
Now your kernel’s file max should show 1M concurrent connections available:

    $ cat /proc/sys/fs/file-max
    1000000

However, check the file-max of your current process and you see it’s much lower than the kernel file-max:

    $ ulimit -Sn
    1024

If you’re using Node.js that’s a max of 1k concurrent connections, but if your web server runs multiple worker processes then multiply 1k by the number of processes to get your max connections.
Either way, that’s not many concurrent connections for a production web server.
To actually use those 1M concurrent connections, your web server process needs its ulimit increased.

### Ulimit (Process file-max)

Your Linux kernel has a file-max of 1M, but let’s check your nginx web server’s process file-max:

    $ cat /proc/`ps -aux | grep -m 1 nginx | awk -F ' ' '{print $2}'`/limits | grep "open files" | awk -F ' ' '{print $4}'
    1024

What’s that?
Nginx can only handle 1k concurrent connections when the kernel supports 1M?

That’s because we need to increase the file-max setting for the `nginx` user.
To do that, add these lines to your `/etc/security/limits.conf` file:

```
* soft nofile 1000000
* hard nofile 1000000
```

The `*` is for all users except the `root` user, and you can specify a username like `nginx soft nofile 1000000`.
Now your Nginx user has permission to open 1 Million connections, but your Nginx process still won’t use those 1M connections.
That’s because you also need to edit your systemd unit file for Nginx.

### Systemd file-max (LimitNOFILE)

When running Nginx with [systemd][systemd], you’ll notice the Nginx processes aren’t showing the 1M file-max limit available.
That’s because a process can set it’s own ulimit, and Systemd defaults to a low limit.
To fix that, add `LimitNOFILE=1000000` to your `/etc/systemd/system/nginx.service` file under the `[Service]` block:

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
LimitNOFILE=1000000

[Install]
WantedBy=multi-user.target
```

Run `systemctl daemon-reload` to apply the changes, and now your Nginx processes show 1M when you check their file-max limit:

    $ cat /proc/`ps -aux | grep -m 1 nginx | awk -F ' ' '{print $2}'`/limits | grep "open files" | awk -F ' ' '{print $4}'
    1000000

### Haproxy

Other software, such as [haproxy][haproxy], sometimes set their own file-max ulimit.
If you’ve increased the above limits and your haproxy child process still isn’t showing 1M then try adding `maxconn 1000000` to your `haproxy.cfg` file.
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
LimitNOFILE=1000000

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

Finally, after increasing your server's process file-max, you must also tweak your kernel’s TCP networking settings using sysctl:

* `net.ipv4.tcp_max_syn_backlog` - max half-open connections which the client has not yet ACK'd
* `net.core.somaxconn` - max backlogged connections which the client has ACK'd
* `net.core.netdev_max_backlog` - max packets in receive queue

The above 3 are the most important, but there’s more too:

* `net.core.rmem_max`
* `net.core.wmem_max`
* `net.ipv4.tcp_rmem`
* `net.ipv4.tcp_wmem`
* `net.ipv4.tcp_tw_reuse`
* `net.ipv4.ip_local_port_range`
* `net.ipv4.tcp_max_tw_buckets`

Check out [this Medium post][sysctl tips], Peter Mescalchin’s [collection of sysctl notes][sysctl notes], and the [sysctl docs][sysctl docs].
Use `sysctl -a` to list all supported configs on your machine, and see their default values.
Add any changes to your `/etc/sysctl.conf` file then run `sysctl -p` to apply them without needing a reboot.

### Conclusion

One final tool worth mentioning is [c1000k][c1000k], to check if your OS supports 1 million connections.
Increasing your file-max limit, and probably a few sysctl kernel tweaks, will unlock the full potential of your web server.
If you liked this post, also check out our previous post about a [disk-backed Redis compatible server called SSDB][blog post 45].
It removes the main limitation of Redis: your data set having to fit in available RAM.


[load balancing blog post]: https://wakatime.com/blog/23-how-to-scale-ssl-with-haproxy-and-nginx
[wakatime]: https://wakatime.com
[digitalocean]: https://www.digitalocean.com/
[file-max ram]: https://serverfault.com/questions/330795/what-are-the-ramifications-of-increasing-the-maximum-of-open-file-descriptors/330981#answer-330981:~:text=One%20file%20with%20associated%20inode%20and%20dcache%20is%20very%20roughly%201K.
[systemd]: https://www.freedesktop.org/software/systemd/man/systemd.service.html
[systemd nofile]: https://www.freedesktop.org/software/systemd/man/systemd.exec.html#LimitCPU=:~:text=LimitNOFILE%3D,-ulimit
[haproxy]: https://cbonte.github.io/haproxy-dconv/2.0/configuration.html#maxconn
[sysctl notes]: https://bl.ocks.org/magnetikonline/2760f98f6bf654d5ad79
[sysctl docs]: https://www.kernel.org/doc/Documentation/sysctl/net.txt
[sysctl tips]: https://medium.com/@pawilon/tuning-your-linux-kernel-and-haproxy-instance-for-high-loads-1a2105ea553e#edfd
[c1000k]: https://github.com/ideawu/c1000k
[blog post 45]: https://wakatime.com/blog/45-using-a-diskbased-redis-clone-to-reduce-aws-s3-bill
