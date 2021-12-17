# Jbuilder Exercises

This project consists of a party tracker database that stores information about
parties, guests, gifts, and invitations. The models and controllers are already
built. Your primary task is to build the API views, writing Jbuilder templates
so that calls to the API will return the right information, correctly formatted.

## Phase 0: Setup

To get started, clone the starter repo from the `Download Project` link at the
bottom of this page. Run `rails db:setup`, which will create, load, and seed
your database. Note that the starter does not have any migrations ready to run;
`db:setup` loads the database configuration directly from the schema by running
`rails db:schema:load` behind the scenes.

The first thing you need to do is set up your routes. Note that all of your
controllers are nested in an __api__ folder and that each controller's class
name begins with `Api::`. This means that your controllers are defined under an
`Api` namespace, so you need to specify that namespace in the routes as well.
For the purposes of this exercise, you only need to define `show` and `index`
routes for `guests`, `gifts`, and `parties`. For `gifts`, nest the `index` route
under the `guests` resource using the [`shallow`] option. Use the code snippet
below as a guide for the formatting:

```ruby
namespace :api, defaults: { format: :json } do
  # Your routes here
end
```

Run `rails routes` to ensure your routes are set up as intended. Your routes
should match the following (the order is not important):

```sh
         Prefix Verb URI Pattern                           Controller#Action
api_guest_gifts GET  /api/guests/:guest_id/gifts(.:format) api/gifts#index {:format=>:json}
       api_gift GET  /api/gifts/:id(.:format)              api/gifts#show {:format=>:json}
     api_guests GET  /api/guests(.:format)                 api/guests#index {:format=>:json}
      api_guest GET  /api/guests/:id(.:format)             api/guests#show {:format=>:json}
    api_parties GET  /api/parties(.:format)                api/parties#index {:format=>:json}
      api_party GET  /api/parties/:id(.:format)            api/parties#show {:format=>:json}
```

Keep the [Jbuilder docs][docs-link] open and use them for reference as you work.

A few notes before you begin:

- Just like with ERB views, Jbuilder views have access to any instance variables
  defined within the controller action that renders the view.

- You can run any Ruby code you want in a Jbuilder template, including
  conditionals. This ends up being really helpful when, for example, you only
  want to send certain user data only if the user requesting it is logged in.

- As you did with HTML and ERB, you can build Jbuilder partials and render them
  using `json.partial!` in your templates.

- Anything that follows `json.` without a `!` will become a literal key name. To
  set key names dynamically, use `json.set!` followed by the expression to be
  evaluated for the key name. For example,

  ```rb
  name = "joe"
  json.name name
  json.set! name, :name
  ```

  yields

  ```rb
  name: "joe"
  joe: "name"
  ```

- You can nest your data in an object under a given key by using a block.
  Consider this example from the [Jbuilder docs][docs-link]:

  ```ruby
  json.author do
    json.name @message.creator.name.familiar
    json.email_address @message.creator.email_address_with_name
    json.url url_for(@message.creator, format: :json)
  end
  ```

  yields:

  ```json
   "author": {
      "name": "David H.",
      "email_address": "'David Heinemeier Hansson' <david@heinemeierhansson.com>",
      "url": "http://example.com/users/1-david.json"
    }
  ```

  Note the nested object and the use of associations (`@message.creator`) and
  view helpers (`url_for`).

## Phase 1: Basic Jbuilder templates

Time to make some templates! Start by making a `show.json.jbuilder` view for
your guest resource. Include the guest's `name`, `age`, and `favorite color`
by specifying each field and its value, like so:

```ruby
json.name @guest.name
json.age ...
```

Make sure you don't include `created_at` or `updated_at`. Test your work by
starting the server and sending a request from Postman or visiting the url
directly (e.g., `api/guests/1`). You should see something with the following
format:

```json
{
  "name": "John Smith",
  "age": 45,
  "favorite_color": "Blue"
}
```

(If you don't have a JSON formatter installed in your browser, try [this
one][formatter-link].)

Once you have it working, go ahead and create an `index` view. For this, use
`json.array!` and pass `@guests`. Use a block to specify what you want to render
for each guest. This time, use `json.extract!` instead of specifying each
field individually. When you finish, navigate to the `index` route in your
browser. Now you should see:

```json
[
  {
  "name": "John Smith",
  "age": 45,
  "favorite_color": "Blue"
  },
  // ...
  {
  "name": "Job Bluth",
  "age": 41,
  "favorite_color": "Red"
  }
]
```

