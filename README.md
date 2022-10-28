# Getting Started with Authentication V2 for Refinitiv Real-Time: Overview
- version: Draft
- Last update: October 2022

**Note**: This is a draft version for the public articles on the Developer Portal. For internal use documents, please check the [Article.RTO.AuthenticationV2.Internal](https://github.com/Refinitiv-API-Samples/Article.RTO.AuthenticationV2.Internal) repository. 

## <a id="intro"></a>Introduction

[Refinitiv Data Platform (RDP)](https://developers.refinitiv.com/en/api-catalog/refinitiv-data-platform/refinitiv-data-platform-apis) gives you seamless and holistic access to all of the Refinitiv content (whether real-time or non-real-time, analytics or alternative datasets), commingled with your content, enriching, integrating, and distributing the data through a single interface, delivered wherever you need it. As part of the Refinitiv Data Platform, the Refinitiv Real-Time - Optimized (RTO) gives you access to best in class Real Time market data delivered in the cloud.  Refinitiv Real-Time - Optimized is a new delivery mechanism for RDP, using the AWS (Amazon Web Services) cloud.

The RTO utilizes the RDP authentication service to obtain Access Token information. The RDP authentication version 2 is a newly introduced authentication service for RTO. It is based on industry standard [OAuth 2.0 - Client Credentials model](https://www.oauth.com/oauth2-servers/access-tokens/client-credentials/) with lot of updates, changes, and benefits over V1 for the RTO users. This document aims for helping developers to understand the Authentication V2 overview and workflow in general. 

This article is focusing on Real-Time users which is a machine-to-machine case only.

## <a id="intro_rdp_auth"></a>Introduction to RDP Authentication Service

The first step of an application workflow is to get a token from RDP Authentication Service, which will allow access to the protected resource, i.e. data REST API, streaming services, etc. Once a valid token is received, this token is sent with every REST API calls to get data. For the Real-Time Streaming service, this token must be sent when the application logins to the streaming server on the cloud.

Refinitiv Data Platform (RDP) entitlement check is based on OAuth 2.0 specification. The RDP Authentication Service version 1 (or simply V1) uses the[Password Grant](https://www.oauth.com/oauth2-servers/access-tokens/password-grant/) and [Refresh Token Grant](https://www.oauth.com/oauth2-servers/access-tokens/refreshing-access-tokens/) models to get the first set of tokens and renew subsequent tokens respectively. 

On the other hand, the RDP Authentication Service version 2 (or simply V2) uses the oAuth2.0 [Client Credentials Grant](https://www.oauth.com/oauth2-servers/access-tokens/client-credentials/) model for getting token information. So, what the Client Credentials Grant Model is?

## <a id="intro_client_credential"></a>What is OAuth 2.0 - Client Credentials Grant Model?

The Client Credentials grant is used when applications request an access token to access their resources, not on behalf of a user.  This flow is the golden standard for machines-to-machines communication. The client credentials grant type allows an application to obtain an access token for resources owned by the client or when authorization has been *“previously arranged with an authorization server.”* This grant type is appropriate for applications that need to access APIs, such as storage services or databases, on behalf of themselves rather than on behalf of a specific user. 

This model uses the ```client_id``` and ```client_secret``` as the client authentication information to authenticate clients for this request. The ```client_id``` is a public identifier for apps, and the ```client_secret``` is the application’s password. The request ```grant_type``` parameter must be set to **client_credentials**. Please find more detail about the Client ID and Secret from the [OAuth 2.0  - The Client ID and Secret](https://www.oauth.com/oauth2-servers/client-registration/client-id-secret/) page.

Example HTTP request message from the [oauth.com][https://www.oauth.com/] website:
``` HTTP
POST /token HTTP/1.1
Host: authorization-server.com
 
grant_type=client_credentials
&client_id=xxxxxxxxxx
&client_secret=xxxxxxxxxx
```
**Note**: The ```Client Credentials  client_id``` **is not the same value** as the ```Password/Refresh Grants client_id``` which is an ```app key```.

If the request for an access token is valid, the authorization server needs to generate an access token and return these to the client, typically along with some additional properties about the authorization.

Example HTTP response message from the [oauth.com][https://www.oauth.com/] website:
``` HTTP
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store
 
{
  "access_token":"YYYYYYYYYYYYY",
  "token_type":"Bearer",
  "expires_in":7199
}
```
That’s all I have to say about the basics of the Client Credentials Model grant model. My next point is the difference between V1 and V2, and what are V2 benefits over V1.

## <a id="v1_v2_summary"></a>Authentication V1 and V2 Differences Summary

1. The API URL (**v1** and **v2**)
2. Credential. The V1 uses Machine Account (username, password, and App-Key), but the V2 uses Service account (Client ID and Client Secret).
3. Grant Type models (V1 - Password/Refresh Grant vs V2 - Client Credentials) and request parameters 
4. Token Response Message between V1 and V2 are different
5. The V2 uses the same Client Credential grant request message for both initial request and re-new the access token.
6. Authentication V2 produces a single Access Token. 

## <a id="v2_benefits"></a>Authentication V2 Benefits Over V1
1. Longer Access Token time (V1's 10 minutes vs V2's 120 minutes).
2. Authentication V2 produces a single Access Token, easy to manage.
3. Simplify an Access Token renewal process
4. The application/API does not need to renew an Access Token (HTTP/RSSL-WebSocket) as long as the streaming channel (WebSocket/RSSL) is active, even if that session time passes the expires_in period. The consumer application only need re-request an Access Token in the following scenarios:
    * When the consumer disconnects and goes into a reconnection state.
    * If the *streaming channel stays in reconnection* long enough to get close to the expiry time of the Access Token.
5. Authentication V2 is highly scalable and resilient infrastructure to support increased user base.
6. Authentication V2 uses standard technologies and protocols. 

That covers a brief introduction of the Authentication V2.

## <a id="v2_detail"></a>Authentication V2 in Details 

Moving on to the technical detail of the Authentication V2. It is part of the RDP HTTP Web-based API that provides authentication service for users and applications via the Request - Response RESTful web service delivery mechanism. In general, the V2 is based on the [OAuth 2.0 - Client Credentials](https://www.oauth.com/oauth2-servers/access-tokens/client-credentials/) model. The API endpoint, credentials, request parameters, and response messages are not compatible with the Authentication V1.

The authentication V2 HTTP workflow is like the following diagram.

![figure-1](images/01_auth_v2_http_flow.png "Authentication V2 HTTP Workflow")


### API URL

The Authentication V2 endpoint URL is **http://api.refinitiv.com/auth/oauth2/v2/token**. 

Please be noticed that is the API version is **v2** (the Authentication V1 uses **v1**).

### HTTP Method

Authentication V2 supports the *HTTP POST* method.

### Content-Type

Authentication V2 supports the *application/x-www-form-urlencoded* HTTP Content-Type.

### Request Parameters and Credentials

The request parameters of V1 and V2 are different. The Authentication V2 requires a Service account which consisting client ID (*not the App Key*) and client Secret credential information for the request parameter.

When you log into the Refinitiv Data Platform (either initial connection or re-new), you must use a ```grant_type``` of **client_credentials** to to get access token information.

The Authentication V2 requires the following access credential information in the HTTP request parameters:
- **grant_type**: The grant_type parameter must be set to **client_credentials**.
- **client_id**:  The ```client_id``` is a public identifier for apps.
- **client_secret**:  The ```client_secret``` is a secret known only to the application and the authorization server. It is essential to the application’s password associated with the client ID.
- **scope (optional)**: Limits the scope of the generated token so that the Access token is valid only for a specific data set

**Note**: The ```V2 client_id``` **is not the same value** as the ```V1 client_id```. The ```V1 client_id``` is an ```app key``` of the [V1 - Password Grant Model](https://www.oauth.com/oauth2-servers/access-tokens/password-grant/).

Example:
``` HTTP
POST /auth/oauth2/v2/token HTTP/1.1
Accept: */*
Content-Type: application/x-www-form-urlencoded
Host: api.refinitiv.com:443
Content-Length: XXX

client_id=KKKKKKKKKKKKKKKKKK
&client_secret=BBBBBBBBBBBBBBBBBBBB
&grant_type=client_credentials
&scope=trapi
```

### Response Message

Once the authentication is successful, the function gets the RDP Auth service response message and keeps the following RDP token information in the variables.
- **access_token**: The token used to invoke REST data API calls as described above. The application must keep this credential for further RDP APIs requests.
- **expires_in**: Access token validity time in seconds.
- **token_type (required)**: The type of token this is, typically just the string “Bearer”.

Example:
``` JSON
{
  "expires_in": 7199,
  "token_type": "Bearer",
  "access_token": "YYYYYYYYYYYYY"
}
```
### Renew Access Token

The Authentication V2 does not use a refresh grant logic. If the application needs a refreshing token, the application can just re-send a new authentication request (with ```grant_type``` of **client_credentials** ) to the RDP endpoint.


### Comparing with the Authentication V1

The Authentication V1 uses the Password Grant model request for requesting initial token, then uses Refresh Grant model request for renewing the access token. The V1 also produces multiple types of token for requesting data and renewal process. It means the application and API needs to manage difference type of request messages in the same application.

The V1 workflow is shown below.

![figure-2](images/02_auth_v1_http_flow.png "Authentication V1 HTTP Workflow")

You see that the V1 HTTP operation workflow is more complex than the V2.

That’s all I have to say about the RDP Authentication Service Version 2 HTTP workflow.

## Authentication V2 - Connecting to Real-Time Streaming

TBD

## <a id="references"></a>References

For further details, please check out the following resources:
* [OAuth 2.0 - Client Credentials](https://www.oauth.com/oauth2-servers/access-tokens/client-credentials/) page.
* [OAuth 2.0 - Access Token Response](https://www.oauth.com/oauth2-servers/access-tokens/access-token-response/) page.
* [OAuth 2.0 - Password Grant](https://www.oauth.com/oauth2-servers/access-tokens/password-grant/) page.
* [OAuth 2.0 - Client Credentials Grant](https://oauth.net/2/grant-types/client-credentials/) page.