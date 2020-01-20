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




## Lookup Data Querying ##

All results are returned as 'data frames', suitable for R client parsing / management.

A basic data query looks like this:

> /api/metadata/entryPoint?token=asdfasdf

| Parameter Name | Type | Required |Explanation |
|----------------|------|----------|------------|
|token|String|Yes|The user's token, returned by the authentication mechanism|

The filter object carries atributes whose name is the field to be filtered, and whose value is the value to
use in filtering. Multiple filters can be included in a request.

The filter mechanism only supports equality operations.


### Projects ###

Return a list of projects that allow public participation, along with protocols

> /api/metadata/projects?token=asdfasdf

| Parameter Name | Type | Required |Explanation |
|----------------|------|----------|------------|
|token|String|Yes|The user's token, returned by the authentication mechanism|



### Protocol Details ###

> /api/metadata/protocols?token=asdfasdf

| Parameter Name | Type | Required |Explanation |
|----------------|------|----------|------------|
|token|String|Yes|The user's token, returned by the authentication mechanism|


### Project Protocols ###

> /api/metadata/projectProtocola?token=asdfasdf

| Parameter Name | Type | Required |Explanation |
|----------------|------|----------|------------|
|token|String|Yes|The user's token, returned by the authentication mechanism|


### Project Provinces ###

> /api/metadata/projectProvinces?token=asdfasdf

| Parameter Name | Type | Required |Explanation |
|----------------|------|----------|------------|
|token|String|Yes|The user's token, returned by the authentication mechanism|


### Project Sites ###

> /api/metadata/projectSites?token=asdfasdf

| Parameter Name | Type | Required |Explanation |
|----------------|------|----------|------------|
|token|String|Yes|The user's token, returned by the authentication mechanism|


### Protocol Species ###

> /api/metadata/protocolSpecies?token=asdfasdf

| Parameter Name | Type | Required |Explanation |
|----------------|------|----------|------------|
|token|String|Yes|The user's token, returned by the authentication mechanism|


### User Projects ###

> /api/metadata/entryPoint?token=asdfasdf

| Parameter Name | Type | Required |Explanation |
|----------------|------|----------|------------|
|token|String|Yes|The user's token, returned by the authentication mechanism|


### User Profile ###

> /api/query/userProfile

### Sites ###

> /api/metadata/sites?token=asdfasdf

| Parameter Name | Type | Required |Explanation |
|----------------|------|----------|------------|
|token|String|Yes|The user's token, returned by the authentication mechanism|




### Breeding Codes ###

> /api/metadata/breeding_codes?token=asdfasdf

| Parameter Name | Type | Required |Explanation |
|----------------|------|----------|------------|
|token|String|Yes|The user's token, returned by the authentication mechanism|


### Provinces ###

> /api/metadata/provinces?token=asdfasdf

| Parameter Name | Type | Required |Explanation |
|----------------|------|----------|------------|
|token|String|Yes|The user's token, returned by the authentication mechanism|

### Regions ###

> /api/query/regions

### Species ###

> /api/query/species

### Species Codes ###

> /api/query/speciesCodes


### Species Groups ###

> /api/query/speciesGroups

### Invalid breeding Codes ###

> /api/query/????????????

### Species Ebird ###

> /api/query/ebird

### Species Ebird Limits ###

> /api/query/ebirdLimits

### Species Provinces ###

> /api/query/speciesProvinces

### Species Regions ###

> /api/query/speciesRegions

## Data Submission #

HTTP POST.

### Add a Site ###



### Submit Observations ###

