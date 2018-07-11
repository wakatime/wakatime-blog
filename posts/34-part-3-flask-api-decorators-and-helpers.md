---
Title: Part 3: Flask API Decorators and Helpers
Date: 2018-07-12
Image: https://wakatime.com/static/img/blog/flask-plus-sqlalchemy.png
Description: Decorator functions and base model methods for Flask APIs.
Author: Alan Hamlett
AuthorUrl: https://wakatime.com/@alan
AuthorGravatar: https://wakatime.com/gravatar/@alan
Category: Engineering
Tags: flask, python, sqlalchemy
Draft: true
---

*This is the third of three posts about building a JSON API with Flask. See [post 1][part1] and [post 2][part2].*

In the [first][part1] post, we used a custom base SQLAlchemy class to serialize and deserialize database models to and from JSON.
The [second][part2] post created a RESTful API listing and updating users.
Now we’re ready for extra API features such as caching responses, rate limiting, and preventing brute forcing.
Also, we add a helper method to our `BaseModel` class from the [first post][part1] for fetching rows or creating them when they don’t already exist.

## Decorators

Use these decorators to add extra functionality to your API view functions.

For example, to rate limit and cache your users api resource:

```python
@app.route("/api/users")
@rate_limited
@cached
def users():
    return json.dumps([user.to_dict() for user in User.query.all()])
```

Order of decorators matters.
Since decorators are executed downwards, make sure `@cached` is always below `@rate_limited` and `@protected`.

To customize decorator options, pass arguments to the decorators:

```python
@app.route("/api/users")
@rate_limited(limit=50, minutes=60)  # only 50 requests from user/ip to this endpoint allowed per hour
@protected(limit=2, minutes=720)  # only 2 404 requests from same ip to this endpoint allowed per 12 hours
@cached(minutes=5)  # response cached for 5 minutes
def users():
    return json.dumps([user.to_dict() for user in User.query.all()])
```

#### Cache an API response with memcached

```python
import hashlib
import memcache
import traceback
from flask import request
from functools import wraps
from wakatime_website import app
from werkzeug.contrib.cache import MemcachedCache

mc = memcache.Client()
cache = MemcachedCache(mc)

def cached(fn=None, unique_per_user=True, minutes=30):
    """Caches a Flask route/view in memcached.

    The request url, args, and current user are used to build the cache key.
    Only GET requests are cached.
    By default, cached requests expire after 30 minutes.
    """

    if not isinstance(minutes, int):
        raise Exception('Minutes must be an integer number.')

    def wrapper(func):
        @wraps(func)
        def inner(*args, **kwargs):
            if request.method != 'GET':
                return func(*args, **kwargs)

            prefix = 'flask-request'
            path = request.full_path
            user_id = app.current_user.id if app.current_user.is_authenticated else None
            key = u('{user}-{method}-{path}').format(
                user=user_id,
                method=request.method,
                path=path,
            )
            hashed = hashlib.md5(key.encode('utf8')).hexdigest()
            hashed = '{prefix}-{hashed}'.format(prefix=prefix, hashed=hashed)

            try:
                resp = cache.get(hashed)
                if resp:
                    return resp
            except:
                app.logger.error(traceback.format_exc())
                resp = None

            resp = func(*args, **kwargs)
            try:
                cache.set(hashed, resp, timeout=minutes * 60)
            except:
                app.logger.error(traceback.format_exc())
            return resp

        return inner
    return wrapper(fn) if fn else wrapper
```

#### Rate limit API requests by IP or Current User

```python
import redis
from flask import abort, request
from functools import wraps
from wakatime_website import app

r = redis.Redis(decode_responses=True)

def rate_limited(fn=None, limit=20, methods=[], ip=True, user=True, minutes=1):
    """Limits requests to this endpoint to `limit` per `minutes`."""

    if not isinstance(limit, int):
        raise Exception('Limit must be an integer number.')
    if limit < 1:
        raise Exception('Limit must be greater than zero.')

    def wrapper(func):
        @wraps(func)
        def inner(*args, **kwargs):
            if not methods or request.method in methods:

                try:
                    if ip:
                        increment_counter(type='ip', for_methods=methods,
                                          minutes=minutes)
                        count = get_count(type='ip', for_methods=methods)
                        if count > limit:
                            abort(429)

                    if user and app.current_user.is_authenticated:
                        increment_counter(type='user', for_methods=methods,
                                          minutes=minutes)
                        count = get_count(type='user', for_methods=methods)
                        if count > limit:
                            abort(429)
                except:
                    pass

            return func(*args, **kwargs)

        return inner
    return wrapper(fn) if fn else wrapper

def get_counter_key(type=None, for_only_this_route=True, for_methods=None): if not isinstance(for_methods, list):
        for_methods = []
    if type == 'ip':
        key = request.remote_addr
    elif type == 'user':
        key = app.current_user.id if app.current_user.is_authenticated else None
    else:
        raise Exception('Unknown rate limit type: {0}'.format(type))
    route = ''
    if for_only_this_route:
        route = '{endpoint}'.format(
            endpoint=request.endpoint,
        )
    return u('{type}-{methods}-{key}{route}').format(
        type=type,
        key=key,
        methods=','.join(for_methods),
        route=route,
    )

def increment_counter(type=None, for_only_this_route=True, for_methods=None,
                      minutes=1):
    if type not in ['ip', 'user']:
        raise Exception('Type must be ip or user.')

    key = get_counter_key(type=type, for_only_this_route=for_only_this_route,
                          for_methods=for_methods)
    r.incr(key)
    r.expire(key, time=60 * minutes)

def get_count(type=None, for_only_this_route=True, for_methods=None):
    key = get_counter_key(type=type, for_only_this_route=for_only_this_route,
                          for_methods=for_methods)
    return int(r.get(key) or 0)
```

