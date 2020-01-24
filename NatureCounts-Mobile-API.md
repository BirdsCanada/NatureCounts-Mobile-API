# NatureCounts Mobile App API Verson 1.0 #

This set of entrypoints is designed to serve the needs of NatureCounts mobile app.

All queries are over HTTPS  as either GET or POST requests. Query responses are JSON object,
mostly structured as a 'data frame': attribute names are the data
field names, and attribute values are vectors of field values.

Use the following host to test against the current sandbox environment:

> https://sandbox.birdscanada.org



### Table of Contents ###

1. [Authentication and User Profile](#authentication-and-user-profile)
	1. [Authentication](#authentication)
	2. [User Profile](#user-profile)
	3. [Project Registration](#project-registration)
2. [API and Data Version](#api-and-data-version)
3. [Errors](#errors)
4. [Reference Data](#reference-data)
5. [Data Submission](#data-submission)


## Authentication and User Profile ##

User identification is accomplished by including a token as a request parameter. The
procedure to obtain a token is described below.

Note that user registration is supported through the website form available here:

[https://www.birdscanada.org/birdmon/default/register.jsp](https://www.birdscanada.org/birdmon/default/register.jsp)

Registration is free.


### Authentication ###

The API controls data access and submission using a token with a limited life span (currently 20 days).
Authentication is based on [NatureCounts login](https://www.birdscanada.org/birdmon/default/register.jsp).

A user may hold multiple valid tokens at a time, allowing them to work from more than one session (or device) at once.

This entrypoint requires a login username and password, and if valid, returns a JSON object carrying
a token with a 20 day validity period, as well as the current `api_version` designation (String).
The client application must then present the token as user credentials in subsequent entrypoints.

Required parameter: **username** - the account username

Required parameter: **password** - the account password

>**Example URL:** /api/data/authenticate?username=asdfasdf&password=qwerty

>**Example Response:** {"token":"1234567890qwerty,"api_version":"2019.01"}

Note that the api_version attribute can be used to validate future versions of the Mobile App.


### User Profile ###


Returns the profile associated with the token.

> /api/mobile/user?token=asdfasdf


**Return JSON attributes:**

| Attribute | Notes |
| --------- | ----- |
| login_name | The login name |
| last_name | The surname |
| first_name | The firstname |
| email | The email address |





### Project Registration ###

**Not available yet (2020-01-23)**





## API and Data Version ##

Returns api and data version attributes.

> /api/mobile/dataVersion

Authenticated: No

**Return JSON attributes:**

| Attribute | Notes |
| --------- | ----- |
| api_version | The API version will normally not change, but may be used in future to signal stale Mobile App version |
| data_version | If this attribute changes, all cached Reference data should be updated before the user interacts with the App |

The `api_version` value can be used in future to validate Mobile App verson compatibility.

The `data_version` parameter should be stored and then used to signal that a cached data refresh is required. If the `data_version`
has changed, then all reference data should be updated. This will not occur frequently.


## Errors ##

Errors generated in the API will result in a JSON encoded error structure,
returned with an appropriate HTTP error status. 

**Return JSON attributes:**

| Attribute | Notes |
| --------- | ----- |
| entryPoint | The api entry point that was queried |
| error | An http error code: also present as the response 'Status Code' |
| errorMsgs | An array of textual error message(s), detailing the condition |

You may retrieve the current error codes and messages with the following entrypoint.

> /api/mobile/errorCodes?token=asdfasdf



## Reference Data ##



Reference data should be cached locally in the app, with a provision for periodic updates (weekely, etc?) and
for a complete forced refresh. The `data_version` parameter returned by the `/api/mobile/dataVersion` entrypoint above should be
checked against a stored value: if this value changes, a complete refresh of Reference Data is in order.

Reference data are returned as JSON objects, structured as 'data frames'. The query result columns (fields) occur
as attribute names within the JSON object, and the values of each attribute is a vector (JSON array) of values. The
query records can be constructed by combining the vector values across all attributes in the JSON object.

**This format can be changed easily to instead return a more traditional array of JSON objects.**

All Reference Data queries require a user authentication `token` parameter, and may include a `lang` 
parameter. If the `lang` paramter is not provided, it will default to `EN`.

**Common Parameters**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| token | String | Yes | The authentication token |
| lang | String | No | A 2-letter language code, defaulting to EN |




### Projects ###

Return a list of all projects relevant to the mobile app:

> /api/mobile/projects?token=asdfasdf&lang=EN




### Provinces ###

Returns a list of Canadian Provinces with their 2-letter codes:

> /api/mobile/provinces?token=asdfasdf&lang=EN





### Projects Provinces ###

Return a list of provinces associated with all projects:

> /api/mobile/projectsProvinces?token=asdfasdf&lang=EN





### Regions ###

Returns a list of regions with their region ID's:

> /api/mobile/regions?token=asdfasdf&lang=EN




### User Projects ###


Returns the projects to which the user is registered, along with a status.

> /api/mobile/userProjects?token=asdfasdf&lang=EN	










### Species ###

Returns a list of species:

> /api/mobile/species?token=asdfasdf&lang=EN



### Species Groups ###

Returns a list of species groups:

> /api/mobile/speciesGroups?token=asdfasdf&lang=EN

 
### Species Codes ###

Returns a list of 4-letter Species Codes:

> /api/mobile/speciesCodes?token=asdfasdf&lang=EN



### Species EBIRD Codes ###

Returns a list of species codes from the EBIRD checklist:

> /api/mobile/speciesEbird?token=asdfasdf&lang=EN


### Breeding Codes ###

Returns a list of breeding evidence codes:

> /api/mobile/breedingCodes?token=asdfasdf&lang=EN



### Species Invalid Breeding Evidence Codes ###

Returns a list of invalid breeding evidence codes for species:

> /api/mobile/speciesInvalidBreedingEvidence?token=asdfasdf&lang=EN



### Project Protocols ###

Returns protocol ID's associated with a project:

> /api/mobile/projectProtocols?token=asdfasdf&lang=EN&projectId=1007



**Additional Parameter(s):**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| projectId | Integer | Yes | A project ID |


### Protocol Details ###

Returns detailed information on a specific protocol associated with a project:

> /api/mobile/protocols?token=asdfasdf&lang=EN&projectId=1007&protocolId=95


**Additional Parameter(s):**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| projectId | Integer | Yes | A project ID |
| protocolId | Integer | Yes | A protocol ID |







### Protocol Species ###

Returns species appropriate for a specific protocol:

> /api/mobile/protocolsSpecies?token=asdfasdf&lang=EN&protocolId=95



**Additional Parameter(s):**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| protocolId | Integer | Yes | A protocol ID |





### Province Species ###

Returns species appropriate for a province:

> /api/mobile/speciesProvince?token=asdfasdf&lang=EN&province=ON



**Additional Parameter(s):**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| province | String | Yes | A 2-letter province code |



### Species Region ###

Returns species appropriate for a region within a province:

> /api/mobile/speciesRegion?token=asdfasdf&lang=EN&province=ON&regionId=15


**Additional Parameter(s):**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| province | String | Yes | A 2-letter province code |
| regionId | Integer | Yes | A region ID |



### Sites by Coordinates ###

Returns information about project sites within a bounding box:

> /api/mobile/sitesCoordinates?token=asdfasdf&lang=EN&province=ON&regionId=15

**Additional Parameter(s):**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| projectId | Integer | Yes | A project ID |
| locType | String | Yes | A location type (e.g.: EBIRD) |
| swLat | Float | Yes | The decimal latitude of the south-west corner of the bounding box |
| swLon | Float | Yes | The decimal longitude of the south-west corner of the bounding box |
| neLat | Float | Yes | The decimal latitude of the north-east corner of the bounding box |
| neLon | Float | Yes | The decimal longitude of the north-east corner of the bounding box |





### Sites by Regions ###

Returns information about project sites within a region:

> /api/mobile/sitesRegions?token=asdfasdf&lang=EN&projectId=1007&locType=EBIRD&province=ON&regionId=15



**Additional Parameter(s):**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| projectId | Integer | Yes | A project ID |
| locType | String | Yes | A location type (e.g.: EBIRD) |
| province | String | Yes | A 2-letter province code |
| regionId | Integer | Yes | A region ID |




### Sites by Square ###

Returns information about sites within a specific UTM square:

> /api/mobile/sitesSquares?token=asdfasdf&lang=EN&projectId=1007&locType=EBIRD&utmSquare=L2291607



**Additional Parameter(s):**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| projectId | Integer | Yes | A project ID |
| locType | String | Yes | A location type (e.g.: EBIRD) |
| utmSquare | String | Yesy | A utm square identifier |




### Find a Square ###

Find a UTM square from a decimal longitude and latitude:

> /api/mobile/findSquare?token=asdfasdf&lang=EN&lon=-76.5050&lat=44.7366



**Additional Parameter(s):**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| lon | Float | Yes | A decimal longitude |
| lat | Float | Yes | A decimal latitude |





## Data Submission #

Submissions will be by HTTP POST. Entrypoints are not yet available.



### Add a Site ###



### Submit Observations ###
