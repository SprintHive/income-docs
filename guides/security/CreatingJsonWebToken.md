# Creating a Json Web Token (JWT)

## Prerequisites

As part of the onboarding process you should have received an email from SprintHive with the following details:

* API key
* Client Id
* Client Secret
* Authentication Endpoint
* Audience

## API Key

Your system integrating with these environments will require a system API key for each environment. 
The API key should be included in an HTTP header called apikey on all API requests.

## Calling the authentication endpoint

An example of what the curl command:
```shell
curl --location --request POST '<authentication_endpoint>' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=client_credentials' \
--data-urlencode 'client_id=<client_id>' \
--data-urlencode 'client_secret=<client_secret>' \
--data-urlencode 'audience=<audience>'
```

This request returns an access_token which must be added to the Authorization header of API requests 
after the Bearer prefix. E.g. Authorization: Bearer <access_token>. 

The response also contains an expires_in field
that reports the number of seconds for which the access_token is valid. It is *critically* important to cache the
access_token for this period of time before a new token is obtained;  
the number of tokens you can request has a limit which will likely be breached if the token is not cached.

> NB: A high volume of new token requests will be treated as a high 
> severity incident for SprintHive and may result in new token requests from being blocked.

