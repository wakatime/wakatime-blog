---
Title: Latency of DigitalOcean Spaces vs AWS S3
Date: 2021-02-20
Image: https://wakatime.com/static/img/blog/s3-vs-spaces-latency.png
Description: Observations on reliability and latency differences between Amazon S3 and DigitalOcean Spaces object storage.
Author: Alan Hamlett
AuthorUrl: https://wakatime.com/@alan
AuthorGravatar: https://wakatime.com/gravatar/@alan
Category: Engineering
Tags: aws, s3, digitalocean, databases, devops
---

WakaTime’s infra is split across [DigitalOcean][digitalocean] and [AWS][aws].
We use DigitalOcean [Droplets][droplets] for compute resources, AWS [S3][s3] to store code stats, and DigitalOcean [Spaces][spaces] for backups.
You can find more info on this split infra decision [in this blog post][blog post 45].
The files we store in S3 are usually between 10KB and 50KB in size, and we store multiple terabytes of these files.
We don’t use Spaces CDN, and our Spaces are located in the same region as our Droplets.

Over the years, we’ve noticed some latency differences between Spaces and S3.
This is why [WakaTime][wakatime] only uses Spaces for backups and S3 as our main database.
Using S3 as a database did increase our [Outbound Data Transfer costs][s3 pricing], but we reduced those costs by [caching S3 reads using SSDB][blog post 45], a disk-backed Redis-compatible database.

### Latency of S3 vs Spaces

<img src="https://wakatime.com/static/img/blog/s3-vs-spaces-latency.png" class="img-thumbnail" alt="S3 vs Spaces latency" style="width:90%" />

The first thing we noticed was writing to DigitalOcean Spaces takes much longer than writing the same objects to S3.
Average write latency for S3 stays around 200ms when writing from DigitalOcean servers.
When writing the same files from the same servers to DigitalOcean Spaces, average write latency is around 2 seconds!😱
This wasn’t that bad for our use case since we parallelize writes, but it means writing terabytes of data to Spaces can take days.

Read latency was more important for us.
We notice consistently faster(lower) reads from S3 compared to Spaces.
Reading from S3 takes around 200ms per object, while reading from Spaces takes around 300ms.
We also noticed a higher rate of failures when reading from Spaces vs S3.
For Spaces we set our [boto3 max retries][max retries] to 5, but we didn’t even need retries for S3.

We start each file path prefix with a random string generated per user, to prevent one user’s reads from bottlenecking reads of other users.
According to [AWS docs][s3 performance], this means we can read up to 5.5k files per second per user from S3.

### WakaTime’s Infra

We’re now using S3 as our primary code stats database, with an [SSDB caching layer][blog post 45], and multiple Postgres databases on DigitalOcean block storage volumes.
WakaTime code stats come in from the [open source plugins][plugins] to the [WakaTime API][api] and are temporarily stored in a Postgres database sharded at the application-layer by day.
Multiple times per day, a background task runs on our RabbitMQ distributed task queue that moves code stats from Postgres into S3 and warming the SSDB cache at the same time.
A similar task also runs each day to backup the code stats into DigitalOcean Spaces.
Our backups in Spaces are [automatically versioned][spaces docs] by DigitalOcean.
If new code stats come in for the same S3 file, the change is replicated and versioned in the Spaces backup.
DigitalOcean Spaces is [priced very affordably][spaces pricing] and latency doesn’t matter as much for infrequent reads, making it a great place to store backups.

<img src="https://wakatime.com/static/img/blog/spaces-bucket-versioning.png" class="img-thumbnail" alt="Spaces versioning" style="width:90%" />

If you liked this post, you can browse similar articles using the [devops tag][devops tag].
Get started with your free programming metrics today by [installing the open source WakaTime plugin][wakatime].


[wakatime]: https://wakatime.com
[digitalocean]: https://www.digitalocean.com/
[droplets]: https://www.digitalocean.com/products/droplets/
[aws]: https://aws.amazon.com/
[s3]: https://aws.amazon.com/s3/
[s3 pricing]: https://aws.amazon.com/s3/pricing/
[spaces]: https://www.digitalocean.com/products/spaces/
[spaces pricing]: https://www.digitalocean.com/pricing/#spaces-object-storage
[s3 performance]: https://docs.aws.amazon.com/AmazonS3/latest/userguide/optimizing-performance.html
[spaces docs]: https://developers.digitalocean.com/documentation/spaces/
[max retries]: https://stackoverflow.com/a/48568320/1290627
[plugins]: https://wakatime.com/plugins
[api]: https://wakatime.com/api
[devops tag]: https://wakatime.com/blog/tag/devops
[blog post 45]: https://wakatime.com/blog/45-using-a-disk-based-redis-clone-to-reduce-aws-s3-bill
