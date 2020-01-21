# NatureCounts Mobile App API Verson 1.0 #

This set of entrypoints is designed to serve the needs of NatureCounts mobile app.

All queries are over HTTPS GET requests. Query responses are JSON object, mostly structured as a 'data frame': attribute names are the data
field names, and attribute values are vectors of field values.

A basic data query looks like this (sandbox server is the only current host supporting the api):

> https://sandbox.birdscanada.org/api/....



## Registration, Authentication ##

Where user identification is required, queries must include a token parameter.
The token is obtained via the authentication mechanism desribed below.





### User Registration ###

Currently by web

https://www.birdscanada.org/birdmon/default/register.jsp



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

Note that the api_version attribute can be used to validate future versions of the Mobile App.


### User Profile ###


Returns the profile asocaited with the token.

> /api/mobile/user?token=asdfasdf

Authenticated: Yes

**Return JSON attributes:**

| Attribute | Notes |
| --------- | ----- |
| login_name | The login name |
| last_name | The surname |
| first_name | The firstname |
| email | The email address |




### Data and API Version ###

Returns api and data version attributes.

> /api/mobile/dataVersion

Authenticated: No

**Return JSON attributes:**

| Attribute | Notes |
| --------- | ----- |
| api_version | The API version will normally not change, but may be used in future to signal stale Mobile App version |
| data_version | If this attribute changes, all cached Reference data should be updated before the user interacts with the App |







### Project Registration / De-registration ###

Not available yet




## Reference Data ##


Reference data should be cached locally in the app, with a provision for periodic updates (weekely, etc?) and
for a complete forced refresh. The `data_version` parameter returned by the `/api/mobile/dataVersion` entrypint should be
checked against a stored value: if this value changes, a complete refresh of Reference Data is in order.

Reference data results are returned as JSON objects, mostly structured as 'data frames'. The query result columns (fields) occur
as attribute names within the JSON object, and the values of each attribute is a vector (JSON array) of values. The
query records can be constructed by combining the vector values across all attributes in the JSON object.

All Reference Data queries require a user authentication token parameter, and additional
filtering parameters are used on some of the queries (see below).




### Projects ###

Return a list of projects:

> /api/mobile/projects?token=asdfasdf&lang=EN

Authenticated: Yes


### Provinces ###

Returns a list of Canadian Provinces:

> /api/mobile/provinces?token=asdfasdf&lang=EN

Authenticated: Yes




### Projects Provinces ###

Return a list of provinces associated with projects:

> /api/mobile/projectsProvinces?token=asdfasdf&lang=EN

Authenticated: Yes



### Regions ###

Returns a list of region:

> /api/mobile/regions?token=asdfasdf&lang=EN

Authenticated: Yes



### User Projects ###


Returns the projects to which the user is registered, along with a status.

> /api/mobile/userProjects?token-asdfasdf

Authenticated: Yes





### Species Codes ###

Returns a list of EBIRD Species Codes:

> /api/mobile/speciesCodes?token=asdfasdf&lang=EN

Authenticated: Yes



### Breeding Codes ###

Returns a list of breeding evidence codes:

> /api/mobile/breedingCodes?token=asdfasdf&lang=EN

Authenticated: Yes



### Species ###

Returns a list of species:

> /api/mobile/species?token=asdfasdf&lang=EN

Authenticated: Yes



### Species Groups ###

Returns a list of species groups:

> /api/mobile/speciesGroups?token=asdfasdf&lang=EN

Authenticated: Yes


### Species EBIRD Codes ###

Returns a list of codes from the EBIRD checklist:

> /api/mobile/speciesEbird?token=asdfasdf&lang=EN

Authenticated: Yes





### Species Invalid Breeding Evidence Codes ###

Returns a list of invalid breeding evidence codes for species:

> /api/mobile/speciesInvalidBreedingEvidence?token=asdfasdf&lang=EN

Authenticated: Yes






### Protocols ###

Returns a list of Protocols:

> /api/mobile/breedingCodes?token=asdfasdf&lang=EN

Authenticated: No