Since your two templates do very similar things, go ahead and refactor the
single guest details into a partial. The naming convention for the partial
itself is the same as for HTML views, i.e., __\_guest.json.jbuilder__. (As with
ERB partials, you should omit the `_` when invoking the partial.) Make sure to
include `api/` in your partial path.

Remember that your partial should not rely on instance variables from a
controller but should instead receive any necessary variables as arguments; this
will enable it to work with views from various actions. If you have trouble with
the syntax, check the [Jbuilder docs][docs-link]. Test your partial to make sure
everything still renders correctly before moving on.

Next, add some associated data so that you can see gifts for individual guests,
but not when you're looking at all guests (this may be too much data). In your
`show` view, render a guest's gifts. Only include the `title`, `description`,
and `party`. This is the format you want to build:

```json
{
  "name": "John Smith",
  "age": 45,
  "favorite_color": "Blue",
  "gifts": [
    {
      "title": "A Bottle of Wine",
      "description": "This one is not so great.",
      "party": "Secret Santa Extravaganza"
    },
    {
      "title": "Another Bottle of Wine",
      "description": "This one is not so bad.",
      "party": "Secret Santa Extravaganza"
    }
  ]
}
```

**N.B.:** You might be tempted to add the gifts using something like
`json.array! @guest.gifts`, but this would try to convert the entire top-level
JSON object into an array and then throw an error because it didn't know how to
merge the other top-level guest information into that array. Instead, nest the
gifts array under a key of `gifts` using the following format:

```rb
json.key_name collection do |collection_item|
  # format data for each collection_item, will be element in array
end
```

When you finish, navigate to the `show` page to check your work.

Time to do some templates on your own! Make both a gift `show` and guest gift
`index` view. They should respectively produce the following responses:

```json
// Gift #1 show
{
  "title": "A Bottle of Wine",
  "description": "This one is not so great.",
  "guest": "John Smith",
  "party": "Secret Santa Extravaganza"
}

// Guest #1 gift index
[
  {
    "title": "A Bottle of Wine",
    "description": "This one is not so great.",
    "party": "Secret Santa Extravaganza"
  },
  {
    "title": "Another Bottle of Wine",
    "description": "This one is not so bad.",
    "party": "Secret Santa Extravaganza"
  }
]
```

Next, make the party `show` and `index` views. In the `index` view, show all
parties and include all of their guests:

```json
[
  {
    "name": "Secret Santa Extravaganza",
    "location": "Portland",
    "guests": [
      "John Smith",
      "Jane Doe",
      "Josh Brown",
      "Helen White",
      "Job Bluth"
    ]
  },
  {
    "name": "Charles' Christmas Party",
    "location": "San Francisco",
    "guests": [
      "Josh Brown",
      "Helen White"
    ]
  }
]
```

**HINT**: Options for building non-top-level arrays inside a Jbuilder template
include Jbuilder's `merge!` command, Rail's built-in association methods (such
as the [`resource_ids` getter][ids-getter]), and standard Ruby methods like
`map`.

In the `show` view, include not only all the guests, but all of the guests'
gifts as well:

```json
{
  "name": "Charles' Christmas Party",
  "location": "San Francisco",
  "guests": [
    {
      "name": "Josh Brown",
      "gifts": [
        "Basketball Tickets"
      ]
    },
    {
      "name": "Helen White",
      "gifts": []
    }
  ]
}
```

Remember that you only want to show a guest's gifts for the specified party!

Finally, modify your guest `index` view to show only guests who are between 40
and 50 years old. Normally you would do this kind of selection using Active
Record, but this gives you an opportunity to practice using Ruby in Jbuilder.

## Phase 2: Fixing N+1 Queries

In writing these views, you've probably generated some gnarly N+1 queries. Are
you calling `.gifts` for every guest in the parties show view? That's a query
for every guest! To fix this, remember your Active Record skills. `.includes`
pre-fetches whatever data you tell it to fetch. For example, in the
`PartiesController#show`, you could call `.includes(guests: [:gifts])`; then,
when you call `.gifts` on each guest in your Jbuilder template, you will use
that pre-fetched data and won't actually have to hit the database again. Go
ahead and fix that now.

Find any other N+1 queries you've made throughout and defeat them. **Hint:**
play around with your API in development and watch your server logs. Look for a
query followed by many repetitive queries. E.g., with the party `show` view,
before you fixed the N+1 query, your server logs would have looked something
like this:

