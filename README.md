Reproduction repository for https://github.com/strapi/strapi/issues/9961

To reproduce:

- Clone this repository
- In the root directory, run `docker-compose up`
- Once the setup is done and the server started, go to the admin to create a user: [http://localhost:1337/admin](http://localhost:1337/admin)
- Change the permissions so anonymous users can query users, in http://localhost:1337/admin/settings/users-permissions/roles/2, under "Users-permission", enable "find" in "user" group and save.
- Create a recipe: http://localhost:1337/admin/plugins/content-manager/collectionType/application::recipe.recipe
- Publish the recipe
- Create a user: http://localhost:1337/admin/plugins/content-manager/collectionType/plugins::users-permissions.user
- Add a "LastViewedRecipes" entry to the user and select the previously created recipe

Now query the user last viewed recipes through the [GraphQL playground](http://localhost:1337/graphql):

```gql
query user($id: ID!) {
  user(id: $id) {
    lastViewedRecipes {
      recipe {
        id
      }
    }
  }
}
```

With these params: `{ "id": 1 }`.

It should work and return a response similar to this:

```json
{
  "data": {
    "user": {
      "lastViewedRecipes": [
        {
          "recipe": {
            "id": "2"
          }
        }
      ]
    }
  }
}
```

Now go to [the recipes collection in the admin](http://localhost:1337/admin/plugins/content-manager/collectionType/application::recipe.recipe) and delete the recipe that was linked.

Run the GraphQL query again. The query now throws:

```json
{
  "errors": [
    {
      "message": "select distinct \"recipes\".* from \"recipes\" where \"recipes\".\"id\" = $1 and (\"recipes\".\"published_at\" is not null) limit $2 - invalid input syntax for type integer: \"{}\"",
      "locations": [
        {
          "line": 4,
          "column": 7
        }
      ],
      "path": ["user", "lastViewedRecipes", 0, "recipe"],
      "extensions": {
        "code": "INTERNAL_SERVER_ERROR",
        "exception": {
          "length": 103,
          "name": "error",
          "severity": "ERROR",
          "code": "22P02",
          "file": "numutils.c",
          "line": "259",
          "routine": "pg_strtoint32",
          "stacktrace": [
            "error: select distinct \"recipes\".* from \"recipes\" where \"recipes\".\"id\" = $1 and (\"recipes\".\"published_at\" is not null) limit $2 - invalid input syntax for type integer: \"{}\"",
            "    at Parser.parseErrorMessage (/srv/app/node_modules/pg-protocol/dist/parser.js:278:15)",
            "    at Parser.handlePacket (/srv/app/node_modules/pg-protocol/dist/parser.js:126:29)",
            "    at Parser.parse (/srv/app/node_modules/pg-protocol/dist/parser.js:39:38)",
            "    at Socket.<anonymous> (/srv/app/node_modules/pg-protocol/dist/index.js:10:42)",
            "    at Socket.emit (events.js:314:20)",
            "    at addChunk (_stream_readable.js:297:12)",
            "    at readableAddChunk (_stream_readable.js:272:9)",
            "    at Socket.Readable.push (_stream_readable.js:213:10)",
            "    at TCP.onStreamRead (internal/stream_base_commons.js:188:23)",
            "From previous event:",
            "    at processImmediate (internal/timers.js:461:21)",
            "From previous event:",
            "    at Sync.<anonymous> (/srv/app/node_modules/bookshelf/lib/sync.js:204:8)",
            "From previous event:",
            "    at Child.<anonymous> (/srv/app/node_modules/bookshelf/lib/collection.js:160:12)",
            "From previous event:",
            "    at Child.fetchAll (/srv/app/node_modules/bookshelf/lib/model.js:878:10)",
            "    at find (/srv/app/node_modules/strapi-connector-bookshelf/lib/queries.js:79:8)",
            "    at Object.findOne (/srv/app/node_modules/strapi-connector-bookshelf/lib/queries.js:66:27)",
            "    at fn (/srv/app/node_modules/strapi-database/lib/queries/helpers.js:31:54)",
            "    at Object.findOne (/srv/app/node_modules/strapi-database/lib/queries/helpers.js:15:24)",
            "    at processTicksAndRejections (internal/process/task_queues.js:97:5)"
          ]
        }
      }
    }
  ],
  "data": {
    "user": {
      "lastViewedRecipes": [
        {
          "recipe": null
        }
      ]
    }
  }
}
```
