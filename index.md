# JobTrack v1 API

1. [Introduction](#introduction)
1. [Schema](#schema)
1. [Authentication](#authentication)
1. [Endpoints](#endpoints)
    1. [Company Masters](#company-masters)
    1. [Companies](#companies)
    1. [Contacts](#contacts)
    1. [Events](#events)
    1. [Job Applications](#job-applications)
    1. [Tasks](#tasks)
    1. [Users](#users)

## Introduction
The JobTrack v1 API adheres to the [JSON API spec](http://jsonapi.org/) and enables users to programatically interact with their JobTrack account. The API exposes a large portion of the JobTrack data model, such as Users, Job Applications and Contacts.

## Schema

Requests must be sent via HTTPS in JSON format and must include the following header: `Content-Type: application/vnd.api+json`. Responses will always be returned in JSON format. Dates and times are returned as ISO 8601 formatted strings. All requests to the API must have URLs relative to the base API URL:

```
https://api.jobtrack.io/api/v1/
```

## Authentication

The JobTrack API uses the [OAuth 2.0 protocol](https://oauth.net/2/) for authentication and supports the Client Credentials and Authorization Code (recommended) flows.

All requests to the JobTrack API must be authenticated with an OAuth bearer token. To register an application, please contact [support@jobtrack.io](mailto:support@jobtrack.io). 

Every request to the JobTrack API must be accompanied by a valid OAuth token, indicating that your application has been authorized by the JobTrack user in question. When you register an application with us, we'll provide you with a secret key. That key is unique to your application, and shouldn't be shared with anyone else. You'll need it to request user tokens.

What follows are the steps to obtain a user token after registering an application. If you are using an OAuth2 library, many of these steps will be handled for you.

To authorize your application for JobTrack API access, you will need to accomplish the following steps for each JobTrack user:

1. Request a short-term code, granted when the JobTrack user agrees to allow your application access.

   Send your user to /oauth/authorize with the REQUIRED parameters client_id, response_type, and redirect_uri.

      * `client_id` is the ID assigned to your application by JobTrack
      * `response_type` must be set to "code"
      * `redirect_uri` must be set to a URL where your application can accept codes and then exchange them for access tokens. It should match the redirect_uri specified when you registered your application.

   Here is an example URL that an application located at "myapp.com" might use. (Linebreaks are not included in the URL.)
   
   ```
   https://api.jobtrack.io/oauth/authorize?response_type=code&client_id=abc123&redirect_uri=http%3A%2F%2Fmyapp.com%2Foauth%2Fcallback
   ```
   

1. The user will be asked by JobTrack if they want to authorize your application to interact with JobTrack on their behalf.

   If something goes wrong (like the user refused to authorize your application), JobTrack will redirect to the `redirect_uri` with query parameters providing information about the error. For example, if authorization is denied, the user will be redirected to:
   
   ```
   $REDIRECT_URI?error=access_denied&error_description=The+resource+owner+or+authorization+server+denied+the+request.
   ```
   
   If the user allows your application, then JobTrack will redirect to the redirect_uri with query parameters providing your application with a time-limited code that your application can exchange for an access token within the next 5 minutes. Here is an example redirection with granted access:
   
   ```
   $REDIRECT_URI?code=abc123
   ```

1. Your application exchanges the code for an access token

   Now that your application has a code, it should make a POST request directly to JobTrack at `https://api.jobtrack.io/oauth/token` to exchange the code for an access token that will allow continued interaction with the JobTrack API. The request must include the `client_id`, `client_secret`, `grant_type`, `code`, and `redirect_uri`parameters.

   * `client_id` is the ID assigned to your application by JobTrack
   * `client_secret` is the secret token assigned to your application by JobTrack
   * `grant_type` must be set to "authorization_code" in order to exchange a code for an access token
   * `code` is the value that was returned in the code query parameter when JobTrack redirected back to your `redirect_uri`
   * `redirect_uri` is the exact same value that you used in the original request to `/oauth/authorize`


   If the request is invalid for some reason, an error response like the one described above will be returned. However, the parameters will be returned in the response body, encoded as JSON, instead of in the URL encoded as query parameters.
   
   If the request is valid, JobTrack will provide a response body, encoded in JSON, containing `access_token` and `token_type`.
   
      * `access_token` is the token that your application will use to authenticate requests to the JobTrack API as this user
      * `token_type` will be "bearer"


1. Your application uses the access token to make authenticated requests to the JobTrack API


To authenticate using the Authorization header, set the header's value to `Bearer <token>`. So, if your token was `abc123`, your HTTP request would include the header `Authorization: Bearer abc123`.

For example, authenticating a request using curl would mean running a command similar to this one:

```
curl -H "Authorization: Bearer abc123" "https://api.jobtrack.io/api/v1/job-applications"
```

All requests to the JobTrack API require an Authorization header. For brevity, future API request examples in this documentation will not include the example Authorization header parameter.

## Endpoints

### Company Masters
A Company Master is a global identifier for a company. In order to create a Company record (see below), a Company Master `id` is required. JobTrack leverages the Glassdoor API to generate a list of Company Master search result objects, which can be queried by name. Company Masters can be created if the record doesn't already exist.

#### `GET company-masters?autocomplete=<name>`
Searching for Company Masters requires the `autocomplete` param equal to the search term. Given a company name, JobTrack will search both the internal database and Glassdoor API to find matching companies. The response is in the following format:

```json
{  
   "data":[  
      {  
         "id":"glassdoor-id-40772",
         "type":"company-masters",
         "attributes":{  
            "created-at":null,
            "updated-at":null,
            "name":"Facebook",
            "url":"www.facebook.com",
            "new":true,
            "glassdoor-id":40772,
            "glassdoor-data":null,
            "image-url":null,
            "industry":null,
            "enriched":null
         }
      }
   ]
}
```

The above response shows an example of a Company Master that doens't exist yet in the JobTrack database. This is indicated by the `"new": true` attribute and the `id` attribute containing "glassdoor-id-".

A Company Master that exists in the JobTrack database is represented as follows:

```json
{  
   "data":[  
      {  
         "id":"18",
         "type":"company-masters",
         "attributes":{  
            "name":"Isobar",
            "url":"www.isobar.com",
            "new":false,
            "glassdoor-id":334219,
            "glassdoor-data":null,
            "image-url":null,
            "industry":null,
            "enriched":null
         }
      }
   ]
}
```

Note that in the above example, `new` is `false` and the `id` is an integer.

#### Creating a Company Master
If the results of `autocomplete` indicate that the Company Master hasn't yet been created in JobTrack, a `POST` request must be issued to the `company-masters` endpoint to create the record. The following attributes are supported:
* `name` (required)
* `url`
* `glassdoor-id` (encouraged, can be retrieved from result of `autocomplete` operation)

The reponse of this `POST` operation includes an `id` that is required for the next step of the process: creating the Company record


### Companies
#### `GET companies`

Filters:

* `name` - The name of the company

#### `GET companies/<id>`
#### `POST companies`

Relationships:
* `company-master`


### Contacts
#### `GET contacts`

Filters:

* `name` - The first or last name of the contact
* `company_id`

#### `GET contacts/<id>`
#### `POST contacts`

Attributes:
* `first_name`
* `last_name`
* `work_phone`
* `personal_phone`
* `email`
* `notes`
* `job_title`
* `introduction_notes`
* `linkedin_url`
* `linkedin_id`

#### `PUT contacts/<id>`

Attributes:
* Same as `POST`

### Events
#### `GET events`

Filters:

* `creatable_by_user`
* `type` - The type of the event, e.g. `Events::Interview`
* `user_id` - Show events for a particular user
* `future` - Only show events in the future
  * When `true`, only future events are returned
  * When `false`, all events are returned

#### `GET events/<id>`
#### `POST events`

Attributes:
* `type` - one of `Events::Interview`, `Events::PhoneCall`
* `occurs-at`

Relationships:

* `user`
* `subject` - `subject_type` must be `JobApplication`, `subject_id` the `JobApplication`'s `id`  

#### `PUT events/<id>`

Attributes:
* `occurs-at`


### Job Applications
#### `GET job-applications`

Filters:

* `display_name` - The job title or company name of the job applications

#### `GET job-applications/<id>`
#### `POST job-applications`

Attributes:
* `job-title`
* `status` - one of `interested`, `applied`, `phone_call`, `assignment`, `interview`, `offer`, `accepted`, `withdrawn`, `expired`, `not_a_fit`
* `location`
* `notes`
* `job-description`
* `url`
* `date-applied` - ISO8601 formatted date with timezone
* `starred` - boolean
* `source` - sub-object with the following attributes:
  * key
  * label
  * code
  * color
 
Relationships:
* `company`
Assigning a company to a job application is a multi-step process that requires creating (or retrieving) both a Company Master and a Company. 

    ##### Company Master vs Company
    A Company Master is a global record in JobTrack that holds metadata such as URL, Industry and Glassdoor reviews.
    A Company is a link record between a Company Master and a User. Each user saving jobs for the same company will receive distinct Company records.

    See [Creating a Company Master](#creating-a-company-master) for instructions on how to create the necessary records.


* `user`

* `contacts`
* `documents`
* `events`

#### `PUT job-applications/<id>`

Attributes:
* Same as `POST`

### Tasks
#### `GET tasks`

Filters:

* `include_completed` - Show or hide completed tasks. By default
 all tasks are returned.
* `with_due_date` - Filter on presence of due date
  * When `true`, only tasks with due dates are returned
  * When `false`, only tasks _without_ due dates are returned

#### `GET tasks/<id>`
#### `POST tasks`

Attributes:
* `title`
* `detail`
* `due-at`
* `completed`

Relationships:
* `company`
* `contact`
* `job-application`

#### `PUT tasks/<id>`
* Same as `POST`

### Users
#### `GET users`

Attributes:
* `first-name`
* `last-name`
* `email`
* `job-applications-count`
* `role-name`
* `last-seen-at`
