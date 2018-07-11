---
Title: Part 2: Building a Flask RESTful API
Date: 2018-07-11
Image: https://wakatime.com/static/img/blog/flask-plus-sqlalchemy.png
Description: How to build an API with Flask and SQLAlchemy.
Author: Alan Hamlett
AuthorUrl: https://wakatime.com/@alan
AuthorGravatar: https://wakatime.com/gravatar/@alan
Category: Engineering
Tags: flask, python, sqlalchemy
Draft: true
---

*This is the second of three posts about building a JSON API with Flask. See [post 1][part1] and [post 3][part3].*

In the [previous post][part1] we learned how to serialize SQLAlchemy models to/from JSON.
Now let’s use that to build a RESTful JSON API with [Flask][flask].

## What is a RESTful API

A RESTfu API is a website that conforms to the [REST][rest] conventions by allowing [CRUD][crud] operations on resources.
Resources are the noun you are fetching or updating, for ex: Users or Goals.
Unlike [GraphQL][graphql], where every resource shares the same url endpoint, RESTful APIs use a different url for each resource.
That means Users would be at `yoursite.com/api/users` and Goals at `yoursite.com/api/goals`.
When you visit `yoursite.com/api/users` in your browser, it should return JSON with a list of users.

## A RESTful API with Flask

Here’s a simple Flask API that always returns an empty list of Users:

```python
from flask import Flask
app = Flask(__name__)

@app.route("/api/users")
def users():
    return "[]"

if __name__ == "__main__":
    app.run()
```

Let’s improve that to return a list of users from our database using SQLAlchemy.

```python
from flask import Flask, json
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
db = SQLAlchemy(app)

class User(BaseModel):
    id = db.Column(UUID(), primary_key=True, default=uuid.uuid4)
    username = db.Column(db.String(), nullabe=False, unique=True)
    password = db.Column(db.String())
    _default_fields = ["username"]
    _hidden_fields = ["password"]

@app.route("/api/users")
def users():
    return json.dumps([user.to_dict() for user in User.query.all()])

if __name__ == "__main__":
    app.run()
```

The `BaseModel` class is the one we created in [Part 1: SQLAlchemy Models as JSON][part1].

Visiting `yoursite.com/api/users` in your browser you should see:

```json
[{"id": "488345de-88a1-4c87-9304-46a1a31c9414", "username": "zzzeek"}]
```

Following this pattern, we create `GET` resources for all our SQLAlchemy models.

## Updating Models with PUT method

Most APIs support more than just `GET` requests.
To update Users let’s add a `PUT` method that accepts JSON and updates a User’s attributes.
This example depends on [WTForms][wtforms] and [WTForms-JSON][wtforms-json] libraries for input validation.

```python
@app.route("/api/users/<string:user_id>", methods=['PUT'])
def users_update(user_id):
    user = User.query.get(user_id)
    form = UserForm.from_json(request.get_json())
    if not form.validate():
        return jsonify(errors=form.errors), 400
    user.from_dict(**form.data)
    db.session.commit()
    return jsonify(user=user.to_dict())
```

Sending `{"username": "zoe"}` to `yoursite.com/api/users/488345de-88a1-4c87-9304-46a1a31c9414` updates User with a new username.
We validate user input with [WTForms][wtforms] and control which database columns are readable and writeable with our [BaseModel SQLAlchemy class][part1].

### Conclusion

Now that we’re building APIs with Flask, it’s time to add extra features with some useful decorators and more `BaseModel` helper methods.
Continue reading [Part 3: Flask API Decorators and Helpers][part3].

[flask]: https://www.palletsprojects.com/p/flask/
[rest]: https://en.wikipedia.org/wiki/Representational_state_transfer
[crud]: https://en.wikipedia.org/wiki/Create,_read,_update_and_delete
[graphql]: https://graphql.org/
[wtforms]: https://wtforms.readthedocs.io/en/stable/
[wtforms-json]: https://wtforms-json.readthedocs.io/en/latest/

[part1]: https://wakatime.com/blog/32-part-1-sqlalchemy-models-to-json
[part3]: https://wakatime.com/blog/34-part-3-flask-api-decorators-and-helpers
