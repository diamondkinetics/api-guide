# Diamond Kinetics Java API Guide
A guide for utilizing and implementing functionality in the Diamond Kinetics Java API.
Please feel free to create pull requests in order to make updates.

---

## Table of Contents
1. Resources
2. Actions
3. Filtering
4. Sorting
5. Searching
6. Aliases
7. Limiting Fields
8. Pagination
9. Relationship Auto Loading
10. Requesting Multiple Objects
11. Versioning
12. Error Handling
13. White Space and Compression

---

## Resources
Resources in the Diamond Kinetics API should always be represented as nouns. When used in an endpoint,
a resource should be plural.
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

## Filtering
Use a unique query parameter in the endpoint URL for each field that implements filtering.
### Examples
- To only list flagged pitching sessions, we use: `GET /pitchingSessions?flagged=true`
- To only list non-deleted pitching sessions, we use: `GET /pitchingSessions?deleted=false`

## Sorting
To sort, we can use a generic parameter such as `sort` for describing any sorting rules on an endpoint.
Sorting with multiple criteria can be accomplished using a list of comma separated rules.
### Examples
- To list pitches from a pitching session sorted by release speed in descending order, we use `GET /pitchingSessions/:uuid/pitches?sort=-releaseSpeed`
- To list pitches from a pitching session sorted by release speed in ascending order, we use `GET /pitchingSessions/:uuid/pitches?sort=releaseSpeed`

Notice that sorting in a descending order, we use a `-` before the property name, and for ascending we just use the property name.

## Searching
If we wanted to essentially combine filtering and sorting in order to create a search for resources, we could just combine query parameters.
### Examples
- To sort and filter pitches from a pitching session, we use `GET /pitchingSessions/:uuid/pitches?flagged=true&sort=-releaseSpeed`
  - This would retrieve flagged pitches and sort them by release speed in descending order.

## Aliases
Packaging sets of conditions into easily accessible paths can make API consumption a bit easier. If there are parameters that are consistently used on a specific endpoint, it may make sense to create an alias for it. These will more often than not be in the form of a `GET` request.
### Examples
- To retrieve our most recent pitching session, create an alias such as: `GET /pitchingSessions/latest`
- To retrieve a group's current join requests, create an alias such as: `GET /groups/:uuid/joinRequests`

## Limiting Fields
Consumers of the API don't always need the full representation of a resource. The ability to select and choose returned fields can let the consumer minimize network traffic and speed up their API usage. To accomplish this, we use a `fields` query parameter that takes a comma separated list of fields to include in the response.
### Examples
- To only retrieve the `uuid` and `releaseSpeed` fields when retrieving pitches for a particular pitching session, we use: `GET /pitchingSessions/:uuid/pitches?fields=uuid,releaseSpeed`
- To only retrieve the `uuid`, `sessionDate`, and `flagged` fields when retrieving a list of the current user's pitching sessions, we use: `GET /pitchingSessions?fields=uuid,sessionDate,flagged`

## Pagination
Including any pagination details for a list of resources should always be accomplished using the `Link` header in the API response. This can return a set of ready-made links so an API consumer does not have to worry about constructing them on their own.
### Examples
- For an API response from a request to list a current user's pitching sessions, the `Link` header for the first page looks like:
```
Link: <https://api.diamondkinetics.com/pitchingSessions?page=2&perPage=10>; rel="next", <https://api.diamondkinetics.com/pitchingSessions?page=10&perPage=10>; rel="last"
```
- The second page `Link` header looks like:
```
Link: <https://api.diamondkinetics.com/pitchingSessions?page=1&perPage=10>; rel="previous", <https://api.diamondkinetics.com/pitchingSessions?page=3&perPage=10>; rel="next", <https://api.diamondkinetics.com/pitchingSessions?page=10&perPage=10>; rel="last"
```

## Relationship Auto Loading
It's common for an API consumer to need data related to a requested resource. The efficient way to accomplish this would be to allow related data to be returned along with the original requested resource. A good way to include related resources would be to use a query parameter named something along the lines of *embed*. This query parameter utilizes a comma separated list for it's value which represents a list of fields to be embedded with the requested resource. Dot-notation can be used to reference sub-fields.

**NOTE:** Be wary of N+1 select scenarios when implementing this approach.

### Examples
- Returning a user along with a pitching session can be accomplished with: `GET /pitchingSessions/:uuid?embed=user`
- Returning just a user's first name, last name, and uuid along with the pitching session can be accomplished with `GET /pitchingSessions/:uuid?embed=user.firstName,user.lastName,user.uuid`

## Requesting Multiple Objects

## Versioning

## Error Handling

## White Space and Compression
