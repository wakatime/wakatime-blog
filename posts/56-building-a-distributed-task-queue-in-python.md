---
Title: Building a Distributed Task Queue in Python
Date: 2022-09-05
Image: https://wakatime.com/static/img/blog/light-queue.png
Description: How I wrote my own background distributed task queue to replace Celery at WakaTime.
Author: Alan Hamlett
AuthorUrl: https://wakatime.com/@alan
AuthorAvatar: https://wakatime.com/photo/@alan?size=420
Category: Engineering
---

<img src="https://wakatime.com/static/img/blog/light-queue.png" class="img-responsive" alt="light queue" />

## Why not just use Celery/RQ/Huey/TaskTiger?

Unfortunately, WakaTime has been using Celery for almost 10 years now.
During that time I’ve experienced [many critical bugs][celery issues], some still open years after being introduced.
Celery used to be pretty good, but feature bloat made the project difficult to maintain.
Also in my opinion, splitting the code into [three separate GitHub repos][celery changes] made the codebase hard to read.

However, the main reason:
**Celery delayed tasks don’t scale.**

If you use Celery delayed tasks, as your website grows eventually you’ll start seeing [this error message][prefetch count]:

    QoS: Disabled: prefetch_count exceeds 65535

When that happens the worker stops processing all tasks, not just delayed ones!
As WakaTime grew, we started running into this bug more frequently.

I tried [RQ][rq], [Huey][huey], and [TaskTiger][tasktiger], but they were missing features and processed tasks slower than Celery.
A distributed task queue is indispensable for a website like WakaTime, and I was tired of running into bugs.
For that reason, I decided to build the simplest distributed task queue possible while still providing all the features required by WakaTime.

## Introducing WakaQ

[WakaQ][wakaq] is a new Python distributed task queue.
Use it to run code in the background so your website stays fast and snappy, and your users stay happy.

## WakaQ is simple

It’s only 1,264 lines of code!

    $ find . -name '*.py' -not -path "./migrations*" -not -path "./venv*" | xargs wc -l | grep " total" | awk '{print $1}' | numfmt --grouping
    1,264

It only took [one week][wakaq first commit] from the first line of code until fully replacing Celery at WakaTime.
That says something about it’s simplicity.

Each queue is implemented using a [Redis list][redis lists].
Delayed tasks get their own queues implemented using [Redis sorted sets][redis sorted sets].
Broadcast tasks share a single [Redis Pub/Sub queue][redis pubsub].

## WakaQ has all the necessary features

* Queue priorities
* Delayed tasks (run tasks after a timedelta eta)
* Scheduled cron periodic tasks
* [Broadcast][broadcast] tasks (run a task on all workers)
* Task [soft][soft timeout] and [hard][hard timeout] timeout limits
* Optionally retry tasks on soft timeouts
* Combats memory leaks by restarting workers when [max\_mem\_percent][max mem] reached
* Super minimal and maintainable

Features considered out of scope are rate limiting, exclusive locking, storing task results, and task chaining.
Those are easy to add in your application’s task code, and you probably want to implement these specific to your app’s needs anyway.

## WakaQ is ready to use

[WakaQ][wakaq] is stable and ready to use in production.
WakaQ currently powers all background tasks in prod for the [WakaTime][wakatime] website, including but not limited to:

* sending code stats email reports
* renewing our LetsEncrypt SSL certs
* pre caching dashboards, repo badges, and embeddable charts
* anything else we don’t want holding up the web requests

It’s released under a [BSD license][license], so you can use it in open and closed source projects.
If you find any bugs please [open an issue][wakaq issues], but think twice before requesting new features :-)

Happy coding!

\- [Alan][alan] from [WakaTime][wakatime]


[celery issues]: https://github.com/celery/celery/issues?q=author%3Aalanhamlett
[celery changes]: https://docs.celeryq.dev/en/2.5-archived/changelog.html
[rq]: https://github.com/rq/rq
[huey]: https://github.com/coleifer/huey
[tasktiger]: https://github.com/closeio/tasktiger
[prefetch count]: https://github.com/celery/celery/issues/3267
[wakaq]: https://github.com/wakatime/wakaq
[broadcast]: https://github.com/wakatime/wakaq/blob/761d08f06d7d88941491e48d1cb524a1c35788ad/wakaq/task.py#L47
[soft timeout]: https://github.com/wakatime/wakaq/blob/761d08f06d7d88941491e48d1cb524a1c35788ad/wakaq/exceptions.py#L8
[hard timeout]: https://github.com/wakatime/wakaq/blob/761d08f06d7d88941491e48d1cb524a1c35788ad/wakaq/worker.py#L370
[max mem]: https://github.com/wakatime/wakaq/blob/a11d22b6a743e4fb0e220673085297bdc4aab710/wakaq/worker.py#L339
[wakaq first commit]: https://github.com/wakatime/wakaq/commits/main?after=a11d22b6a743e4fb0e220673085297bdc4aab710+104&branch=main&qualified_name=refs%2Fheads%2Fmain
[redis lists]: https://redis.io/docs/data-types/lists/
[redis sorted sets]: https://redis.io/docs/data-types/sorted-sets/
[redis pubsub]: https://redis.io/docs/manual/pubsub/
[wakatime]: https://wakatime.com/
[license]: https://github.com/wakatime/wakaq/blob/main/LICENSE
[wakaq issues]: https://github.com/wakatime/wakaq/issues
[alan]: https://wakatime.com/@alan