```shell
Started GET "/api/parties/1" for ::1 at 2021-11-10 00:29:14 -0500
Processing by Api::PartiesController#show as JSON
  Parameters: {"id"=>"1"}
  Party Load (0.2ms)  SELECT "parties".* FROM "parties" WHERE "parties"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
  Rendering api/parties/show.json.jbuilder
  Rendered api/parties/_party.json.jbuilder (Duration: 0.1ms | Allocations: 18)
  Guest Load (0.6ms)  SELECT "guests".* FROM "guests" INNER JOIN "invitations" ON "guests"."id" = "invitations"."guest_id" WHERE "invitations"."party_id" = $1  [["party_id", 1]]
  Gift Load (0.2ms)  SELECT "gifts".* FROM "gifts" WHERE "gifts"."guest_id" = $1  [["guest_id", 1]]
  Gift Load (0.2ms)  SELECT "gifts".* FROM "gifts" WHERE "gifts"."guest_id" = $1  [["guest_id", 2]]
  Gift Load (5.9ms)  SELECT "gifts".* FROM "gifts" WHERE "gifts"."guest_id" = $1  [["guest_id", 3]]
  Gift Load (4.0ms)  SELECT "gifts".* FROM "gifts" WHERE "gifts"."guest_id" = $1  [["guest_id", 4]]
  Gift Load (4.4ms)  SELECT "gifts".* FROM "gifts" WHERE "gifts"."guest_id" = $1  [["guest_id", 5]]
  Rendered api/parties/show.json.jbuilder (Duration: 87.4ms | Allocations: 18963)
Completed 200 OK in 116ms (Views: 55.6ms | ActiveRecord: 41.9ms | Allocations: 23634)
```

See all those Gift Loads? Those are the N queries to accompany your 1 query for
the party and 1 query for the guests.

When it's fixed, it should look more like this:

```sh
Started GET "/api/parties/1" for ::1 at 2021-11-10 00:27:18 -0500
Processing by Api::PartiesController#show as JSON
  Parameters: {"id"=>"1"}
  Party Load (0.2ms)  SELECT "parties".* FROM "parties" WHERE "parties"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
  Invitation Load (2.5ms)  SELECT "invitations".* FROM "invitations" WHERE "invitations"."party_id" = $1  [["party_id", 1]]
  Guest Load (2.1ms)  SELECT "guests".* FROM "guests" WHERE "guests"."id" IN ($1, $2, $3, $4, $5)  [["id", 1], ["id", 2], ["id", 3], ["id", 4], ["id", 5]]
  Gift Load (42.1ms)  SELECT "gifts".* FROM "gifts" WHERE "gifts"."guest_id" IN ($1, $2, $3, $4, $5)  [["guest_id", 1], ["guest_id", 2], ["guest_id", 3], ["guest_id", 4], ["guest_id", 5]]
  Rendering api/parties/show.json.jbuilder
  Rendered api/parties/_party.json.jbuilder (Duration: 0.1ms | Allocations: 13)
  Rendered api/parties/show.json.jbuilder (Duration: 0.7ms | Allocations: 495)
Completed 200 OK in 66ms (Views: 1.3ms | ActiveRecord: 46.9ms | Allocations: 4084)
```

Only one query per table! That's what you want to see. Notice the difference in
overall time: 66ms for the pre-fetched version vs. 116ms for the version with
multiple Gift Loads. And that's just for 5 guests. Imagine how that time
differential will balloon as a guest list grows into the hundreds...

### Phase 3: Building a Normalized State

The [Redux docs][normalized-state] identify four characteristics of a
_normalized state_:

> - Each type of data gets its own "table" in the state.
>
> - Each "data table" should store the individual items in an object, with the
>   IDs of the items as keys and the items themselves as the values.
>
> - Any references to individual items should be done by storing the item's ID.
>
> - Arrays of IDs should be used to indicate ordering.

You have learned a bit about the benefits of maintaining a normalized state.
Today, you will practice using Jbuilder to form responses that reflect a
normalized state. Returning data formatted in this way can help you keep your
reducers clean on the frontend of a full stack application.

According to the definition above, a _normalized_ response for a guest's gift
`index` might look like this:

```json
{
  "gifts": {
    "1": {
      "id": 1,
      "title": "A Bottle of Wine",
      "description": "This one is not so great.",
      "partyId": 1,
      "guestId": 1
    },
    "2": {
      "id": 2,
      "title": "Another Bottle of Wine",
      "description": "This one is not so bad.",
      "partyId": 1,
      "guestId": 1
    }
  }
}
```

As this example shows, each gift is represented by a key-value pair where the
key is the gift's `id` and the value is the gift's information. All of the gifts
are then nested under a key of `gifts`.

In the gifts controller, change the `index` action to render `:normalized_index`
instead of `:index`, then create a corresponding Jbuilder file in __views__.
Inside the file, write a Jbuilder template to build the normalized gift state
shown above. Test your work in Postman or the browser.

