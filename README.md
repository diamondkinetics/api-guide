# Diamond Kinetics Java API Guide
A guide for utilizing and implementing functionality in the Diamond Kinetics Java API.
Please feel free to create pull requests in order to make updates.

---

## Table of Contents
- [Concepts](#concepts)
- [Naming Conventions](#naming-conventions)
- [Actions](#actions)
- [Non-standard Actions](#non-standard-actions)
- [Filtering](#filtering)
- [Sorting](#sorting)
- [Searching](#searching)
- [Aliases](#aliases)
- [Limiting Fields](#limiting-fields)
- [Pagination](#pagination)
- [Relationship Auto Loading](#relationship-auto-loading)
- [Batch Operations](#batch-operations)
- [Versioning](#versioning)
- [Error Handling](#error-handling)
- [White Space and Compression](#white-space-and-compression)
- [References](#references)

---

## Concepts
Concepts that are helpful to know in order to comprehend and use the API.

### Response Codes
| Response Code | Meaning      | Diamond Kinetics Specific |
| ------------- | -------      | ------------------------- |
| 200           | OK           | Successful request & response |
| 201           | CREATED      | Successful creation |
| 400           | BAD REQUEST  | Validation failure |
| 401           | UNAUTHORIZED | There isn't enough information in the request to authenticate the user |
| 403           | FORBIDDEN    | The authenticated user is unable to perform the requested action |
| 409           | CONFLICT     | The object you are trying to create already exists. This is always for UUID conflict. |

### Identifiers
All entities in the Diamond Kinetics API will have *fixed* UUIDs to reference externally. This is generally a good practice (citation needed) and limits leaking any information about row numbers/count of our tables.

### Data Synchronization
sensorCreated
clientCreated
serverCreated

### Authorization
Any user in the system is authorized to see the sessions of any other user if they fall into one of the following categories.

#### UserToUserConnection
Any two connected users can see each others sessions.

#### Groups
Any admin of a group can see all sessions of any other user in the group.

#### Group role permissions
Session View Role & DATA_ACCESSOR role description here.

## Naming Conventions
Resources in the Diamond Kinetics API should always be represented as nouns. When used in an endpoint,
a resource should be plural.

All such resources should be represented in camelCase.
### Examples
- To represent pitching session resources, we use: `/pitchingSessions`
- To represent batting session resources, we use: `/battingSessions`

## Actions
We perform actions on resources, such as create, read, update, and delete (usually via an HTTP request type).
### Examples
- To create a pitching session, we use: `POST /pitchingSessions`
- To get a list of pitching sessions for the current user, we use: `GET /pitchingSessions`
- To get a specific pitching session, we use: `GET /pitchingSessions/:uuid`
  - `PUT` or `UPDATE` can also be used in order to update or delete a specific resource as well.

## Non-standard Actions
When it comes to dealing with non-standard actions, we have a few choices available to us for URL mapping.
[<sup>[1]</sup>](https://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api#restful)
- Restructure the action to appear like a field of a resource: `PUT /bullpens/:uuid/complete`
- Treat the action like a sub-resource: `PUT /pitchingSessions/:uuid/flag` and `DELETE /pitchingSessions/:uuid/flag`
[<sup>[5]</sup>](https://developer.github.com/v3/gists/#star-a-gist)
- In the event there is no way to map an action to a sensible RESTful structure, using a generic endpoint is alright.
  - A multi-resource search can use something such as `GET /search` 

## Filtering
Use a unique query parameter in the endpoint URL for each field that implements filtering.
[<sup>[1]</sup>](https://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api#advanced-queries)
### Examples
- To only list flagged pitching sessions, we use: `GET /pitchingSessions?flagged=true`
- To only list non-deleted pitching sessions, we use: `GET /pitchingSessions?deleted=false`

## Sorting
To sort, we can use a generic parameter such as `sort` for describing any sorting rules on an endpoint.
Sorting with multiple criteria can be accomplished using a list of comma separated rules.
[<sup>[1]</sup>](https://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api#advanced-queries)
### Examples
- To list pitches from a pitching session sorted by release speed in descending order, we use
`GET /pitchingSessions/:uuid/pitches?sort=-releaseSpeed`
- To list pitches from a pitching session sorted by release speed in ascending order, we use
`GET /pitchingSessions/:uuid/pitches?sort=releaseSpeed`

Notice that sorting in a descending order, we use a `-` before the property name, and for ascending we just use the
property name.

## Searching
If we wanted to essentially combine filtering and sorting in order to create a search for resources, we could just
combine query parameters.
[<sup>[1]</sup>](https://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api#advanced-queries)
### Examples
- To sort and filter pitches from a pitching session, we use
`GET /pitchingSessions/:uuid/pitches?flagged=true&sort=-releaseSpeed`
  - This would retrieve flagged pitches and sort them by release speed in descending order.

## Aliases
Packaging sets of conditions into easily accessible paths can make API consumption a bit easier. If there are parameters
that are consistently used on a specific endpoint, it may make sense to create an alias for it. These will more often
than not be in the form of a `GET` request.
### Examples
- To retrieve our most recent pitching session, create an alias such as: `GET /pitchingSessions/latest`
- To retrieve a group's current join requests, create an alias such as: `GET /groups/:uuid/joinRequests`

## Limiting Fields
Consumers of the API don't always need the full representation of a resource. The ability to select and choose returned
fields can let the consumer minimize network traffic and speed up their API usage. To accomplish this, we use a `fields`
query parameter that takes a comma separated list of fields to include in the response.
### Examples
- To only retrieve the `uuid` and `releaseSpeed` fields when retrieving pitches for a particular pitching session, we
use: `GET /pitchingSessions/:uuid/pitches?fields=uuid,releaseSpeed`
- To only retrieve the `uuid`, `sessionDate`, and `flagged` fields when retrieving a list of the current user's pitching
sessions, we use: `GET /pitchingSessions?fields=uuid,sessionDate,flagged`

## Pagination
Including any pagination details for a list of resources should always be accomplished using the `Link` header in the
API response. This can return a set of ready-made links so an API consumer does not have to worry about constructing
them on their own.
[<sup>[1]</sup>](https://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api#pagination)
### Examples
- For an API response from a request to list a current user's pitching sessions, the `Link` header for the first page
looks like:
```
Link: <https://api.diamondkinetics.com/pitchingSessions?page=2&perPage=10>;
rel="next", <https://api.diamondkinetics.com/pitchingSessions?page=10&perPage=10>; rel="last"
```
- The second page `Link` header looks like:
```
Link: <https://api.diamondkinetics.com/pitchingSessions?page=1&perPage=10>; rel="previous",
<https://api.diamondkinetics.com/pitchingSessions?page=3&perPage=10>; rel="next",
<https://api.diamondkinetics.com/pitchingSessions?page=10&perPage=10>; rel="last"
```

## Relationship Auto Loading
It's common for an API consumer to need data related to a requested resource. The efficient way to accomplish this would
be to allow related data to be returned along with the original requested resource. A good way to include related
resources would be to use a query parameter named something along the lines of *embed*. This query parameter utilizes a
comma separated list for it's value which represents a list of fields to be embedded with the requested resource.
Dot-notation can be used to reference sub-fields.

**NOTE:** Be wary of N+1 select scenarios when implementing this approach.

### Examples
- Returning a user along with a pitching session can be accomplished with: `GET /pitchingSessions/:uuid?embed=user`
- Returning just a user's first name, last name, and uuid along with the pitching session can be accomplished with
`GET /pitchingSessions/:uuid?embed=user.firstName,user.lastName,user.uuid`

### Proposals
- When a single related resource is not auto-loaded, we will supply the `UUID` for it. Such as `userUuid` for a `User` resource.
- When a set of related resources are not auto-loaded, we will provide metadata about the set such as `count`, `deletedCount`, and `maxLastUpdated`.

- If we aren't auto-loading related entities, do we include metadata about the related entity/entities.
  - For a single entity, we would just supply the UUID such as `userUuid`.
  - For a set of entities, we will have an object that contains metadata such as `count`, `deletedCount`, and `maxLastUpdated`. For example, a batting session structure without swings loaded:
  ```
  {
    uuid: string,
    created: string,
    lastUpdated: string,
    deleted: boolean,
    swings: {
        count: integer,
        deletedCount: integer,
        maxLastUpdated: string
    }
  }
  ```
  - If resources are loaded, they will be contained in an array named `data`. For example, a batting session with swings
  loaded.
  ```
  {
      uuid: string,
      created: string,
      lastUpdated: string,
      deleted: boolean,
      swings: {
          data: array,
          count: integer,
          deletedCount: integer,
          maxLastUpdated: string
      }
  }
  ```

## Batch Operations
Sometimes we want to perform a batch operation such as `GET` multiple users, `GET` batting sessions for multiple users,
or `DELETE` multiple swings. To accomplish this, rather than specifying a single `UUID` like we would when fetching a 
specific resource, we need to specify a list of comma separated `UUID` values.
### Examples
- Requesting multiple users: `GET /users/:uuids`
- Requesting batting sessions for multiple users: `GET /users/:uuids/battingSessions`
- Deleting multiple swings: `DELETE /swings/:uuids`

## Versioning
We declare the major version of the API to use in the URL
### Examples
- User profile version 2: `GET /v2/users/:uuid/profile`
- Batting sessions version 3: `GET /v3/battingSessions`

#### Future considerations
- Documentation
  - Better Swagger front-end
    - [redoc](https://github.com/Redocly/redoc)
    - [Swagger UI](https://swagger.io/tools/swagger-ui/)
    - [LucyBot](https://lucybot.com/)
- Version fallback for unimplemented next version URLs
  - For discussion: What happens when we fallback to a version with DRASTICALLY different formats?
- Date-based sub-versions that can be specified using custom HTTP request headers.

## Error Handling
An API should provide a useful error message in a known consumable format such as JSON. The error body should provide
at least the HTTP status code, along with a message. A more detailed description of the error can be included as needed.
### Examples
- Conflict, resource already exists.
```
{
    "code": 409,
    "message": "That thing already exists",
    "description": "You can't make this, it's already a thing."
}
```

- Field specific validation errors should be included for `PUT`, `PATCH`, and `POST` requests. This should be modeled
using a fixed top-level error code for validation failures and providing the detailed errors in an additional
***errors*** field.
```
{
    "code": 400,
    "message": "Validation failed",
    "errors": [
        {
            "code": 123,
            "field": "firstName",
            "message": "First name is required"
        },
        {
            "code": 124,
            "field": "nickname",
            "message": "Nickname cannot contain spaces"
        }
    ]
}
```


## White Space and Compression
Pretty print responses with the extra whitespace. Removing whitespace does not save much bandwidth, usually around 8.5%.
With GZip compression enabled, removing all whitespace only saves around 2.5%. GZipping in itself saves around 60% in
bandwidth savings. Pretty printing with GZip enabled is preferred.

## References
1. [Vinay Sanhi - Best Practices for Designing a Pragmatic RESTful API](https://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api)
2. [O'Reilly - How a RESTful API represents resources](https://www.oreilly.com/ideas/how-a-restful-api-represents-resources)
3. [O'Reilly - How to design a RESTful API architecture from a human-language spec](https://www.oreilly.com/learning/how-to-design-a-restful-api-architecture-from-a-human-language-spec)
4. [Google - API Design Guide](https://cloud.google.com/apis/design/)
5. [GitHub API - Star a Gist](https://developer.github.com/v3/gists/#star-a-gist)