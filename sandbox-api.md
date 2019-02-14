# Reco Sandbox API Documentation

The Sandbox API consist of two different types of APIs.
The standard [reco-api-v3](https://gitlab.com/reco.se-api/docs/blob/master/reco-api-v3.md) and 
the [partner-api](https://gitlab.com/reco.se-api/docs/blob/master/partner-api.md). If you are not sure
which one to use please contact [kundtjanst@reco.se](mailto:kundtjanst@reco.se)
and they will get you sorted out. 

Also you will need to provide a sandbox email, that will receive all emails and a phone number
that will receive all sms when in sandbox, to get a Sandbox API key.
Please include it when requestion the key from [kundtjanst@reco.se](mailto:kundtjanst@reco.se)

# 1. Reco API V3

## Create email invite

To create invites you need to POST a request to: 

```
https://api.reco.se/sandbox/invite/mail/venue/{venue-id}
```

Reco's customer service [kundtjanst@reco.se](mailto:kundtjanst@reco.se) will help you with what `venue-id` belongs to your company.

#### Required Headers 

| Header        | Value            | Description |
| ------------- | ---------------- | ----- |
| X-Reco-ApiKey |                  | The API-key that you received from Reco.se. Please contact kundtjanst@reco.se if you do not have an API-key. |
| Content-Type  | application/json |  |

#### POST Payload

The data sent in the request should look like:

```
{
  "invites": [{
    "email": "email@example.org",
    "firstName": "John",
    "lastName": "Doe"
  }]
}
```

It's possible to send multiple invites in one request

#### Example call

```
curl -v \
  -XPOST \
  -H "Content-Type: application/json" \
  -H "X-Reco-ApiKey: sandbox-api-key \
  "https://api.reco.se/sandbox/invite/mail/venue/123" \
  -d '{
    "invites": [{
      "email": "email@example.org",
      "firstName": "John",
      "lastName": "Doe"
     }]
  }'
```

#### Example answer

If the request is successful the API will anser with status `200 OK`.

```
{
  "errors": [ {
    "errorCode": 4,
    "email": "john.doe@example.org",
    "firstName": "John",
    "lastName": "Doe"
  } ],
  "success": [{
    "email": "jane.doe@example.org",
    "firstName": "Jane",
    "lastName": "Doe"
  }]
}
```

If the API-key is wrong the API will answer with status `403 Forbidden`.

### Send with metadata

It is possible to send metadata with the invites. E.g to categorize the reviews later on.

```
{
  "invites": [
    {
      "email": "email@example.org",
      "firstName": "John",
      "lastName": "Doe",
      "metadata": [{
      	"key": "customerId",
      	"value": "13348fa8-f53e-4b1b-91f5-0383f454f58d"
      }, {
      	"key": "category",
      	"value": "books"
      }]
    }
  ]
}
```

### Send with scheduled

If you want to postpone the invites you include sendDateFrom in a scheduled object with the request.
This will tell reco from when it's allowed to send the invites. Reco will then decide
the best day and time to send out the actual invites.

```
curl -v \
  -XPOST \
  -H "Content-Type: application/json" \
  -H "X-Reco-ApiKey: sandbox-api-key \
  "https://api.reco.se/sandbox/invite/mail/venue/123" \
  -d '{
    "invites": [{
      "email": "email@example.org",
      "firstName": "John",
      "lastName": "Doe"
     }],
     "scheduled": {
      "sendDateFrom":"2018-01-20"
    }
  }'
```

`customerId` i used to identify the customer or the order, so that Reco can later on measure how my of your clients have been contacted about writing reviews.

#### Error codes

Error codes could be read from `errors[].errorCode` in the response from the API:

| Kod | Description |
| --- | ----------- |
| 1   | Internal error |
| 2   | Wrong input params |
| 3   | The customer have already been asked to review |
| 4   | The customer has opt-out from receiving review requests from Reco.se |

### Register customer ids

To register cuomster ids you call:

```
https://api.reco.se/sandbox/v3/customerids/{venue-id} -H 'Content-Type: application/json'
```
Reco's customer service [kundtjanst@reco.se](mailto:kundtjanst@reco.se) will help you with what `venue-id` belongs to your company.

#### Required Headers

| Header        | Value            | Description |
| ------------- | ---------------- | ----- |
| X-Reco-ApiKey |                  | The API-key that you received from Reco.se. Please contact kundtjanst@reco.se if you do not have an API-key. |
| Content-Type  | application/json |  |

#### POST Payload

The data sent in the request should look like:

```
{
  "ids": [
    "b75dc04a-e42c-4820-bfb6-9be818add74d",
    "55dc6c60-397d-4c7f-824e-961cf0ef8e6b",
    ...
  ]
}
```

# 2. Partner API

The purpose of Reco Partner API is to make it really simple to integrate with your system or platform.
Instead of using Reco's ids you will use your own internal ids and Reco will take care of the mapping.

### 1. Check if venue is present on Reco.se

Begin by making a request, for every venue you want to create an invite for and check if it's present on Reco.se.

The endpoint consist of the `mappingType` and query following endpoint with the `platform` and platform specific venue `id`:

`GET https://api.reco.se/sandbox/v3/<mappingType>/customer?platform=<platform>&id=<id>`

Parameters:

| Parameter     | Description |
| ------------- | ----------- |
| `mappingType` | The type of invite venue eg. `restaurant` |
| `platform`    | The platform of the mappingType eg: `waiteraid`|
| `id`          | The platform-specific venue id eg: "123" |

An API key is required and is sent by providing a `X-Reco-ApiKey` header.

Example:

```bash
curl -H 'X-Reco-ApiKey: provided-key-by-Reco.se' \
  'https://api.reco.se/sandbox/v3/restaurant/customer?id=bf936053b93bf3e04cff3536a0L0l4&platform=waiteraid'
```

Response is JSON:

```json
{
  "exists": <true | false>
}
```

If `exists` is `false`, then the venue is not present on Reco.se. If that is the case please
email your contact at Reco.se with the data about the venue so that we can create the appropriate mapping.

### 2. Create invites to review for your platform

To create invites for your platform you need to post a request to: 

`POST https://api.reco.se/sandbox/v3/<mappingType>/invite?platform=<platform>&id=<id>`

Parameters:

| Parameter  | Description |
| ---------- | ----------- |
| `mappingType` | The type of invite venue eg. `restaurant` |
| `platform`    | The platform of the mappingType eg: `waiteraid`|
| `id`          | The platform-specific venue id eg: "123" |

`POST` body needs to be in `JSON` format, i.e. `Content-Type: application/json` needs to be present.  
You can send multiple contacts to invite them to review the venue.


#### Example call

```
curl -v \
  -XPOST \
  -H "Content-Type: application/json" \
  -H "X-Reco-ApiKey: provided-key-by-Reco.se \
  "https://api.reco.se/sandbox/v3/<mappingType>/invite?platform=<platform>&id=<id>" \
  -d '{
    "invites": [{
      "email": "email@example.org",
      "firstName": "John",
      "lastName": "Doe"
     }]
  }'
```

#### Example answer

If the request is successful the API will anser with status `200 OK`.

```
{
  "errors": [ {
    "errorCode": 4,
    "email": "john.doe@example.org",
    "firstName": "John",
    "lastName": "Doe"
  } ],
  "success": [{
    "email": "jane.doe@example.org",
    "firstName": "Jane",
    "lastName": "Doe"
  }]
}
```

If the API-key is wrong the API will answer with status `403 Forbidden`.

### 3. Send with metadata

It is possible to send metadata with the invites. E.g to categorize the reviews later on.

```
{
  "invites": [
    {
      "email": "email@example.org",
      "firstName": "John",
      "lastName": "Doe",
      "metadata": [{
        "key": "funeralFlowers",
        "value": "true",
        "type": "text"
      },
      {
        "key": "funeralBookingDate",
        "value": "2018-05-25T19:00:00",
        "type": "ts"
      },
      {
        "key": "funeralDate",
        "value": "2018-10-25T19:00:00",
        "type": "ts"
      },
      {
        "key": "cost",
        "value": "5995",
        "type": "num"
          
      }
    ]
}
```

### 4. Send with scheduled

If you want to postpone the invites you include sendDateFrom in a scheduled object with the request.
This will tell reco from when it's allowed to send the invites. Reco will then decide
the best day and time to send out the actual invites.

```
curl -v \
  -XPOST \
  -H "Content-Type: application/json" \
  -H "X-Reco-ApiKey: provided-key-by-Reco.se \
  "https://api.reco.se/sandbox/v3/<mappingType>/invite?platform=<platform>&id=<id>" \
  -d '{
    "invites": [{
      "email": "email@example.org",
      "firstName": "John",
      "lastName": "Doe"
     }],
     "scheduled": {
      "sendDateFrom":"2018-01-20"
    }
  }'
```

#### Error codes

Following errors could be found in `errors[].errorCode`:

| Code | Description |
| --- | ----------- |
| 1   | Internal server error |
| 2   | Wrong params |
| 3   | The contact have already been asked to review |
| 4   | The contact has opt-out from receiving review requests from Reco.se |