Now do the same for the guest `index`. When you finish, your `normalized_index`
of guests should look like this:

```json
{
  "guests": {
    "1": {
      "id": 1,
      "name": "John Smith",
      "age": 45,
      "favorite_color": "Blue",
      "gift_ids": [
        1,
        2
      ]
    },
    "2": {
      "id": 2,
      "name": "Jane Doe",
      "age": 47,
      "favorite_color": "Green",
      "gift_ids": [
        3
      ]
    },
    "3": {
      "id": 3,
      "name": "Josh Brown",
      "age": 22,
      "favorite_color": "Brown",
      "gift_ids": [
        4,
        5
      ]
    },
    "4": {
      "id": 4,
      "name": "Helen White",
      "age": 25,
      "favorite_color": "White",
      "gift_ids": [
        6
      ]
    },
    "5": {
      "id": 5,
      "name": "Job Bluth",
      "age": 41,
      "favorite_color": "Red",
      "gift_ids": []
    }
  }
}
```

Note that if you were actually storing this information in your frontend state,
`gift_ids` would be superfluous: you already have an association of each gift
with its guest through the gift's `guest_id` field. You could accordingly
construct a list of a guest's gifts in the frontend whenever you need it simply
by using a selector, e.g.,

```js
let guestId = 3
let gifts = Object.values(state.gifts)
                  .filter(gift => gift.guestId === guestId)
```

Since storing an array of `gift_ids` under each guest would result in the
guest-gift association's being stored in two separate places, it would also
complicate deletions and risk the data getting out of sync. For these reasons, a
normalized state will often store a foreign key but forego storing the ids of
the `has_many` side of the association. (You are including such id arrays in
this project for the practice in building Jbuilder templates.)

Finally, create a normalized response for a party's `show` page. You should have
top-level keys for `parties`, `guests`, and `gifts`. Go ahead and include arrays
for the `guest_ids` and `gift_ids` under the individual `party`. It should look
like this:

```json
{
  "parties": {
    "2": {
      "id": 2,
      "name": "Charles' Christmas Party",
      "location": "San Francisco",
      "guest_ids": [
        3,
        4
      ],
      "gift_ids": [
        5
      ]
    }
  },
  "guests": {
    "3": {
      "id": 3,
      "name": "Josh Brown",
      "age": 22,
      "favorite_color": "Brown",
      "gift_ids": [
        4,
        5
      ]
    },
    "4": {
      "id": 4,
      "name": "Helen White",
      "age": 25,
      "favorite_color": "White",
      "gift_ids": [
        6
      ]
    }
  },
  "gifts": {
    "5": {
      "id": 5,
      "title": "Basketball Tickets",
      "description": "The team's going 82-0 this year.",
      "partyId": 2,
      "guestId": 3
    }
  }
}
```

Two more things to consider. First, if the requesting frontend is written in
JavaScript, then it will likely expect its keys to be in camelCase instead of
the snake_case used by Rails. (See, e.g., `favorite_color` and `gift_ids`
above.) You can address this programmatically by telling Jbuilder always to
convert keys to camelCase. To do this, simply include the following lines at the
bottom of __config/environment.rb__:

```rb
Jbuilder.key_format camelize: :lower
# Jbuilder.deep_format_keys true
```

(Depending on how you have set up your partials and templates, the second line
may or may not be necessary. You will want to include it if you need the key
transformation applied to hashes passed to `set!`, `array!`, `merge!`, and the
like. You can also set it locally by calling `json.deep_format_keys! true` in
the template where you need it.)

You've changed one of your server's environment files, so you will need to
restart the server to see the changes. Once it is restarted, navigate to the
guest `index` or a party's `show` page: the keys should all be camelCase now!

Second, your normalized templates have probably introduced another N+1 query.
Test those three templates again, checking your server logs as you do. Fix any
N+1 issues that you find.

## What you've learned

In this project, you have learned how to use Jbuilder to select and format the
data that Rails sends to the frontend. Along the way, you also practiced
eliminating N+1 queries and building a normalized state.

[`shallow`]: https://guides.rubyonrails.org/routing.html#shallow-nesting
[formatter-link]: https://chrome.google.com/webstore/detail/json-formatter/bcjindcccaagfpapjjmafapmmgkkhgoa?hl=en
[docs-link]: https://github.com/rails/jbuilder
[ids-getter]: https://guides.rubyonrails.org/association_basics.html#methods-added-by-has-many-collection-singular-ids
[normalized-state]: https://redux.js.org/usage/structuring-reducers/normalizing-state-shape#designing-a-normalized-state