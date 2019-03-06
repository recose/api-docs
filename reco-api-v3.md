---
title: Untitled Page
---
# Reco API v3 Documentation

## Create email invite

To create an email invite call:

```
https://api.reco.se/invite/mail/venue/{venue-id}
```

Reco's customer service ([kundtjanst@reco.se](mailto:kundtjanst@reco.se)) will help you with what `venue-id` belongs to your company.

### Required Headers 

| Header        | Värde            | Beskrivning |
| ------------- | ---------------- | ----- |
| X-Reco-ApiKey |                  | The API-key that you received from Reco.se. Please contact [kundtjanst@reco.se](mailto:kundtjanst@reco.se) if you do not have an API-key. |
| Content-Type  | application/json |  |

### POST Payload

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

### Email example call

```
curl -v \
-XPOST \
-H "Content-Type: application/json" \
-H "X-Reco-ApiKey: 3x3mp3l \
"https://api.reco.se/invite/mail/venue/123" \
-d '{
	"invites": [{
		"email": "email@example.org",
		"firstName": "John",
		"lastName": "Doe"
	 }]
}'
```

### Email example answer

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

## Send with metadata

It is possible to send metadata with the invites e.g to categorize the reviews later on.

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

`customerId` is used to identify the customer or the order, so that Reco can later on measure how my of your clients have been contacted about writing reviews.

## Schedule invites

If you want to postpone the invites you include `sendDateFrom` in a scheduled object with the request.
This will tell reco from when it's allowed to send the invites. Reco will then decide
the best day and time to send out the actual invites.

```
curl -v \
-XPOST \
-H "Content-Type: application/json" \
-H "X-Reco-ApiKey: 3x3mp3l \
"https://api.reco.se/invite/mail/venue/123" \
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

## Create sms invite

To create a sms invite call:

```
https://api.reco.se/invite/sms/venue/{venue-id}
```
#### Sms example call

```
curl -v \
-XPOST \
-H "Content-Type: application/json" \
-H "X-Reco-ApiKey: 3x3mp3l \
"https://api.reco.se/invite/sms/venue/123" \
-d '{
	"invites": [{
		"mobile": "0730000000",
		"firstName": "John",
		"lastName": "Doe"
		"metadata": [{
			"key": "customerId",
			"value": "13348fa8-f53e-4b1b-91f5-0383f454f58d"
		}, {
			"key": "category",
			"value": "books"
		}]
	 }]
}'
```

## Error codes

Error codes could be read from `errors[].errorCode` in the response from the API:

| Kod | Description |
| --- | ----------- |
| 1   | Internal error |
| 2   | Wrong input params |
| 3   | The customer have already been asked to review |
| 4   | The customer has opt-out from receiving review requests from Reco.se |

## Register customer ids

To register cuomster ids you call:

```
https://api.reco.se/v3/customerids/{venue-id} -H 'Content-Type: application/json'
```
Reco's customer service ([kundtjanst@reco.se](mailto:kundtjanst@reco.se)) will help you with what `venue-id` belongs to your company.

### Required Headers

| Header        | Värde            | Beskrivning |
| ------------- | ---------------- | ----- |
| X-Reco-ApiKey |                  | The API-key that you received from Reco.se. Please contact kundtjanst@reco.se if you do not have an API-key. |
| Content-Type  | application/json |  |

### POST Payload

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

