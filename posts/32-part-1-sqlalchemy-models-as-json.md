---
Title: Part 1: SQLAlchemy Models to JSON
Date: 2018-07-11
Image: https://wakatime.com/static/img/blog/flask-plus-sqlalchemy.png
Description: How to serialize SQLAlchemy models into json.
Author: Alan Hamlett
AuthorUrl: https://wakatime.com/@alan
AuthorGravatar: https://wakatime.com/gravatar/@alan
Category: Engineering
Tags: flask, python, sqlalchemy
---

*This is the first of three posts about building a JSON API with Flask.*
<br />*[Part 2][part2] and [part 3][part3] arrive tomorrow and the day after.*

I've [seen][so1] [a][so2] [lot][so3] [of][so4] [questions][so5] on StackOverflow about how to turn SQLAlchemy models into JSON.
When building a JSON API with Flask and [SQLAlchemy][sqlalchemy], you end up writing a lot of boilerplate api code just to serialize your models into JSON.
Since I encountered this problem early on at WakaTime, I decided to share my solution here.
You have some libraries available to help such as [Flask-RESTful][flask-restful], [Flask-Restless][flask-restless], or [flask-restutils][flask-restutils].
However, back in 2013 when I started WakaTime these either weren’t available or weren’t stable.
Thankfully it was trivial to loop through the columns of an SQLAlchemy model and produce a `dict`.
That `dict` object could then be jsonified with `json.dumps` and returned from an API.

## SQLAlchemy Model to Dictionary

To add a serialization method have all your SQLAlchemy models inherit from an abstract base class.
This base class defines the `to_dict` method that loops through the model’s columns and returns a dictionary.

```python
from flask import json
from flask_sqlalchemy import SQLAlchemy
from sqlalchemy.orm.attributes import QueryableAttribute
from wakatime_website import app

db = SQLAlchemy(app)

class BaseModel(db.Model):
    __abstract__ = True

    def to_dict(self, show=None, _hide=[], _path=None):
        """Return a dictionary representation of this model."""

        show = show or []

        hidden = self._hidden_fields if hasattr(self, "_hidden_fields") else []
        default = self._default_fields if hasattr(self, "_default_fields") else []
        default.extend(['id', 'modified_at', 'created_at'])

        if not _path:
            _path = self.__tablename__.lower()

            def prepend_path(item):
                item = item.lower()
                if item.split(".", 1)[0] == _path:
                    return item
                if len(item) == 0:
                    return item
                if item[0] != ".":
                    item = ".%s" % item
                item = "%s%s" % (_path, item)
                return item

            _hide[:] = [prepend_path(x) for x in _hide]
            show[:] = [prepend_path(x) for x in show]

        columns = self.__table__.columns.keys()
        relationships = self.__mapper__.relationships.keys()
        properties = dir(self)

        ret_data = {}

        for key in columns:
            if key.startswith("_"):
                continue
            check = "%s.%s" % (_path, key)
            if check in _hide or key in hidden:
                continue
            if check in show or key in default:
                ret_data[key] = getattr(self, key)

        for key in relationships:
            if key.startswith("_"):
                continue
            check = "%s.%s" % (_path, key)
            if check in _hide or key in hidden:
                continue
            if check in show or key in default:
                _hide.append(check)
                is_list = self.__mapper__.relationships[key].uselist
                if is_list:
                    items = getattr(self, key)
                    if self.__mapper__.relationships[key].query_class is not None:
                        if hasattr(items, "all"):
                            items = items.all()
                    ret_data[key] = []
                    for item in items:
                        ret_data[key].append(
                            item.to_dict(
                                show=list(show),
                                _hide=list(_hide),
                                _path=("%s.%s" % (_path, key.lower())),
                            )
                        )
                else:
                    if (
                        self.__mapper__.relationships[key].query_class is not None
                        or self.__mapper__.relationships[key].instrument_class
                        is not None
                    ):
                        item = getattr(self, key)
                        if item is not None:
                            ret_data[key] = item.to_dict(
                                show=list(show),
                                _hide=list(_hide),
                                _path=("%s.%s" % (_path, key.lower())),
                            )
                        else:
                            ret_data[key] = None
                    else:
                        ret_data[key] = getattr(self, key)

        for key in list(set(properties) - set(columns) - set(relationships)):
            if key.startswith("_"):
                continue
            if not hasattr(self.__class__, key):
                continue
            attr = getattr(self.__class__, key)
            if not (isinstance(attr, property) or isinstance(attr, QueryableAttribute)):
                continue
            check = "%s.%s" % (_path, key)
            if check in _hide or key in hidden:
                continue
            if check in show or key in default:
                val = getattr(self, key)
                if hasattr(val, "to_dict"):
                    ret_data[key] = val.to_dict(
                        show=list(show),
                        _hide=list(_hide), _path=("%s.%s" % (_path, key.lower()))
                        _path=('%s.%s' % (path, key.lower())),
                    )
                else:
                    try:
                        ret_data[key] = json.loads(json.dumps(val))
                    except:
                        pass

        return ret_data
```

