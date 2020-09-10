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
3. [API Last Modified Times Bundle](#api-last-modified-times-bundle)
4. [Errors](#errors)
5. [Reference Data](#reference-data)
6. [Data Submission](#data-submission)
	1. [Adding a Site](#add-a-site)
	2. [Checklist Submission](#checklist-submission)


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

>**Example URL:** /api/mobile/authenticate?username=asdfasdf&password=qwerty

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

Project registration may require that the project administrator authorizes the registration.
A user's status in a project is returned as part of the response to the `/api/mobile/userProjects`
entrypoint (see below).

A user registration request is triggred by the following:

> /api/mobile/projectRegister?token=asdfasdf&projectId=95

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| token | String | Yes | The authentication token |
| projectId | Integer | Yes | The project to which the user will be registered |
| lang | String | No | A 2-letter language code, defaulting to EN |



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



## API Last Modified Times Bundle ##

Returns a set of entrypoints and parameters that should be used to
refresh local cache with modified data.


**Precise description of this data package to be added shortly.**

> /api/mobile/sync?projectIds=1007,1009&ifModifiedSince=2019-12-15T15:23:12Z

Authenticated: Yes

**Parameters**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| projectIds | String | Yes | A comma delimated list of project Ids |
| ifModifiedSince | String | No | A ISO-8601 formatted datetime (eg: 2020-01-12T15:30:23Z) |

**Return JSON attributes:**


| Attribute | Type | Notes |
| --------- | ---- | ----- |
| entryPoint | String | The entry point suffix to call |
| projectId | Integer | A project Id |
| checklistId | Integer | A checklist Id |
| statprov | Integer | A 2-letter province code |
| protocolId | Integer | A protocol Id |
| regionId | Integer | A region Id |

Any attributes that are present and not null should be used to:

1. delete records from appropriate tables in the local cache
2. query the api to get replacement records for those deleted.




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

Reference data are returned as a JSON object with a single element `'items'`, which
is an array of JSON objects. Each object corresponds to a single data record.


Reference Data queries do not require user authentication, but may include a `lang` 
parameter. If the `lang` paramter is not provided, it will default to `EN`.

**Common Parameters**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| lang | String | No | A 2-letter language code, defaulting to EN |




### Projects ###

Return a list of all projects relevant to the mobile app:

> /api/mobile/projects?lang=EN




### Provinces ###

Returns a list of Canadian Provinces with their 2-letter codes:

> /api/mobile/provinces?lang=EN





### Projects Provinces ###

Return a list of provinces associated with all projects:

> /api/mobile/projectsProvinces?lang=EN





### Regions ###

Returns a list of regions with their region ID's:

> /api/mobile/regions?lang=EN




### User Projects ###


Returns the projects to which the user is registered, along with a status.

> /api/mobile/userProjects?lang=EN	










### Species ###

Returns a list of species:

> /api/mobile/species?lang=EN



### Species Groups ###

Returns a list of species groups:

> /api/mobile/speciesGroups?lang=EN

 
### Species Codes ###

Returns a list of 4-letter Species Codes:

> /api/mobile/speciesCodes?lang=EN


### Species Status Symbols ###

Returns status symbols for species:

> /api/mobile/speciesStatusSymbols?lang=EN


### Species EBIRD Codes ###

Returns a list of species codes from the EBIRD checklist:

> /api/mobile/speciesEbird?lang=EN


### Species EBIRD Limits ###


Returns a list of limits for species in an EBIRD checklist:

> /api/mobile/speciesEbirdLimits?lang=EN&checlistId=CL23742



**Additional Parameter(s):**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| checklistId | String | No | An Ebird id (eg: CL23472) |
| statprov | String | No | A 2-letter province code |


### Breeding Codes ###

Returns a list of breeding evidence codes:

> /api/mobile/breedingCodes?lang=EN



### Species Invalid Breeding Evidence Codes ###

Returns a list of invalid breeding evidence codes for species:

> /api/mobile/speciesInvalidBreedingEvidence?lang=EN



### Project Protocols ###

Returns protocol ID's associated with a project:

> /api/mobile/projectProtocols?lang=EN&projectId=1007



**Additional Parameter(s):**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| projectId | Integer | Yes | A project ID |


### Protocol Details ###

Returns detailed information on a specific protocol associated with a project:

> /api/mobile/protocolDetails?lang=EN&projectId=1007&protocolId=95


**Additional Parameter(s):**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| projectId | Integer | Yes | A project ID |
| protocolId | Integer | Yes | A protocol ID |




### Protocol Intervals ###

Returns information on a specific protocol intervals:

> /api/mobile/protocolIntervals?lang=EN&projectId=1007&protocolId=95


**Additional Parameter(s):**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| projectId | Integer | Yes | A project ID |
| protocolId | Integer | Yes | A protocol ID |



### Protocol Custom Variables ###

Returns a list of custom variables (array of objects) appropriate for the protocol and project:

> /api/mobile/protocolCustomVars?lang=EN&projectId=1047&protocolId=95


**Additional Parameter(s):**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| projectId | Integer | Yes | A project ID |
| protocolId | Integer | Yes | A protocol ID |

The variable definitions include the following fields:

| Field | Type | Notes |
| --------- | ---- | ----- |
| custom_id | String | Must be sent back to the api when checklist is submitted |
| level | String | One of: survey, start, end, station, species, interval. Must be sent back to the API when checklist is submitted. |
| label | String | Display label |
| label_fr | String | French display label |
| field_type | String | Specifies the UI element to display (see types below) |
| options | JSON Array | Present for `dropdown` field_type: an array of OPTION_JSON object (see below). |
| sort_order | Integer | If relevant. Determines the display order in the app, within a level. |
| range | String | Context specific |
| required | Boolean | False if null. Whether a value is required for the field. Always enforced for type = “start”, “end” or “station”. For type = “species”, only applies if there is at least one record with a value for the species (i.e. across all records if multiples are allowed). For type = “interval”, only applies if there is at least one value among the other interval variables. (e.g., if the user enters a count, they must also enter a bearing or vice-versa). |
| include_in_total | String | One of N, Y, null |

possible field_type values:

| field_type | Description |
| --------- | ----- |
| text | any arbitrary text is allowed. Opens a modal window for species and interval levels. |
| numeric | any arbitrary number is allowed, including floats (within the min/max range, if provided) |
| integer | any arbitrary integer number is allowed (within the min/max range, if provided)  |
| counter | any arbitrary positive integer number is allowed (within the range, if provided). Important: a single tap on the field would increment the value by 1. A long press would allow keyboard entry. |
| checkbox | a checkbox (true or false). defaults to false. |
| dropdown | A drop down list, with values defined in `options` attribute. Open a modal window in all cases. If the field is not required, null should be added as one of the possible options. |
| yes/no | species case of list defined for convenience, with “yes” and “no” values only. Null are also allowed. |
| radio	| a radio button, with values defined in options attribute. If the field is not required, null should be added as one of the possible options so the field can be deselected. |
| multi-checkbox | similar to a radio type, but using checkboxes that can support multiple selections |

OPTION_JSON Object

Carries option values and labels for field_type `dropdown`

| Attribute | Description |
| --------- | ----------- |
| label | English label to display in dropdown |
| label_fr | French label to display in dropdown |
| value | The option's value, for submission back to the checklist entrypoint |



### Protocol Types ###

Returns details relevant for specific protocols:

> /api/mobile/protocolTypes?lang=EN&projectId=1007&protocolId=95


**Additional Parameter(s):**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| projectId | Integer | Yes | A project ID |
| protocolId | Integer | Yes | A protocol ID |






### Protocol Species ###

Returns species appropriate for a specific protocol:

> /api/mobile/protocolsSpecies?lang=EN&protocolId=95



**Additional Parameter(s):**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| protocolId | Integer | Yes | A protocol ID |
| projectId | Integer | No | A project ID |





### Province Species ###

Returns species appropriate for a province:

> /api/mobile/speciesProvince?lang=EN&statprov=ON



**Additional Parameter(s):**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| statprov | String | Yes | A 2-letter province code |



### Species Region ###

Returns species appropriate for a region within a province:

> /api/mobile/speciesRegion?lang=EN&statprov=ON&regionId=15


**Additional Parameter(s):**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| statprov | String | Yes | A 2-letter province code |
| regionId | Integer | No	| A region ID |



### Sites by Coordinates ###

Returns information about project sites within a bounding box:

> /api/mobile/sitesCoordinates?lang=EN&statprov=ON&regionId=15

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

> /api/mobile/sitesRegions?lang=EN&projectId=1007&locType=EBIRD&statprov=ON&regionId=15



**Additional Parameter(s):**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| projectId | Integer | Yes | A project ID |
| locType | String | Yes | A location type (e.g.: EBIRD) |
| statprov | String | Yes | A 2-letter province code |
| regionId | Integer | Yes | A region ID |



### UTM Squares by Region ###

Returns information about project sites within a region:

> /api/mobile/utmSquares?lang=EN&projectId=1007&statprov=ON&regionId=15



**Additional Parameter(s):**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| projectId | Integer | Yes | A project ID |
| statprov | String | Yes | A 2-letter province code |
| regionId | Integer | Yes | A region ID |




### Sites by Square ###

Returns information about sites within a specific UTM square:

> /api/mobile/sitesSquares?lang=EN&projectId=1007&locType=EBIRD&utmSquare=L2291607



**Additional Parameter(s):**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| projectId | Integer | Yes | A project ID |
| locType | String | Yes | A location type (e.g.: EBIRD) |
| utmSquare | String | Yesy | A utm square identifier |




### Find a Square ###

Find a UTM square from a decimal longitude and latitude:

> /api/mobile/findSquare?lang=EN&lon=-76.5050&lat=44.7366



**Additional Parameter(s):**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| lon | Float | Yes | A decimal longitude |
| lat | Float | Yes | A decimal latitude |





## Data Submission #

Submissions will be by HTTP POST.




### Add a Site ###

Not available yet....


### Checklist Submission ###

> /api/mobile/submitChecklist

A checklist submission must be by http POST, with the following variables:

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| token | String | Yes | The user's token |
| projectId | Integer | Yes | A decimal project ID |
| trace | Integer | No | A value greater than one turns on tracing for development team |
| checklist | JSON Object | Yes | JSON structure of type CHECKLIST_JSON (see below) |


The response to a valid checklist submisson event:

| Parameter | Type | Notes |
| --------- | ---- | ----- |
| status | String | Normally: 'success' |
| formId | Integer | The record ID, whether newly created or existing |


The response to an invalid checklist submisson has not yet been defined......


> /api/mobile/submitChecklistEx

**This is the new checklist submission entrypoint to be used for phase 2 development (which includes custom variable handling at three
levels). The original submission entrypoint defined immediately above is maintained only for the initial release of the mobile app: it will be deprecated in time?**

A checklist submission must be by http POST, with the following variables:

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| token | String | Yes | The user's token |
| projectId | Integer | Yes | A decimal project ID |
| trace | Integer | No | A value greater than one turns on tracing for development team |
| checklist | JSON Object | Yes | JSON structure of type CHECKLIST_JSON (see below) |


The response to a valid checklist submisson event:

| Parameter | Type | Notes |
| --------- | ---- | ----- |
| status | String | Normally: 'success' |
| formId | Integer | The record ID, whether newly created or existing |


The response to an invalid checklist submisson has not yet been defined......



**The CHECKLIST_JSON structure:**

| Attribute | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| obsDate | String | Yes | The observation date in ISO format (eg: 2020-01-25) |
| nObservers | Integer | No | Number of observers (default: 1) |
| utmSquare | String | No | The utm square as 7-character code, if applicable |
| statprov | String | No | 2-character province code |
| regionId | Integer | No | Region ID |
| ebirdChecklistId | String | No | eBird checklist ID used to validate this checklist, when applicable |
| protocolId | Integer | Yes | Protocol ID |
| track | JSON Array | No | A vector of JSON objects of type TRACK_JSON (when the track feature is enabled) |
| isComplete | Boolean | No | Set to true if checklist reports every species detected |
| stations | JSON Array | Yes | A vector of JSON objects of type STATION_JSON |
| customVars | JSON Array | No | JSON structure of type CUSTOM_JSON (see below) |


**The TRACK_JSON structure:**

| Attribute | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| longitude | Vector of Floats | Yes | Vector of decimal longitude coordinates recorded from the GPS  |
| latitude | Vector of Floats | Yes | Vector of decimal latitude coordinate recorded from the GPS |
| altitude | Vector of Floats | Yes | Vector of altitude (m) recorded from the GPS |
| timestamp | Vector of Floats | Yes | Vector of Unix timestamp recorded from the GPS |

Note: decimal coordinates should be rounded to the nearest 6th decimals, and altitude and timestamps to the nearest decimal.

```
Example: track: {"longitude": [-80.123456, -80.123457, -80.123458], "latitude": [45.987654, 45.987653, 45.987652], , "altitude": [100.1, 100.5, 100.2], "timestamp": [1591120800.0, 1591120830.0, 1591120860.0]}
```

**The CUSTOM_JSON structure:**

| Attribute | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| customId | String | Yes | The customId as provided in the protocol custom variable query response |
| level | String | Yes | The level value provided as part of the customVars structure |
| values | String | Yes | A possibly comma-delimited string of values |



**The STATION_JSON structure:**

| Attribute | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| stationId | Integer | No | stationId 0 is reserved for representing the entire checklist period, and stationId 1 or greater represent linked survey events within the checklist (e.g. point counts) |
| startTime | Float | Yes | Start time of the observation. For stationId 0, this is the start of the entire checklist (in decimal hours) |
| effortType | String | Yes | One of: incidental, traveling, stationary or area search |
| duration | Float | Yes | Duration in decimal hours, required except for incidental protocols. For stationId 0, this is the duration of the entire checklist INCLUDING sub stations |
| distance | Float | Yes | Distance in km for travelling protocols. For stationId 0, this is the distance of the entire checklist INCLUDING sub stations |
| area | Float | Yes | Area in ha for area search protocols. For stationId 0, this is the area of the entire checklist INCLUDING sub stations |
| subProtocolId | Integer | No | Identifier for the station sub-protocol. Ignored when stationId = 0 |
| intervalIds | Integer | No | list of interval_id's used for the protocol in the species data (if missing, the id's are assumed to be incremental (e.g. 1, 2, 3), but this may not always be true for all protocols. If provided, this is expected to match the size of the counts array in species under this station. For station 0, use the intervals matching the protocolId, for other stations, this should be the intervals matching subProtocolId |
| latitude | Float | Yes | Decimal latitiude at the start of the station |
| longitude | Float | Yes | Degrees longitude at the start of the station |
| locId | Integer | No | Existing location ID when submitting from an existing site, or resubmitting an existing checklist |
| locName | String | Yes | Name of the location. names of public sites should not be editable |
| comments | String | No | General comments for the station |
| customVars | JSON Array | No | Vector of custom variables, unique to the protocol |
| species | JSON Array | Yes | A vector of JSON objects of type SPECIES_JSON or of type SPECIES2_JSON |
| customVars | JSON Array | No | JSON structure of type CUSTOM_JSON (see above) |

Note: the SPECIES2_JSON structure is used only when submitting via the /api/mobile/submitChecklistEx entrypoint

**The SPECIES_JSON structure:**

**This will be maintained only for the lifespan of the initial app release.**

| Attribute | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| speciesId | Integer | Yes | numeric NatureCounts taxononic ID |
| recordId | Integer | No | existing recordId provided by the API when resubmitting an existing checklist, blank for new submissions |
| breedingEvid | Integer | Yes | numeric ID for the breeding evidence code. Users should only see the associated alpha breeding code, but the API requires the numeric identifier |
| counts | JSON Array | Yes | A vector of counts matching the protocol requirement |
| comments | String | No | additional species comments provided by the user |
| distanceToBird | Float | No | Not yet applicable: for protocols that support multiple records per species in the same station |
| bearingToBird | Float | No | Not yet applicable: for protocols that support multiple records per species in the same station |
| positionsLongitude | JSON Array | No | Not yet applicable: list of coordinates representing individual longitude of birds of a given species |
| positionsLatitude | JSON Array | No | Not yet applicable: list of coordinates representing individual latitude of birds of a given species |
| positionsCounts | JSON Array | No | Not yet applicable: list of integer values representing individual counts of birds of a given species at the coordinates given by positionsLongitude and positionsLatitude (at the same index position) |
| flag | Integer | Yes | Code for the type of flag used to validate the data based on the species lists, indicating which observations should be documented<sup>1</sup> |

**The SPECIES2_JSON structure:**

| Attribute | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| speciesId | Integer | Yes | numeric NatureCounts taxononic ID |
| records | JSON Array | Yes | and array of RECORD_JSON objects |
| flag | Integer | Yes | Code for the type of flag used to validate the data based on the species lists, indicating which observations should be documented<sup>1</sup> |
| comments | String | No | additional species comments provided by the user |

<div style="padding: 15px">
<sup>1</sup> Values for SPECIES_JSON.flag:

<ul>
	<li>0 - common bird, no flag required</li>
	<li>1 - rare species based on eBird filters. Species missing from filter or maximum set a 0 for the date.</li>
	<li>2 - high count based on eBird filters. Observation where the actual count exceeds the filter for the date.</li>
	<li>3 - rare breeder. Observation reported with a breeding evidence code (H or higher) should be documented.</li>
</u>
</div>

**The RECORD_JSON structure:**

| Attribute | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| recordId | Integer | No | existing recordId provided by the API when resubmitting an existing checklist, blank for new submissions |
| breedingEvid | Integer | Yes | numeric ID for the breeding evidence code. Users should only see the associated alpha breeding code, but the API requires the numeric identifier |
| counts | JSON Array | Yes | A vector of counts matching the protocol requirement |
| distanceToBird | Float | No | Not yet applicable: for protocols that support multiple records per species in the same station |
| bearingToBird | Float | No | Not yet applicable: for protocols that support multiple records per species in the same station |
| positionsLongitude | JSON Array | No | Not yet applicable: list of coordinates representing individual longitude of birds of a given species |
| positionsLatitude | JSON Array | No | Not yet applicable: list of coordinates representing individual latitude of birds of a given species |
| positionsCounts | JSON Array | No | Not yet applicable: list of integer values representing individual counts of birds of a given species at the coordinates given by positionsLongitude and positionsLatitude (at the same index position) |
| time | Float | Yes | The decimal minutes since start of the observation |
| longitude | Float | Yes | the longitude of the record observation |
| latitude | Float | Yes | the latitude of the record observation |
| customVars | JSON Array | No | JSON structure of type CUSTOM_JSON (see above) |
