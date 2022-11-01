# external-library-notes
Notes and helpful hints for the libraries that the Relevize project uses. 

## A highlevel look at what a bluprint is doing

TODO: go line by line and explain what is going on

```py
@blueprint.route("", methods=["GET"], provide_automatic_options=False) # thing
@use_kwargs                                                            # thing
@marshal_with(DataSchema(many=True))                                   # thing
@doc(                                                                  # thing
    description="Allows users to get data, if they are a vendor",      # thing
    tags=["v2_data_getter"],                                           # thing
)
@authenticate(mode="valid")                                            # thing
def get_some_data(user_obj):
    (vendor, partner) = user_obj.entity_for_user()                     # thing

    if partner:
        abort(403)

    vendors = Advertiser.query.filter(Advertiser.ad_properties.contains(partner))

    return vendors
```

# SQLAlchemy

`SQLAlchemy is the Python SQL toolkit and Object Relational Mapper that gives application developers the full power and flexibility of SQL.`

[v1.2 Docs](https://docs.sqlalchemy.org/en/12/)

These notes are specifically for the version listed above. Consult the Relevize project to be sure these docs are not out of date at the time of reading this.

Example Tables

Users Table
| id     | name             | rank                 |
|--------|------------------|----------------------|
| 1      | Bradward Boimler | Ensign               |
| 2      | Data             | Lieutenant Commander |
| 3      | Christopher Pike | Captain              |

Logs Table
| id     | user_id  | content              |
|--------|----------|----------------------|
| 11     | 1        | dui accumsan         |
| 22     | 2        | aliquet eget         |
| 33     | 3        | rutrum quisque       |

ShiftTasks Table
| id     | user_id  | completed  | task           |
|--------|----------|------------|----------------|
| 11     | 1        | True       | malesuada nunc |
| 22     | 2        | False      | tristique et   |
| 33     | 3        | False      | volutpat odio  |


## FAQ (frequently asked queries 🤣)
If the definition is in quotes, it is copy/pasted from the [tutorial docs](https://docs.sqlalchemy.org/en/12/orm/tutorial.html). If they are not they were written by a team memeber, and should maybe be taken with a grain of salt.

The anatomy of adding and removing data will follow a standard transaction model.
```py
# You must import the db object - this is where we keep it in the Relevize project
flaskapp.extensions import db 

# Then we can add, modify, or delete data:

# Add data
new_user = User(name="tsuki", rank="captain")
db.session.add(new_user)                           

# Modify data
user_to_update = User.query.get(42)
user_to_update.rank = "Captain"
db.session.add(user_to_update)

# Delete data
user_to_delete = User.query.get(43)
db.session.add(user_to_delete)

# Then you commit all of the above changes to the database
db.session.commit()
```

### Getting Data
* `.all()`
    * Returns all values that the query finds.
    * `User.query.filter_by(name="tsuki").all()`
* `.get(<primary_key>)`
    * "Return an instance based on the given primary key identifier, or None if not found."
    * `User.query.get(42)`
* `.count()`
    * "The `.count()` method is used to determine how many rows the SQL statement would return."
    * `User.query.count()`
* `.limit()`
    * Limits the number of results that we recieve.
    * `User.query.limit(25)`
* `.offset()`
    * Works with limit to indicate which items are recieved (1-25 / 150)
    * `User.query.offset(2).limit(25)`

### Filtering Data
* `.filter_by()`
    * Seems to be used for quering for direct column values.
    * `User.query.filter_by(name="tsuki")`
* `.filter()`
    * Seems to be used for more robust queries
    * `User.query.filter(User.id == Logs.user_id)`
* Filtering with the `LIKE` operator
    * `User.query.filter(User.name.like('%ed%'))` - if the `%` symbol looks foreign to you, go google postgres like operators to get a better understanding of the syntax.

### Getting One (or None) Items
* `.first()`
    * "Applies a limit of one and returns the first result."
    * `User.query.filter_by(name="tsuki").first()`
* `.one()`
    * "Fully fetches all rows, and if not exactly one object identity or composite row is present in the result, raises an error."
    * `User.query.get(42).one()`
* `.one_or_none()`
    * "`.one_or_none()` is like Query.one(), except that if no results are found, it doesn’t raise an error; it just returns None. Like Query.one(), however, it does raise an error if multiple results are found."
    *  `User.query.filter_by(id=42).one_or_none()`

### Ordering Data
* `.order_by()`
    * order your returning values - by default results will be in ascending order
    ```py
    User.query.order_by(User.created_at) # order returning values by timestamp
    User.query.order_by(User.id)         # order returning values by id
    User.query.order_by(User.id.desc())  # this will return ids in descending order
    User.query.order_by(User.id.asc())   # you can explicitly say you want return values in ascending order
    ```
* `.group_by()`
    * Groups return values into summary rows (group all _ by _)
    * `Logs.query.group_by(user_id)`

### Create Data
Updating involves creating a new object, and adding it to the session:
```py
cool_new_user = User(name="tsuki", rank="captain")
db.session.add(cool_new_user)
db.session.commit()
```

### Updating Data
Updating seems to be a multi-step process, where you first must find the item you want to update, then update it.
```py
tsuki = User.query.filter_by(name="tsuki")
tsuki.rank = "Captain"
db.session.add(tsuki)
db.session.commit()
```

### Deleting Data
* `.delete()`
    * The tutorial docs don't spend much time on this.
    * `User.query.filter_by(id=42).delete()`

### Joining Tables
One can implicitly joining based on foreign key: `User.query.join(Log)`, "... .join() knows how to join between (table 1) and (table 2) because there’s only one foreign key between them.".
    * `User.query.join(Log)`
```py
.join(Address, User.id==Address.user_id)    # explicit condition
.join(User.addresses)                       # specify relationship from left to right
.join(Address, User.addresses)              # same, with explicit target
.join('addresses')                          # same, using a string
```


# Marshmallow

`marshmallow is an ORM/ODM/framework-agnostic library for converting complex datatypes, such as objects, to and from native Python datatypes.`

[v3 Docs](https://marshmallow.readthedocs.io/en/stable/)

TODO: Explain how it relates with a model

Frequently used field options:
```py
class ExampleSchema(Schema):
    id = fields.Integer
    name = fields.String()
    campaign_names = fields.List(fields.String())
    campaigns = fields.List(fields.Nested(CampaignsSchema)) # explain this
    users = fields.Nested(UserSchema(many=True)) 

    # how to alias a value
    my_campaign_names = fields.List(fields.String(
        attribute="campaign_names", default=""
    ))    


    # TODO: difference between this and .Method and .Function
    brand_assets = fields.Function(lambda obj: obj.brand_assets())

    
    calculated_value = fields.Method("calculate_my_value")


    def calculate_my_value(self, obj):
        # the obj is the entire object that the query returns
        # TODO write something here
        pass
```


# API_SPEC (doc, marshal_with, use_kwargs)

`flask-apispec is a lightweight tool for building REST APIs in Flask. flask-apispec uses webargs for request parsing, marshmallow for response formatting, and apispec to automatically generate Swagger markup. You can use flask-apispec with vanilla Flask or a fuller-featured framework like Flask-RESTful.`

These are the [flask-apispec docs](https://flask-apispec.readthedocs.io/en/latest/). These are the [apispec docs](https://pypi.org/project/apispec/), whos api reference can be used.