#### Prevent brute forcing secrets or tokens

```python
import redis
from flask import abort, request
from functools import wraps
from wakatime_website import app
from werkzeug.exceptions import NotFound

r = redis.Redis(decode_responses=True)

def protected(fn=None, limit=10, minutes=60):
    """Bans IP after requesting a protected resource too many times.

    Prevents IP from making more than `limit` requests per `minutes` to
    the decorated route. Prevents enumerating secrets or tokens from urls or
    query arguments by blocking requests after too many 404 not found errors.
    """

    if not isinstance(limit, int):
        raise Exception('Limit must be an integer number.')
    if not isinstance(minutes, int):
        raise Exception('Minutes must be an integer number.')

    def wrapper(func):
        @wraps(func)
        def inner(*args, **kwargs):
            key = u('bruteforce-{}-{}').format(request.endpoint, request.remote_addr)
            try:
                count = int(r.get(key) or 0)
                if count > limit:
                    r.incr(key)
                    seconds = 60 * minutes
                    r.expire(key, time=seconds)
                    app.logger.info('Request blocked by protected decorator.')
                    abort(403)
            except:
                app.logger.error(traceback.format_exc())

            try:
                result = func(*args, **kwargs)
            except NotFound:
                try:
                    r.incr(key)
                    seconds = 60 * minutes
                    r.expire(key, time=seconds)
                except:
                    pass
                raise

            if isinstance(result, tuple) and len(result) > 1 and result[1] == 404:
                try:
                    r.incr(key)
                    seconds = 60 * minutes
                    r.expire(key, time=seconds)
                except:
                    pass

            return result

        return inner
    return wrapper(fn) if fn else wrapper
```

## Get or Create SQLAlchemy Helper

A common pattern is to fetch a User from the database, creating one if necessary, then update some attributes on that user and save back to the database.
With SQLAlchemy, this looks something like:

```python
user = User.query.filter_by(username="zzzeek").first()
if not user:
    user = User(username="zzzeek")
    db.session.add(user)
user.first_name = "Mike"
db.session.commit()
```

This works, but means repeating boilerplate code.
It’s also prone to errors from race conditions, when a user is created with the same username after the first query but before the commit.

A better way is adding a `get_or_create` convenience method to the `BaseModel` SQLAlchemy class from the [previous post][part1]:

```python
from sqlalchemy.exc import IntegrityError, OperationalError

class BaseModel(db.Model):
    __abstract__ = True

    ...

    @classmethod
    def _get_or_create(
        cls,
        _session=None,
        _filters=None,
        _defaults={},
        _retry_count=0,
        _max_retries=3,
        **kwargs
    ):
        if not _session:
            _session = db.session
        query = _session.query(cls)
        if _filters is not None:
            query = query.filter(*_filters)
        if len(kwargs) > 0:
            query = query.filter_by(**kwargs)

        instance = query.first()
        if instance is not None:
            return instance, False

        _session.begin_nested()
        try:
            kwargs.update(_defaults)
            instance = cls(**kwargs)
            _session.add(instance)
            _session.commit()
            return instance, True

        except IntegrityError:
            _session.rollback()
            instance = query.first()
            if instance is None:
                raise
            return instance, False

        except OperationalError:
            _session.rollback()
            instance = query.first()
            if instance is None:
                if _retry_count < _max_retries:
                    return cls._get_or_create(
                        _filters=_filters,
                        _defaults=_defaults,
                        _retry_count=_retry_count + 1,
                        _max_retries=_max_retries,
                        **kwargs
                    )
                raise
            return instance, False

    @classmethod
    def get_or_create(cls, **kwargs):
        return cls._get_or_create(**kwargs)[0]
```

Now using this helper, our above code becomes:

```python
updates = {"first_name": "Mike"}
user = User.get_or_create(username="zzzeek", _defaults=updates)
user.from_dict(**updates)
db.session.commit()
```

### Conclusion

Hopefully these patterns and base methods will make creating APIs with Flask a breeze!

By the way, [WakaTime][wakatime] is built with Flask along with these patterns ;)

[wakatime]: https://wakatime.com

[part1]: https://wakatime.com/blog/32-part-1-sqlalchemy-models-to-json
[part2]: https://wakatime.com/blog/33-part-2-building-a-flask-restful-api
