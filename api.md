The Open 311 Simple API
==

There are two basic classes of API methods: _list_ methods and _info_
methods. The former are meant return the minimum amount of data around a service
or incident to allow a user to perform a discrete action; the latter return all
the information available for a service or incident.

Specification
==

The API method specification is defined as a JSON data structure containing basic metadata about each individual method such that it can be used by actual running code to delegate API requests and generate documentation.

Developers are still required to implement their own dispatching system in order to serve and process those API methods. The modeling of the API specification in JSON is simply meant as a language-agnostic configuration file that can be easily shared across programming languages (since they basically all have JSON decoders to translate the data in to native data structures).

API methods are defined in [api-methods.json](https://github.com/straup/open311-simple/blob/master/api-methods.json) document. The [api-methods.md](https://github.com/straup/open311-simple/blob/master/api-methods.md) document contains human-friendly documentation for the API.

Transport
==

All API requests and responses are delivered using [HTTP](http://www.w3.org/Protocols/). Wherever possible API requests are performed using the HTTP _GET_ method. In some cases where security considerations demand it (for example, reporting an incident) API requests are performed using the HTTP _POST_ method.

Successful API methods are returned using HTTP responses in the 200 range. All others are expected to return responses in the 400-500 range although this is left to the discretion of individual cities.

_The emphasis here is that, as much as possible, the API may be explored and tested by as many people as possible using nothing more complicated that a web browser._

Response formats
==

The default API response format is [JSON](http://www.json.org/).

Additional responses are left to the discretion of individual cities and are identified in API request using the _format_ parameter.

(U)IDs
==

The Open311 Simple specification is agnostic about whether or not unique identifiers (UID) should be strings or integers. This is left to the discretion of individual cities and the requirements of their technical infrastructure.

Out of necessity, though, _client applications_ may need to treat every UID as a string in order to provide consistent access (read: local database storage) multiple cities.

The example responses included with the specification assume integer based UIDs.

See also: [Open311 Simple Issues #1](https://github.com/straup/open311-simple/issues/1)

Character encoding
==

UTF-8.

Pagination
==

If an API method is a _list_-style method (described above) then it will always paginate its responses. Pagination is simply a way to limit the number of responses returned for a single method call, as defined by a _per_page_ parameter (as in the number of results to return). Offsets are calculated by multiplying the _page_ and _per_page_ parameters (or attributes in the case of API responses).

Requests to paginated methods may pass the following parameters:

* **page** – the current offset (calculated by multiplying _page_ by _per_page_) to start fetching results from.

* **per_page** – the number of results to return for a given API request.

Default values are left to the discretion of individual cities.

Paginated responses will always return the following attributes:

* **total** – The total number of results for the query issued by the API method, regardless of pagination.

* **pages** – The total number of paginated results (or "pages") that will be returned by the API method. This number is calculated by dividing _total_ by the _per_page_ parameter (or default value, defined above).

* **page** – The current offset "page" for the total result set.

For example:

	GET example.com/api?method=open311.services.getList?page=2&per_page=2

	{
		"total": 5,
		"pages": 3,
		"page": 2,
		"services": [
			{ "id": 1, "name": "...", "type": "..." },
			{ "id": 2, "name": "...", "type": "..." },
		]
	}

Incidents
==

The minimum (default) data structure for incidents is very simple: It contains a unique identifier, references to the service that was reported, the incident's current status and its location, relevent dates and any descriptive text that the reporting the incident may have included. For example:

	{
		"id": 999,
		"service_id": 2,
		"status_id": 1,
		"created": "2011-03-15T23:22:45Z",
		"modified": "2011-03-16T10:17:33Z",
		"latitude": 37.23,
		"longitude": -122.45,
		"description": "This fire hydrant has been leaking for two days now",
	}

That's it. Any additional properties passed back in an incident response are left to the discretion of individual cities. Which means the following keys (properties) are reserved:

* **id** - A unique identifier for the incident report. Incident IDs are generated by and unique to inidividual cities. Client MUST be able to pass this to the _open311.indicidents.getInfo_ API method to retrieve details for a report.

* **service_id** – A unique identifier for the service associated with an incident report. Service IDs are generated by and unique to inidividual cities. Client MUST be able to call the _open311.services.getList_ API method to retrieve a complete list of service IDs and their descriptions.

* **status_id** - A unique identifier for the status of an incident report. Status IDs are generated by and unique to inidividual cities. Client MUST be able to call the _open311.incidents.getStatuses_ API method to retrieve a complete list of status IDs and their descriptions.

* **created** - The date the incident was reported; see the "Dates" section (below) for details on dates are represented.

* **modified** - The most recent date the incident was modified; see the "Dates" section (below) for details on dates are represented.

* **latitude** - The latitude for the incident report; see the "Geo" section (below) for details on how coordinates are represented.

* **longitude** - The longitude for the incident report; see the "Geo" section (below) for details on how coordinates are represented.

* **description** - Any addition descriptive text that the user who reported the incident may have included with the initial report.

_For the time being, whether or not the a reference to the individual user reporting an incident is included in an incident report is left to the discretion of individual cities. Trying to reconcile the privacy requirements, and technical implimentations to support them, across multiple jurisdictions is outside the scope of this document._

Non-standard properties should, wherever applicable, adopt the [OpenStreetMap (OSM) standard for tagging](https://wiki.openstreetmap.org/wiki/Any_tags_you_like) and encourage users to follow [the guidelines and naming conventions that the OSM community has developed](https://wiki.openstreetmap.org/wiki/Map_Features) and refined over time. This model has been proven to be both robust enough to be used to power the OSM map renderer and flexible enough to meet the needs of tinkerers and playful enough to encourage innovations no one even considered at the outset.

Dates
==

All dates are recorded using the [W3C DateTime format](http://www.w3.org/TR/NOTE-datetime). 

All dates are passed a single-value reference to a date or date range, replacing
convential "start" and "stop" prefixes for the various date arguments.

Because the "-" and ":" characters are used by the W3C DateTime format date
ranges are separated using a semicolon (;) character.

Example values:

* A single, full day: 2010-05-26.

* A full week: 2010-05-23;2010-05-29.

* A single hour: 2010-05-26T21:00:00Z;2010-05-26T22:00:00Z.

Geo
==

All geographic data should be passed to (and returned from) the API using the
unprojected [WGS84](http://spatialreference.org/ref/epsg/4326/) datum. The phrase "unprojected WGS84 data" can be roughly translated as: _Plain-old latitude and longitude, the way those of us who don't study GIS think about things._

Geographic coordinates are expressed as latitude followed by longitude. Bounding boxes are expressed as a set of coordinates representing the South-West and North-East edges of the container.

The "where" argument
--

The "where" argument (used by the _open311.incidents.search_ method) wraps all geographic queries in a single
interface. Argument values are prefixed with a human-readable string followed by
a colon (":") followed a string representing a geographic location.

The prefix is used by parsers to determine how the rest of a "where" string
should be interpreted. For example:

* Bounding box: ?where=bbox:37.788,-122.344,37.857,-122.256

* Around a point: ?where=near:37.804376,-122.271180

* In a geohash: ?where=geohash:9q9p1dhf7

* In a zip code: ?where=zip:94612

The single parameter removes possible conflicts or overlaps between other
parameters, and introduces an extensible way to namespace "known" areas like zip
codes, countries and allow individual cities to introduce place types specific
to their jurisdiction (for example: housing lots or building identifiers).

**All Open311 Simple providers MUST implement the "bbox:" prefix to allow for geographic queries within a bounding box.** All other prefixes are left to the discretion (and technical infrastructure) of individual cities. API clients may request a list of supported prefixed using the _open311.where.getList_ API method.

Authentication
==

Not all cities require that incident reports be filed with an associated user (or account) ID. Those that don't really should in order that both cities and individual contributors may review and audit incident reports. Requiring user accounts introduces an extra burden on both cities and users. For cities it means maintaining an additional database of user accounts and for users it means an extra sign-up process and another password to remember. This problem can by and large be mitigated by using one or more social web services are a single-sign-on (or validation) service. Both Facebook and Twitter are happy to perform this role for third-parties and have well-developed and widely adopted platforms for implementing this scenario.

The Open311 Simple specification dictates that all API methods that require authentication use the [OAuth2](http://oauth.net/2/) delegated authentication standard. Additionally it should be possible for an individual user to scope a query (using the _open311.incidents.search_ API method) an Open311 Simple dataset to themselves by passing a valid OAuth 2 token and signature with their request.

Questions:
==

* Should there be a _open311.incidents.getHistory_ API method to return the comments and progress, over time, for an incident report ?

See also:
==

* [open311-simple-papp](https://github.com/straup/open311-simple-app) - A reference implementation of the Open311 spec built using [Flamework](https://github.com/straup/flamework) (read: PHP and MySQL).
