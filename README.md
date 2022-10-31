# external-library-notes
Notes and helpful hints for the libraries that the Relevize project uses. 

## A highlevel look at what a bluprint is doing

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

Description + link to docs
THESE NOTES ARE SPECIFICALLY FOR VERSION ___. 
CHECK THE RELEVIZE PROJECT VERSION OF SQLALCHEMY, as these notes may become outdated.

table examples
Captains Log - just some tables that let us play with queries

Logs | Users | DailyTasks

| Item         | Price     | # In stock |
|--------------|-----------|------------|
| Juicy Apples | 1.99      | *7*        |
| Bananas      | **1.89**  | 5234       |

### getting data
get, get_first, get_first_or_none

### joining tables

### "where clause" (filter and filter_by)

### 

# Marshmallow

Description + link to docs

Explain how it relates with a model

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


    brand_assets = fields.Function(lambda obj: obj.brand_assets()) # difference between this and .Method

    
    calculated_value = fields.Method("calculate_my_value")




    def calculate_my_value(self, obj):
        # the obj is the entire object that the query returns
        # write something here
        pass
```


# APISPEC (doc, marshal_with, use_kwargs)

Description + link to docs