Now we use this base class to print a `User` as a dictionary.

```python
class User(BaseModel):
    id = db.Column(UUID(), primary_key=True, default=uuid.uuid4)
    username = db.Column(db.String(), nullabe=False, unique=True)
    password = db.Column(db.String())
    email_confirmed = db.Column(db.Boolean())
    modified_at = db.Column(db.DateTime())
    created_at = db.Column(db.DateTime(), nullable=False, default=datetime.utcnow)

    _default_fields = [
        "username",
        "joined_recently",
    ]
    _hidden_fields = [
        "password",
    ]
    _readonly_fields = [
        "email_confirmed",
    ]

    @property
    def joined_recently(self):
        return self.created_at > datetime.utcnow() - timedelta(days=3)

user = User(username="zzzeek")
db.session.add(user)
db.session.commit()

print(user.to_dict())
```

Which prints:

```python
{
    'id': UUID('488345de-88a1-4c87-9304-46a1a31c9414'),
    'username': 'zzzeek',
    'joined_recently': True,
    'modified_at': None,
    'created_at': datetime.datetime(2018, 7, 11, 6, 28, 56, 905379),
}
```

And is easily jsonified with:

```python
json.dumps(user.to_dict())
```

## Defaults and hidden fields

You might have noticed the attributes on `User` listing default and hidden fields.
These allow you to customize which columns from `User` are included in the returned dictionary.
For example, if you want to include `email_confirmed` in your serialized user you would do:

```python
print(user.to_dict(show=['email_confirmed', 'password']))
```

Which prints:

```python
{
    'id': UUID('488345de-88a1-4c87-9304-46a1a31c9414'),
    'username': 'zzzeek',
    'email_confirmed': None,
    'joined_recently': True,
    'modified_at': None,
    'created_at': datetime.datetime(2018, 7, 11, 6, 28, 56, 905379),
}
```

Also notice that `password` was not included, since it’s listed as hidden on `User`.

## Updating an SQLAlchemy Model from a Dictionary

We have a `to_dict` method, but to support POST, PUT, and PATCH methods we need a `from_dict` method that takes a dictionary and updates the model’s columns with the provided data.
Let’s add the `from_dict` method like this:

```python
from sqlalchemy.sql.expression import not_

class BaseModel(db.Model):
    __abstract__ = True

    def __init__(self, **kwargs):
        kwargs["_force"] = True
        self.from_dict(**kwargs)

    def to_dict(self, show=None, _hide=[], _path=None):
        ...

    def from_dict(self, **kwargs):
        """Update this model with a dictionary."""

        _force = kwargs.pop("_force", False)

        readonly = self._readonly_fields if hasattr(self, "_readonly_fields") else []
        if hasattr(self, "_hidden_fields"):
            readonly += self._hidden_fields

        readonly += ["id", "created_at", "modified_at"]

        columns = self.__table__.columns.keys()
        relationships = self.__mapper__.relationships.keys()
        properties = dir(self)

        changes = {}

        for key in columns:
            if key.startswith("_"):
                continue
            allowed = True if _force or key not in readonly else False
            exists = True if key in kwargs else False
            if allowed and exists:
                val = getattr(self, key)
                if val != kwargs[key]:
                    changes[key] = {"old": val, "new": kwargs[key]}
                    setattr(self, key, kwargs[key])

        for rel in relationships:
            if key.startswith("_"):
                continue
            allowed = True if _force or rel not in readonly else False
            exists = True if rel in kwargs else False
            if allowed and exists:
                is_list = self.__mapper__.relationships[rel].uselist
                if is_list:
                    valid_ids = []
                    query = getattr(self, rel)
                    cls = self.__mapper__.relationships[rel].argument()
                    for item in kwargs[rel]:
                        if (
                            "id" in item
                            and query.filter_by(id=item["id"]).limit(1).count() == 1
                        ):
                            obj = cls.query.filter_by(id=item["id"]).first()
                            col_changes = obj.from_dict(**item)
                            if col_changes:
                                col_changes["id"] = str(item["id"])
                                if rel in changes:
                                    changes[rel].append(col_changes)
                                else:
                                    changes.update({rel: [col_changes]})
                            valid_ids.append(str(item["id"]))
                        else:
                            col = cls()
                            col_changes = col.from_dict(**item)
                            query.append(col)
                            db.session.flush()
                            if col_changes:
                                col_changes["id"] = str(col.id)
                                if rel in changes:
                                    changes[rel].append(col_changes)
                                else:
                                    changes.update({rel: [col_changes]})
                            valid_ids.append(str(col.id))

                    # delete rows from relationship that were not in kwargs[rel]
                    for item in query.filter(not_(cls.id.in_(valid_ids))).all():
                        col_changes = {"id": str(item.id), "deleted": True}
                        if rel in changes:
                            changes[rel].append(col_changes)
                        else:
                            changes.update({rel: [col_changes]})
                        db.session.delete(item)

                else:
                    val = getattr(self, rel)
                    if self.__mapper__.relationships[rel].query_class is not None:
                        if val is not None:
                            col_changes = val.from_dict(**kwargs[rel])
                            if col_changes:
                                changes.update({rel: col_changes})
                    else:
                        if val != kwargs[rel]:
                            setattr(self, rel, kwargs[rel])
                            changes[rel] = {"old": val, "new": kwargs[rel]}

        for key in list(set(properties) - set(columns) - set(relationships)):
            if key.startswith("_"):
                continue
            allowed = True if _force or key not in readonly else False
            exists = True if key in kwargs else False
            if allowed and exists and getattr(self.__class__, key).fset is not None:
                val = getattr(self, key)
                if hasattr(val, "to_dict"):
                    val = val.to_dict()
                changes[key] = {"old": val, "new": kwargs[key]}
                setattr(self, key, kwargs[key])

        return changes
```

