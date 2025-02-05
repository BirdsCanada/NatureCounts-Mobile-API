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
7. [Binary files](#binary-files)


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
| isAdmin | 'Y' if the user has admin access |

Note: The isAdmin parameter can be included on all of the data queries listed below. Where
appropriate it will be used to filter results.


### Project Registration ###

Project registration may require that the project administrator authorizes the registration.
A user's status in a project is returned as part of the response to the `/api/mobile/userProjects`
entrypoint (see below).

A user registration request is triggred by the following:

> /api/mobile/projectRegister?token=asdfasdf&projectId=95&userCode=user+code&note=user+note

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| token | String | Yes | The authentication token |
| projectId | Integer | Yes | The project to which the user will be registered |
| userCode | String | No | An optional project-specific user code provided by the user (max 20 chars) |
| note | String | No | An optional project-specific registration note provided by the user (unlimited chars) |

**Return JSON attributes:**

| Attribute | Type | Notes |
| --------- | ---- | ----- |
| success | String | success or failed depending on the status of the response |
| newStatus | Integer | new user status after the update (2 = pending approval, 7 = registered, etc.) |
| respCode | String | response code (success: already_registered, registr_success, registr_pending; failure: cannot_register; global admins only (by passes approval): registr_approved) |


### Project De-Registration ###

Project de-registration is only available for non-admin project users or global admins.
A user's status in a project is returned as part of the response to the `/api/mobile/userProjects`
entrypoint (see below).

A user de-registration request is triggred by the following:

> /api/mobile/projectDeregister?token=asdfasdf&projectId=95&note=user+note

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| token | String | Yes | The authentication token |
| projectId | Integer | Yes | The project to which the user will be registered |
| note | String | No | An optional project-specific de-registration note provided by the user (unlimited chars) |

**Return JSON attributes:**

| Attribute | Type | Notes |
| --------- | ---- | ----- |
| success | String | success or failed depending on the status of the response |
| newStatus | Integer | new user status after the update (2 = pending approval, 7 = registered, etc.) |
| respCode | String | response code (success: not_registered, delete_success; failure: cannot_delete) |

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

Supports isAdmin parameter.


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

Supports isAdmin parameter.


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


### Protocol Intervals Bands ###

Returns information on a specific protocol intervals distance bands for the Point Count protocols:

> /api/mobile/protocolIntervalsBands?lang=EN&projectId=1007&protocolId=95

**Additional Parameter(s):**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| projectId | Integer | Yes | A project ID |
| protocolId | Integer | Yes | A protocol ID |

### Protocol Intervals Time ###

Returns information on a specific protocol time intervals for the Point Count protocols:

> /api/mobile/protocolIntervalsTime?lang=EN&projectId=1007&protocolId=95

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

> /api/mobile/sitesCoordinates?lang=EN&projectId=1007&locType=EBIRD&swLat=46&swLon=-74&neLat=46.5&neLon=-73.5

**Additional Parameter(s):**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| projectId | Integer | Yes | A project ID |
| locType | String | Yes | A location type (e.g.: EBIRD) |
| swLat | Float | Yes | The decimal latitude of the south-west corner of the bounding box |
| swLon | Float | Yes | The decimal longitude of the south-west corner of the bounding box |
| neLat | Float | Yes | The decimal latitude of the north-east corner of the bounding box |
| neLon | Float | Yes | The decimal longitude of the north-east corner of the bounding box |
| protocolId | Integer | No | A protocol ID used to filter based on locType |



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

Returns information about atlas squares within a region:

> /api/mobile/utmSquares?lang=EN&projectId=1007&statprov=ON&regionId=15


**Additional Parameter(s):**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| projectId | Integer | Yes | A project ID |
| statprov | String | Yes | A 2-letter province code |
| regionId | Integer | Yes | A region ID |



### Sites by Square ###

Returns information about sites within a specific UTM square:

> /api/mobile/sitesSquares?lang=EN&projectId=1007&locType=EBIRD&utmSquare=17TNH42


**Additional Parameter(s):**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| projectId | Integer | Yes | A project ID |
| locType | String | Yes | A location type (e.g.: EBIRD) |
| utmSquare | String | Yesy | A utm square identifier |
| protocolId | Integer | No | A protocol ID used to filter based on locType |



### Sites by radius ###

Returns information about sites within a certain radius (in meters) from a central point:

> /api/mobile/sitesRadius?lang=EN&projectId=1007&protocolId=116&lat=42.614571&lon=-80.456971&radius=10000


**Additional Parameter(s):**

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| projectId | Integer | Yes | A project ID |
| protocolId | Integer | No | A protocol ID used to filter based on locType |
| locType | String | No | A location type (e.g.: EBIRD) |
| lat | Float | Yes | decimal degrees of latitude |
| lon | Float | Yes | decimal degrees of longitude |
| radius | Float | Yes | maximum distance in meters |



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

DEPRECATED. DO NOT USE. A checklist submission must be by http POST, with the following variables:

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


> /api/mobile/submitChecklistEx: flexible protocols (classic list entry)

> /api/mobile/submitChecklistPc: point count protocols (map entry)

**The 2 entry points are for the checklist submission entrypoint to be used for flexible protocols (which includes custom variable handling at three
levels), and the point count protocols, respectively. Both use a very similar format, but the checklist objects vary in a few ways documented below **

A checklist submission must be by http POST, with the following variables:

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| token | String | Yes | The user's token |
| projectId | Integer | Yes | A decimal project ID |
| trace | Integer | No | A value greater than one turns on tracing for development team |
| checklist | JSON Object | Yes | JSON structure of type CHECKLIST_JSON (see below) |

The response to a valid checklist submission event:

| Parameter | Type | Notes |
| --------- | ---- | ----- |
| status | String | Normally: 'success' |
| formId | Integer | The record ID, whether newly created or existing |


TODO: need to document the response to an invalid checklist submisson.


**The CHECKLIST_JSON structure:**

| Attribute | Type | Required | Notes | Mode |
| --------- | ---- | -------- | ----- | ---- |
| obsDate | String | Yes | The observation date in ISO format (eg: 2020-01-25) | Ex\|PC |
| nObservers | Integer | No | Number of observers (default: 1) | Ex\|PC |
| utmSquare | String | Yes | The utm square as 7-character code, if applicable | Ex\|PC |
| statprov | String | Yes | 2-character province code | Ex\|PC |
| regionId | Integer | Yes | Region ID | Ex\|PC |
| ebirdChecklistId | String | Yes | eBird checklist ID used to validate this checklist, when applicable | Ex\|PC |
| protocolId | Integer | Yes | Protocol ID | Ex\|PC |
| track | JSON Object | No | A vector of JSON objects of type TRACK_JSON (when the track feature is enabled) | Ex |
| isComplete | Boolean | No | Set to true if checklist reports every species detected | Ex\|PC |
| stations | JSON Array | Yes | A vector of JSON objects of type STATION_JSON | Ex\|PC |
| customVars | JSON Array | No | JSON structure of type CUSTOM_JSON (see below) at the checklist level | Ex\|PC |
| accuracyThreshold | Float | No | Accuracy threshold selected by the user to clean the GPS track | Ex |
| mobileId | String | Yes | Universal Unique Identifier (UUID) generated by the submission app | Ex\|PC |
| device | String | No | Device model | Ex\|PC |
| OS | String | No | Device OS version | Ex\|PC |
| appVersion | String | No | NatureCounts app version number (e.g. 4.4.0) | Ex\|PC |
| comments | String | No | User comments for the submission | Ex\|PC |

**The TRACK_JSON structure:**

Only applicable for the submitChecklistEx entry point

| Attribute | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| longitude | Vector of Floats | Yes | Vector of decimal longitude coordinates recorded from the GPS  |
| latitude | Vector of Floats | Yes | Vector of decimal latitude coordinate recorded from the GPS |
| altitude | Vector of Floats | Yes | Vector of altitude (m) recorded from the GPS |
| timestamp | Vector of Floats | Yes | Vector of Unix timestamp recorded from the GPS |
| accuracy | Vector of Floats | Yes | Vector of accuracy values (in meters) recorded from the GPS |

Note: decimal coordinates should be rounded to the nearest 6th decimals, and altitude, timestamps and accuracy to the nearest decimal.

```
Example: track: {"longitude": [-80.123456, -80.123457, -80.123458], "latitude": [45.987654, 45.987653, 45.987652], , "altitude": [100.1, 100.5, 100.2], "timestamp": [1591120800.0, 1591120830.0, 1591120860.0], "accuracy": [12.5, 20.1, 1.2]}
```

**The CUSTOM_JSON structure:**

| Attribute | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| customId | String | Yes | The customId as provided in the protocol custom variable query response |
| level | String | Yes | The level value provided as part of the customVars structure |
| values | JSON Array | Yes | A JSON Array of String for the values of the custom variable. NOTE: So far, I believe that all variables used do not allow multiple values, so the array seems to always be of length 1. Even the Java class only supports a single string for now. |


**The STATION_JSON structure:**

The station object structure is used for both the list and the map entry modes (Ex and PC).

| Attribute | Type | Required | Notes | Mode |
| --------- | ---- | -------- | ----- | ---- |
| stationId | Integer | Yes | stationId 0 is reserved for representing the entire checklist period, and stationId 1 or greater represent linked survey events within the checklist (e.g. point counts) | Ex\|PC |
| startTime | Float | Yes | Start time of the observation. For stationId 0, this is the start of the entire checklist (in decimal hours) | Ex\|PC |
| effortType | String | Yes | One of: incidental, traveling, stationary or area search. Always stationary for PC | Ex\|PC |
| duration | Float | Yes | Duration in decimal hours, required except for incidental protocols. For stationId 0, this is the duration of the entire checklist INCLUDING sub stations | Ex\|PC |
| distance | Float | Yes | Distance in km for travelling protocols. For stationId 0, this is the distance of the entire checklist INCLUDING sub stations | Ex |
| area | Float | Yes | Area in ha for area search protocols. For stationId 0, this is the area of the entire checklist INCLUDING sub stations | Ex |
| latitude | Float | Yes | Decimal latitude at the start of the station | Ex\|PC |
| longitude | Float | Yes | Degrees longitude at the start of the station | Ex\|PC |
| locationAccuracyRange | Float | No | Location accuracy (in meters) provided by the GPS | Ex\|PC |
| locationSelectionTime | Long | No | Unix Timestamp indicating when the site location was selected by the user | Ex\|PC |
| locationAutoSelected | Boolean | No | Whether the location coordinates were derived from the GPS (true), or from the map (false) | Ex\|PC |
| locId | Integer | No | Existing location ID when submitting from an existing site, or resubmitting an existing checklist | Ex\|PC |
| locName | String | Yes | Name of the location. names of public sites should not be editable | Ex\|PC |
| comments | String | No | General comments for the station | Ex\|PC |
| customVars | JSON Array | No | Vector of custom variables, unique to the protocol. JSON structure of type CUSTOM_JSON (see above) at the station level| Ex\|PC |
| species | JSON Array | Yes | A vector of JSON objects of type SPECIES_JSON or of type SPECIES2_JSON | Ex\|PC |
| subProtocolId | Integer | *Deprecated* | Identifier for the station sub-protocol. Ignored when stationId = 0 | Ex |
| intervalIds | Integer | *Deprecated* | list of interval_id's used for the protocol in the species data (if missing, the id's are assumed to be incremental (e.g. 1, 2, 3), but this may not always be true for all protocols. If provided, this is expected to match the size of the counts array in species under this station. For station 0, use the intervals matching the protocolId, for other stations, this should be the intervals matching subProtocolId. Note: now that interval counts are stored in the customVars object, the list of intervalIds should no longer be used. | Ex\|PC |

Note: the SPECIES2_JSON structure is used only when submitting via the /api/mobile/submitChecklistEx entrypoint

**The SPECIES_JSON structure:**

**Deprecated: This will be maintained only for the lifespan of the initial app release.**

| Attribute | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| speciesId | Integer | Yes | numeric NatureCounts taxononic ID |
| recordId | Integer | No | existing recordId provided by the API when resubmitting an existing checklist, blank for new submissions |
| breedingEvid | Integer | Yes | numeric ID for the breeding evidence code. Users should only see the associated alpha breeding code, but the API requires the numeric identifier |
| counts | JSON Array | *Deprecated* | DEPRECATED. A vector of counts matching the protocol requirement |
| comments | String | No | additional species comments provided by the user |
| distance | Float | No | Distance (in meter) between the bird and the observer. Only applicable to protocols supporting multiple records per species. |
| distanceToBird | Float | *Deprecated*  | DEPRECATED. use "distance". Distance (in meter) between the bird and the observer. Only applicable to protocols supporting multiple records per species. |
| direction | String | No | Cardinal direction (e.g. N, NW, SE, etc.) of the bird relative to the observer. Only applicable to protocols supporting multiple records per species. |
| bearing | Integer | No | Bearing (in degrees) of the bird position relative to the observer. Only applicable to protocols supporting multiple records per species. |
| bearingToBird | Integer | *Deprecated*  | DEPRECATED. Use "bearing". Bearing (in degrees) of the bird position relative to the observer. Only applicable to protocols supporting multiple records per species. |
| positionsLongitude | JSON Array | No | List of coordinates representing individual longitude of birds of a given species |
| positionsLatitude | JSON Array | No | List of coordinates representing individual latitude of birds of a given species |
| positionsCounts | JSON Array | No | List of integer values representing individual counts of birds of a given species at the coordinates given by positionsLongitude and positionsLatitude (at the same index position) |
| positionsBreedingEvid | JSON Array | No | List of integer values representing the breeding evidence code of birds of a given species at the coordinates given by positionsLongitude and positionsLatitude (at the same index position) |
| flag | Integer | Yes | Code for the type of flag used to validate the data based on the species lists, indicating which observations should be documented<sup>1</sup> |

**The SPECIES2_JSON structure:**

| Attribute | Type | Required | Notes | Mode |
| --------- | ---- | -------- | ----- | ---- |
| speciesId | Integer | Yes | numeric NatureCounts taxononic ID | Ex\|PC |
| records | JSON Array | Yes | and array of RECORD_JSON objects | Ex\|PC |
| flag | Integer | Yes | Code for the type of flag used to validate the data based on the species lists, indicating which observations should be documented<sup>1</sup> | Ex\|PC |
| comments | String | No | additional species comments provided by the user | Ex\|PC |

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

In list entry (Ex), each record may either represent a single species (multiple_records=N) or a single individual (multiple_records=Y). For point counts, the option multiple_records = Y is ALWAYS assumed (each record matches a single point on the map).

| Attribute | Type | Required | Notes | Mode |
| --------- | ---- | -------- | ----- | ---- |
| recordId | Integer | No | existing recordId provided by the API when resubmitting an existing checklist, blank for new submissions | Ex\|PC |
| breedingEvid | Integer | No | numeric ID for the breeding evidence code. Users should only see the associated alpha breeding code, but the API accepts either the numeric value or the alpha code | Ex |
| distanceToBird | Float | No | For protocols that support multiple records per species in the same station. Distance to birds from observer in meters. | Ex\|PC |
| bearingToBird | Float | No | For protocols that support multiple records per species in the same station. Bearing to birds from observer in degrees (0 = north).  | Ex\|PC |
| positionsArray | JSON Array | No | Array of POSITIONS_JSON objects (see below; markers for rare species in list entry), each with the following elements: **latitude, longitude, count, breedingEvid (including bandId and timeIntervalId for point counts)**, representing individual point markers  | Ex\|PC |
| customVars | JSON Array | No | JSON structure of type CUSTOM_JSON (see above) at the record level. | Ex\|PC |
| speciesCode | Integer | Yes | species 4-letter code entered by the user for point count entry. | PC |
| counts | JSON Array | *Deprecated*  | DEPRECATED (see positionsArray). A vector of counts matching the protocol requirement. For point counts, the array is always of length 1. Note: counts for intervals are now stored in the customVars object.  | Ex |
| positionsLongitude | JSON Array | *Deprecated*  | DEPRECATED (see positionsArray). List of coordinates representing individual longitude of birds of a given species | Ex |
| positionsLatitude | JSON Array | *Deprecated*  | DEPRECATED (see positionsArray). List of coordinates representing individual latitude of birds of a given species | Ex |
| positionsCounts | JSON Array | *Deprecated*  | DEPRECATED (see positionsArray). List of integer values representing individual counts of birds of a given species at the coordinates given by positionsLongitude and positionsLatitude (at the same index position) | Ex |
| positionsBreedingEvid | JSON Array | *Deprecated*  | DEPRECATED (see positionsArray). List of integer values representing the breeding evidence code of birds of a given species at the coordinates given by positionsLongitude and positionsLatitude (at the same index position) | Ex |
| longitude | Float | *Deprecated* | the longitude of the record observation. | Ex |
| latitude | Float | *Deprecated* | the latitude of the record observation. | Ex |
| time | Float | *Deprecated* | The decimal minutes since start of the observation | Ex |

**The POSITIONS_JSON structure:**

| Attribute | Type | Required | Notes | Mode |
| --------- | ---- | -------- | ----- | ---- |
| latitude | Float | Yes | decimal latitude coordinates for the marker  | Ex\|PC |
| longitude | Float | Yes | decimal longitude coordinates for the marker  | Ex\|PC |
| breedingEvid | String/Integer | No | breeding code for the marker  | Ex\|PC |
| count | Integer | No | count of the number of individual at the marker position  | Ex\|PC |
| bandId | Integer | Yes | distance band ID (as defined by the protocol) matching the distance from the observer for the marker (e.g. bandId = 1 for 0-50 meters) | PC |
| timeIntervalId | JSON Array | Yes | array of time intervals ID's (as defined by the protocol) and matching the time intervals in which the observer has reported a record. The protocol option multiple_intervals defines whether multiple values can be entered in this array. | PC |

## Binary Files ##

Binary files were introduced in late 2023 as a way to optimize the offline data caching process and replace several of the API calls that were retrieving large amounts of data in a JSON format and inserting them locally in a sqlite file within the app. 

Each table (previously accessed as a JSON object) are now stored in one of the available binaries. Binaries are cached on the API server, and updated at a varying frequency depending on the type of data. For for types of binaries, there are multiple files available depending on the language, each represented by a unique key (e.g. project-1007-en), the project and the state/province(s) selected by the user.  Note that all keys are in **lower case**. Binaries are grouped as follow:

| binary name | parameters | scope | update freq. | key structure |
| ----------- | ---------- | ----- | ------------ | ------------- |
| core | lang | app-wide | uncommon | core-&lt;lang&gt; |
| session | lang | app-wide | common | session-&lt;lang&gt; |
| project | lang, projectId | project | common | project-&lt;projectId&gt;-&lt;lang&gt; |
| sites | projectId, statprov | project and jurisdisction | monthly or less | sites-&lt;projectId&gt;-&lt;statprov&gt; |
| species | statprov | jurisdisction | monthly | species-&lt;statprov&gt; |
| squares | statprov | jurisdisction | rare | squares-&lt;statprov&gt; |

### List of tables included in binaries ###

Note that the table names and their content should match the API entry points used previously to download JSON data (e.g. breedingCodes contains the same data as /api/mobile/breedingCodes). Please refer to the documentation above for each table.

**core**:
- breedingCodes
- languages
- provinces
- regions
- species
- speciesGroups
- speciesInvalidBreedingEvidence
- speciesCodes
- speciesStatusSymbols

**session**:
- projects
- projectsProvinces
- listBinaries 

**project**:
- protocolIntervals
- protocolIntervalsBands
- protocolIntervalsTime
- projectProtocols
- protocolsSpecies
- protocolCustomVars
- protocolDetails

**sites**:
- sitesSquares

**species**:

- speciesProvince
- speciesRegion
- speciesEbird
- speciesEbirdLimits 
- speciesBreedingDates

**squares**:
- utmSquares

### Binary API details ###

### List of Binaries ###

> /api/mobile/listBinaries

Returns a complete table of the binaries available for all projects on the server side, including the date when they were last modified. This entry point is more for information and likely not needed by the app. Note that the content of this query is also available within the _session_ binaries.

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| token | String | Yes | The user's token |

The response to a valid request is a JSON array containing the list of all binary files available across all projects, languages and regions:

| Parameter | Type | Notes |
| --------- | ---- | ----- |
| key | String | a String representing the key for the binary file (e.g. core-en, project-1007-en) |
| table group | String | The table group to which the binary belongs (core, project, session, sites, species, or squares) |
| statprov | String | The state of province code for the binary (only applicable for binaries within the sites, species and squares groups) |
| project_id | Integer | The project ID for the binary (only applicable for binaries within the project and sites groups) |
| lang | String | The language code (EN or FR) for the binary (only applicable for binaries within the core, project and session) |
| last_modified | Datetime | The datetime string at which the binary was last modified |
| file_size | Integer | The size of the binary in bytes |

### Downloading a single binary file ###

> /api/mobile/binaries/sqlite

Returns a single binary file (sqlite format) matching the unique key (e.g. project-1007-fr).  If the file requested does not exist a 404 error is returned. I suspect this entry point will not be necessary and all downloads should be requested using the zip format, including those that only require a single file.

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| token | String | Yes | The user's token |
| file | String | Yes | the key for the binary file (e.g. project-1007-en) |

Response: a sqlite file 

### Downloading multiple binary files as a zip archive ###

> /api/mobile/binaries/zip

Returns a zip file containing several binaries files (sqlite format) matching the unique keys provided in the request (e.g. project-1007-fr). Binaries will be included in the response only when the server version of the binary is newer than the last_modified value, or when the last_modified value provided in the request is missing or null. If no binaries match the request (i.e. the app already has all the required files), an empty zip file will be returned.

| Parameter | Type | Required | Notes |
| --------- | ---- | -------- | ----- |
| token | String | Yes | The user's token |
| files | JSON array | Yes | An array of objects containing details about the files to download. Each object contains a file (required) and a last_modified (optional) value (using Linux timestamps in seconds since the Unix epoch)  |

Example request: token=XXXXX&files=[{file: “project-1007-en”,last_modified: 1701916988},{file: “project-1007-fr”,last_modified: 1701916997},{file:”session-en”}]

Response: a zip file containing sqlite binaries

