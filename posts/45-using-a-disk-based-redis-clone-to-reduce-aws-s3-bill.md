---
Title: Using a disk-based Redis clone to reduce AWS S3 bill
Date: 2021-02-19
Image: https://wakatime.com/static/img/blog/redis.png
Description: Using SSDB to solve the Redis RAM limitation.
Author: Alan Hamlett
AuthorUrl: https://wakatime.com/@alan
AuthorGravatar: https://wakatime.com/gravatar/@alan
Category: Engineering
Tags: redis, ssdb, caching, databases, devops
---

[Redis][redis] is an in-memory database with very high write and read speed, and a limitation that data sets can’t be larger than available RAM.
It’s like [memcached][memcached] but supports data structures instead of just strings as values.
Redis is great for caching lookups to AWS S3 from an external server, which can speed up your S3 reads and save you money on [Outgoing Data Transfer costs][s3 pricing].
However, having your entire data set limited to the size of available RAM on the machine means you can only cache a small fraction of your possible AWS S3 keys.
Also with Redis, if snapshotting ever fails because you’ve used more RAM than available, your [snapshot will fail][snapshot failure] and force your Redis instance into [read-only mode][read only mode].
This means your production Redis database might suddenly stop allowing writes until you restart it, yikes!😱

### Redis Cluster

Right now, you might be thinking of a solution… shard the data across a multi-machine Redis Cluster.
That increases your max OPS, but also has some downsides:

* more machines to maintain, trading one problem for another
* RAM is expensive, costs add up fast

### Caching S3 with Redis

[WakaTime][wakatime] uses [S3][s3] as a database for user code stats, because hosting for traditional relational databases with data sets in the terabytes is expensive.
Not to mention traditional relational databases don’t have the reliability, resiliency, scalability, and serverless features of S3.

WakaTime hosts programming dashboards on [DigitalOcean][digitalocean] servers.
That’s because DigitalOcean servers come with dedicated attached SSDs, and overall give you much more compute bang for the buck than EC2.
DigitalOcean’s S3 compatible object storage service ([Spaces][spaces]) is very inexpensive, but we couldn’t use DigitalOcean Spaces for our primary database because it’s [much slower than S3][blog post 46].

Transferring data from S3 to DigitalOcean servers costs $0.09 per GB, because Amazon charges a higher price for outgoing data transfers vs internal transfers.
Our monthly data transfer costs were $246.
Also, S3 charges separately for every read/write/list operation performed.
The request operations portion of our S3 bill was $549/mo.
Using S3 as a database was becoming a significant portion of our monthly AWS bill.

<img src="https://wakatime.com/static/img/blog/s3-monthly-costs.png" class="img-thumbnail" alt="S3 monthly costs" style="width:90%" />

Data storage in S3 is cheap, it’s the transfer and operations that get expensive.
Also, S3 comes with built-in replication and reliability so we want to keep using S3 as our database.
To reduce costs and improve performance, we tried caching S3 reads with Redis.
However, with the cache size limited by RAM it barely made a dent in our reads from S3.
The amount of RAM we would need to cache multiple terabytes of S3 data would easily cost more than our total AWS bill.
Instead, we decided to try several disk-based Redis alternatives and found one that fit our needs perfectly.

### SSDB - A Redis alternative

We’ve been [using a disk-backed Redis alternative][hn comment] in production for over 3 years now called [SSDB][ssdb].
SSDB is a drop-in replacement for Redis, so you don’t need to change any client libraries.
It uses [LevelDB][leveldb] (a key-value storage library by Google) behind the scenes to achieve performance comparable to Redis.
[According to benchmarks][ssdb performance], writes to SSDB are slightly slower but reads are actually faster than Redis!😲

SSDB supports [replication][ssdb replication] built-in and clustering with [twemproxy][twemproxy].
Most importantly, it stores your data set on disk using RAM for caching.
Our SSDB data set is 500GB and does a good job of lowering the number of reads we make to S3.
Remember to [increase the file-max][blog post 47] of your SSDB server to at least 10k, to prevent getting errors like `Too many open files` or `Connection reset by peer`.
Also, remember to periodically trigger garbage collection on your SSDB data files by running ssdb-cli compact with this crontab:

    newline=$'\n' && ssdb-cli <<< "compact${newline}q"

Or else you’ll see your SSDB disk usage [grow forever][ssdb compact] even after deleting keys.

### Conclusion

Using SSDB to cache S3 has significantly reduced the outbound data transfer portion of our AWS bill! 🎉
With performance comparable to Redis and using the disk to bypass Redis’s RAM limitation, SSDB is a powerful tool for lean startups.

If you liked this post, browse similar writeups using the [devops tag][devops tag].
To get started with your free code time insights today, [install the WakaTime plugin for your IDE][wakatime].


[wakatime]: https://wakatime.com
[memcached]: https://memcached.org/
[redis]: https://redis.io/
[snapshot failure]: https://redis.io/topics/faq#background-saving-fails-with-a-fork-error-under-linux-even-if-i-have-a-lot-of-free-ram
[read only mode]: https://redis.io/topics/faq#what-happens-if-redis-runs-out-of-memory
[ssdb]: https://github.com/ideawu/ssdb
[ssdb performance]: https://github.com/ideawu/ssdb#performance
[hn comment]: https://news.ycombinator.com/item?id=21185735
[leveldb]: https://github.com/google/leveldb
[ssdb replication]: http://ssdb.io/docs/replication.html
[twemproxy]: https://github.com/twitter/twemproxy
[digitalocean]: https://www.digitalocean.com/products/droplets/
[s3]: https://aws.amazon.com/s3/
[s3 pricing]: https://aws.amazon.com/s3/pricing/
[devops tag]: https://wakatime.com/blog/tag/devops
[spaces]: https://www.digitalocean.com/products/spaces/
[blog post 46]: https://wakatime.com/blog/46-latency-of-digitalocean-spaces-vs-aws-s3
[blog post 47]: https://wakatime.com/blog/47-maximize-your-concurrent-web-server-connections
[ssdb compact]: https://github.com/ideawu/ssdb/issues/1363