Using the new `from_dict` method we update our user with a dictionary:

```python
updates = {
    "username": "zoe",
    "email_confirmed": True,
}
user.from_dict(**updates)
db.session.commit()

print(user.to_dict(show=['email_confirmed']))
```

Which prints:

```python
{
    'id': UUID('488345de-88a1-4c87-9304-46a1a31c9414'),
    'username': 'zoe',
    'email_confirmed': None,
    'joined_recently': True,
    'modified_at': datetime.datetime(2018, 7, 11, 6, 36, 47, 939084),
    'created_at': datetime.datetime(2018, 7, 11, 6, 28, 56, 905379),
}
```

Notice that `email_confirmed` is still `None` because it’s marked as read only.

## Relationships

Our `to_dict` and `from_dict` methods also work for relationships.
For example, when our `User` model has many `Goal` models we can serialize `Goal` by default or with `show`:

```python
class User(BaseModel):
    ...
    goals = db.relationship('Goal', backref='user', lazy='dynamic')

class Goal(BaseModel):
    id = db.Column(UUID(), primary_key=True, default=uuid.uuid4)
    title = db.Column(db.String(), nullabe=False)
    accomplished = db.Column(db.Boolean())
    created_at = db.Column(db.DateTime(), nullable=False, default=datetime.utcnow)

    _default_fields = [
        "title",
    ]

goal = Goal(title="Mountain", accomplished=True)
user.goals.append(goal)
db.session.commit()

print(user.to_dict(show=['goals', 'goals.accomplished']))
```

Which prints:

```python
{
    'id': UUID('488345de-88a1-4c87-9304-46a1a31c9414'),
    'username': 'zoe',
    'goals': [
        {
            'id': UUID('c72cfef0-0988-45e4-9f4b-8a4a7d4f8d8f'),
            'title': 'Mountain',
            'accomplished': True,
            'created_at': datetime.datetime(2018, 7, 11, 6, 45, 18, 299924),
        },
    ],
    'joined_recently': True,
    'modified_at': datetime.datetime(2018, 7, 11, 6, 36, 47, 939084),
    'created_at': datetime.datetime(2018, 7, 11, 6, 28, 56, 905379),
}
```

It even allows customizing columns of relationships, for ex: `goals.accomplished`.

To see how this all fits into a RESTful API continue with [Part 2: Building a Flask RESTful API][part2].

[so1]: https://stackoverflow.com/questions/5022066/how-to-serialize-sqlalchemy-result-to-json
[so2]: https://stackoverflow.com/questions/7102754/jsonify-a-sqlalchemy-result-set-in-flask
[so3]: https://stackoverflow.com/questions/13539082/restful-interface-in-flask-and-issues-serializing
[so4]: https://stackoverflow.com/questions/25737050/jsonify-flask-sqlalchemy-many-to-one-relationship-in-flask
[so5]: https://stackoverflow.com/questions/37783919/flask-restful-vs-flask-restless-which-should-be-used-and-when
[sqlalchemy]: https://www.sqlalchemy.org/
[flask-restful]: https://flask-restful.readthedocs.io/en/latest/
[flask-restless]: https://flask-restless.readthedocs.io/en/stable/
[flask-restutils]: https://github.com/closeio/flask-restutils

[part2]: https://wakatime.com/blog/33-part-2-building-a-flask-restful-api
[part3]: https://wakatime.com/blog/34-part-3-flask-api-decorators-and-helpers
