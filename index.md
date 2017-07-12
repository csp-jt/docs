# JobTrack v1 API

1. [Introduction](#introduction)
1. [Schema](#schema)
1. [Authentication](#authentication)
1. [Endpoints](#endpoints)
    1. [Companies](#companies)
    1. [Contacts](#contacts)
    1. [Job Applications](#job-applications)
    1. [Users](#users)

## Introduction
The JobTrack v1 API adheres to the [JSON API spec](http://jsonapi.org/) and enables users to programatically interact with their JobTrack account. The API exposes a large portion of the JobTrack data model, such as Users, Job Applications and Contacts.

## Schema

Requests must be sent via HTTPS in JSON format and must include the following header: `Content-Type: application/vnd.api+json`. Responses will always be returned in JSON format. Dates and times are returned as ISO 8601 formatted strings. All requests to the API must have URLs relative to the base API URL:

```
https://jobtrack.io/api/v1/
```

## Authentication

The JobTrack API uses the [OAuth 2.0 protocol](https://oauth.net/2/) for authentication and supports the Client Credentials and Authorization Code (recommended) flows.

All requests to the JobTrack API must be authenticated with an OAuth bearer token. To register an application, please contact [support@jobtrack.io](mailto:support@jobtrack.io).

To authenticate using the Authorization header, set the header's value to `Bearer <token>`. So, if your token was `abc123`, your HTTP request would include the header `Authorization: Bearer abc123`.

For example, authenticating a request using curl would mean running a command similar to this one:

```
curl -H "Authorization: Bearer abc123" "https://jobtrack.io/api/v1/job-applications"
```

All requests to the JobTrack API require an Authorization header. For brevity, future API request examples in this documentation will not include the example Authorization header parameter.

## Endpoints

### Companies
* `GET companies`
* `GET companies/<id>`
* `POST companies`

### Contacts
* `GET contacts`
* `GET contacts/<id>`
* `POST contacts`
* `PUT contacts/<id>`

### Job Applications
* `GET job-applications`
* `GET job-applications/<id>`
* `POST job-applications`
* `PUT job-applications/<id>`

### Users
* `GET users`
* `POST users/invite`
