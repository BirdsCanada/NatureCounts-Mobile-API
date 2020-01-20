# NatureCounts Mobile App API Verson 1.0 #

This set of entrypoints is designed to serve the needs of NatureCounts mobile app.

All queries are over HTTPS GET requests. Query responses are JSON object, mostly structured as a 'data frame': attribute names are the data
field names, and attribute values are vectors of field values.

## Registration, Authentication ##

Where authemtication is needed, queries must include a token parameter. The token is obtained via the authentication mechanism built for the 
NatureCounts R-client API.

### Authentication ###

The API controls collection data access using a token that is generated when a user authenticates.
The token has a limited life span (currently 20 days), so the client application should require user authentication 
for each new session. Authentication is based on [NatureCounts login](https://www.birdscanada.org/birdmon/default/register.jsp).
Registration is free.

A user may hold multiple valid tokens at a time, allowing them to work from more than one session (or device) at once.

This entrypoint requires a login username and password, and if valid, returns a JSON object carrying
a token with a 20 day validity period, as well as the current api_version designation (String).
The client application can then present the token as user credentials in the data access entry points, where applicable.

Required parameter: **username** - the account username

Required parameter: **password** - the account password

>**Example URL:** /api/data/authenticate?username=asdfasdf&password=qwerty

>**Example Response:** {"token":"1234567890qwerty,"api_version":"2019.01"}


### User Registration ###



### User Signin ###



### Project Registration / De-registration ###




## Reference Data ##


Reference data should be cached locally in the app, with a provision for periodic updates (weekely, etc?) and
for a complete forced refresh. The dataVersion parameter returned by the first entry point below should be
checked against a stpred value: if this parameter value cahnges, a complete refresh of Reference Data is in order.

All Reference data results are returned as JSON objects, structured as 'data frames'. The query result columns (fields) occur
as attribute names within the JSON object, and the values of each attribute is a vector (JSON array) of values. The
query records can be constructed by combining the vectore values across all attributes in the JSON object.


A basic data query looks like this (sandbox server is the only current host supporting the api):

> https://sandbox.birdscanada.org/api/....

Certain Reference Data queries require a user authentication token parameter, and additional
filtering parameters are used on some of the queries (see below).



### Data and API Version ###

Returns appi and data version attributes.

> /api/mobile/dataVersion

Authenticated: No

**Return JSON attributes:**

| Attribute | Notes |
| --------- | ----- |
| api_version | The API version will normaly not change, but may be used in future to signal stale Mobile App version |
| data_version | If this attribute changes, all cached Reference data should be updated before the use interacts with the App |





### Breeding Codes ###

Returns a list of breeding evidence codes.

> /api/metadata/breeding_codes

Authenticated: No


### Provinces ###

Returns a list of Canadian Provinces:

> /api/metadata/provinces

Authenticated: No

