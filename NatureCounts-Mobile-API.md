# NatureCounts Mobile App API Verson 1.0 #

This set of entrypoints is designed to serve the needs of NatureCounts mobile app.

All queries are over HTTPS GET requests. Query responses are JSON object, mostly structured as a 'data frame': attribute names are the data
field names, and attribute values are vectors of field values.

## Registration, Authentication ##

Where authemtication is needed, queries must include a token parameter. The token is obtained via the authentication mechanism built for the 
NatureCounts R-client API.

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
