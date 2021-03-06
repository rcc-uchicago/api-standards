# RCC Web Service Standards

This document provides guidelines and examples for Web APIs produced by the RCC, encouraging consistency, maintainability, and best practices across applications. RCC Web Services aim to balance a truly RESTful API interface with a positive developer experience (DX).

* [Basics](#basics)
* [Example](#example)
* [URLs](#urls)
* [HTTP Verbs](#http-verbs)
* [Responses](#responses)
* [Errors](#errors)
* [Pagination](#pagination)
* [More Examples](#more-examples)
* [Tutorials](#tutorials)
* [See Also](#see-also)


## Basics

* Your API should be designed around the idea of [resources](http://www.crummy.com/writing/RESTful-Web-Services/html/#calibre_link-1637).

* Resources represent **things** (nouns) and not behaviors (verbs).

* A URL identifies a resource.

* Each resource should have a canonical/unique URL (`/api/v2/users/bill`, `/api/v2/users/mary`).

* When identifying a resource with a URL, use plural nouns only for consistency (no singular nouns).

* HTTP methods (`GET`, `POST`, `PUT`, etc.) represent behaviors on your resources.
They indicate the type of action to be taken on a resource.

* Instance resources (`/api/users/bob`) should be represented as a child of some parent collection resource (`/api/users`).

* Collections are themselves resources with their own properties.

* Collections should support both **create** and **query** requests.

* Instance resources should support **read**, **update**, and **delete** operations.

* Return all resource properties in the return payload.

* Put the version number of the API in the base of the URL: `http://ws.rcc.uchicago.edu/api/v1/...`

* Allow users to request resource representations in particular formats, e.g.:
  * `/api/v1/users.json`
  * `/api/v1/users.html`
  * `/api/v1/users.csv`

* You shouldn’t need to go deeper than `resource/identifier/resource`.

* Info included in the header of a request for a resource (such as authentication info) shouldn't generally affect how that resource is represented in the response. Different resource representations should only result from requesting different URLs.


## Example

The HTTP verb (method) specified in a request indicates the type of action to be taken on a resource.  Below is an example of how the standard HTTP methods map to CRUD (create, read, update, delete) operations in a particular context.


#### `/users`

We'll represent the set of all Acme users with a user collection resource (`/users`) and handle any incoming http requests to `http://acme.org/api/v2/users` on the basis of the request method:

* `GET` - return a list of URIs for the user instance resources in this collection.

* `PUT` - **replace** the entire collection with a new set of users.

* `POST` - **create** a new user and returning the new user's unique resource URI.

* `DELETE` - **delete** the collection of users.


#### `/users/{id}`

We'll represent an Acme user (e.g., "Bob") with a user instance resource (`/users/bob`) and handle any incoming http requests to `http://acme.org/api/v2/users/bob` on the basis of the request method:

* `GET` - return a representation of the user (e.g., a JSON-encoded data structure containing info about the user).

* `PUT` - **replace** the user at the specified URI or **create** if they don't yet exist. (Note that you typically want to create new users with a `POST` method on the collection.)

* `POST` - this method is not typically utilized on an instance resource, but if you're representing individual users as sub-collections (say, collections of individual responsibilities, each represented as instance resources) then you would handle the request by creating a new entry.  For example, if sending a `POST` request to `/users/bob` with info about Bob's responsibility to care for the chickens you might return and instance resource URI like `/users/bob/chickens`, etc.

* `DELETE` - **delete** the user.

In other words ...

|               | CREATE          | READ       | UPDATE      | DELETE   |
| ------------- | --------------- | ---------- | ----------- | -------- |
|               | `POST`          | `GET`      | `PUT`       | `DELETE` |
| `/users`      | create user     | list users | bulk update | delete all users! |
| `/users/{id}` | error           | show user  | if user exists, update; if not, error | delete user |


## URLs

Here are some additional examples of good URL design.

List of users returned in JSON format:

    GET http://ws.rcc.uchicago.edu/api/v1/users.json

Query string filtering:

    GET http://ws.rcc.uchicago.edu/api/v1/users.json?group=rcc&sort=usage

A single user in JSON format:

    GET http://ws.rcc.uchicago.edu/api/v1/users/{id}.json

All running jobs belonging to a user:

    GET http://ws.rcc.uchicago.edu/api/v1/users/{id}/jobs.html

All jobs returned in JSON format:

    GET http://ws.rcc.uchicago.edu/api/v1/users/{id}/jobs.json

Specify optional fields in a comma separated list:

    GET http://ws.rcc.uchicago.edu/api/v1/users/{id}.json?fields=title,group,quota

Add a new article to a particular magazine:

    POST http://ws.rcc.uchicago.edu/api/v1/magazines/1234/articles


## Responses

When repsonding to requests, avoid returning values in the keys of your
response payload.

For example, return this ...

    "tags": [
      {"id": "125", "name": "Environment"},
      {"id": "834", "name": "Water Quality"}
    ],

... instead of this ...

    "tags": [
      {"125": "Environment"},
      {"834": "Water Quality"}
    ],


## Errors

Error responses should always include one of the conventional HTTP status
codes, the three most common being ...

* `200 - OK` - success
* `400 - Bad Request` - failure due to client-side problem
* `500 - Internal Server Error` - failure due to server-side problem

An error response might additionally contain a message for the developer, a message for the end-user (when appropriate), an internal error code (corresponding to some specific internally determined ID), and links where developers can find more info. For example:

    {
      "status" : 400,
      "dev" : "Message for devs describing what went wrong.",
      "msg" : "Message that can be passed along to users.",
      "err" : "4444",
      "info" : "http://ws.rcc.uchicago.edu/dev/errors/4444"
    }


## Pagination

Collection resources can vary in size, and it's often useful to limit the
number of elements from the collection represented in the response.  If no
limit is specified in the request, you might want to return the response with a
default limit.

Use a default limit to number of users returned:

    https://acme.org/api/v2/users

Limit to 10 users:

    https://acme.org/api/v2/users?limit=10

Limit to 10 users, offset by 25 (i.e, users 26 to 36):

    https://acme.org/api/v2/users?limit=10&offset=25

In the above, `offset=25` means, ‘skip the first 25 records’.

Information about record limits and total available count should also be included in the response. For example:

    {
        "metadata": {
            "resultset": {
                "count": 227,
                "offset": 25,
                "limit": 25
            }
        },
        "results": []
    }


## More Examples

Below are some examples of requests and responses to those requests.


#### `GET /magazines`

For example, consider a `GET` request made to `http://ws.rcc.uchicago.edu/api/v1/magazines`. The response body might be ...

    {
        "metadata": {
            "resultset": {
                "count": 123,
                "offset": 0,
                "limit": 10
            }
        },
        "results": [
            {
                "id": "1234",
                "type": "magazine",
                "title": "Public Water Systems",
                "tags": [
                    {"id": "125", "name": "Environment"},
                    {"id": "834", "name": "Water Quality"}
                ],
                "created": "1231621302"
            },
            {
                "id": 2351,
                "type": "magazine",
                "title": "Public Schools",
                "tags": [
                    {"id": "125", "name": "Elementary"},
                    {"id": "834", "name": "Charter Schools"}
                ],
                "created": "126251302"
            }
            {
                "id": 2351,
                "type": "magazine",
                "title": "Public Schools",
                "tags": [
                    {"id": "125", "name": "Pre-school"},
                ],
                "created": "126251302"
            }
        ]
    }


#### `GET /magazines/{id}`

For example, consider a `GET` request made to `http://ws.rcc.uchicago.edu/api/v1/magazines/{id}.json`. The response body might be ...

    {
        "id": "1234",
        "type": "magazine",
        "title": "Public Water Systems",
        "tags": [
            {"id": "125", "name": "Environment"},
            {"id": "834", "name": "Water Quality"}
        ],
        "created": "1231621302"
    }


#### `POST /magazines/{id}/articles`

    [
        {
            "title": "Raising Revenue",
            "author_first_name": "Jane",
            "author_last_name": "Smith",
            "author_email": "jane.smith@ws.rcc.uchicago.edu",
            "year": "2012",
            "month": "August",
            "day": "18",
            "text": "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Etiam eget ante ut augue scelerisque ornare. Aliquam tempus rhoncus quam vel luctus. Sed scelerisque fermentum fringilla. Suspendisse tincidunt nisl a metus feugiat vitae vestibulum enim vulputate. Quisque vehicula dictum elit, vitae cursus libero auctor sed. Vestibulum fermentum elementum nunc. Proin aliquam erat in turpis vehicula sit amet tristique lorem blandit. Nam augue est, bibendum et ultrices non, interdum in est. Quisque gravida orci lobortis... "
        }
    ]


## Tutorials

There are numerous tutorials on the web that will walk you through building a
basic REST API.  If you're comfortable using [node](https://nodejs.org/), the
following two articles cover the basics using two widely-used node frameworks:

* [express](https://scotch.io/tutorials/build-a-restful-api-using-node-and-express-4) - utilizes Express 4 and MongoDB.

* [restify](https://stormpath.com/blog/build-api-restify-stormpath/) - utilizes
  Restify and implements an OAuth2 client credential workflow with a helper
  library, nicely overviewing how you might go about authenticating your API;
  also walks you through developing a simple client for the API.

If you'd rather develop your REST APIs using Go (recommended!), try:

* [json api](http://thenewstack.io/make-a-restful-json-api-go/) - nice
  step-by-step walkthrough.

* [how i start](https://howistart.org/posts/go/1) - works through building
  a backend service proving weather data.


## See Also

* [REST API Quick Reference](http://www.restapitutorial.com/) - nice reference with a summary a [basic tips](http://www.restapitutorial.com/lessons/restquicktips.html).

* [RESTful Web Services](http://www.crummy.com/writing/RESTful-Web-Services/html)
  * [Resource-Oriented Architecture](http://www.crummy.com/writing/RESTful-Web-Services/html/#calibre_link-33)
  * [Best Practices](http://www.crummy.com/writing/RESTful-Web-Services/html/#calibre_link-15)

* [HTTP API Design Guide](https://github.com/interagent/http-api-design#http-api-design-guide), describing current best practices in additional detail

* [Whitehouse API Standards](https://github.com/WhiteHouse/api-standards), from which this guide was originally forked



* [Designing HTTP Interfaces and RESTful Web Services](https://www.youtube.com/watch?v=zEyg0TnieLg)

* [API Facade Pattern](http://apigee.com/about/resources/ebooks/api-fa%C3%A7ade-pattern), by Brian Mulloy, Apigee

* [Web API Design](http://pages.apigee.com/web-api-design-ebook.html), by Brian Mulloy, Apigee
