# Getting Started with Authentication V2 for Refinitiv Real-Time: Overview
- version: Draft
- Last update: October 2022

**Note**: This is a draft version for the public articles on the Developer Portal. For internal use documents, please check the [Article.RTO.AuthenticationV2.Internal](https://github.com/Refinitiv-API-Samples/Article.RTO.AuthenticationV2.Internal) repository. 

## <a id="intro"></a>Introduction

[Refinitiv Data Platform (RDP)](https://developers.refinitiv.com/en/api-catalog/refinitiv-data-platform/refinitiv-data-platform-apis) gives you seamless and holistic access to all of the Refinitiv content (whether real-time or non-real-time, analytics or alternative datasets), commingled with your content, enriching, integrating, and distributing the data through a single interface, delivered wherever you need it. As part of the Refinitiv Data Platform, the Refinitiv Real-Time - Optimized (RTO) gives you access to best in class Real Time market data delivered in the cloud.  Refinitiv Real-Time - Optimized is a new delivery mechanism for RDP, using the AWS (Amazon Web Services) cloud.

The RTO utilizes the RDP authentication service to obtain Access Token information. The RDP authentication version 2 is a newly introduced authentication service for RTO. It is based on industry standard [OAuth 2.0 - Client Credentials model](https://www.oauth.com/oauth2-servers/access-tokens/client-credentials/) with lot of updates, changes, and benefits over V1 for the RTO users. This document aims for helping developers to understand the Authentication V2 overview and workflow in general. 

This article is focusing on the Refinitiv Real-Time - Optimized developers (a machine-to-machine case) which need to migrate their applications to the V2 only.

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

## <a id="v2_http_detail"></a>Authentication V2 HTTP Operation in Details 

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

## <a id="service_discovery"></a>Refinitiv Real-Time Service Discovery

That brings us to the next RDP service, the Service Discovery.

After you obtain an authentication token, depending on its authorization and scope, you can use it to retrieve content from the RDP and connect to the RTO Streaming server. To connect to RTO Streaming server, one can either specify an endpoint (VIP) or discover the list of endpoints (VIP) using a Refinitiv Data Platform service called *Service Discovery*.

To retrieve VIPs, application must call the url **https://api.refinitiv.com/streaming/pricing/v1/** API endpoint with the access token from the RDP Authentication Service in the request message header. The Service Discovery supports both V1 and V2 access tokens with the same HTTP GET request message.

Example HTTP GET request message.

``` HTTP
GET /streaming/pricing/v1/ HTTP/1.1
Accept: */*

Authorization: Bearer <Access Token from either V1 or V2>
Host: api.refinitiv.com:443
```
The Service Discovery response message structure is the same.

## <a id="v2_streaming_detail"></a>Authentication V2 - Connecting to Real-Time Streaming

My next point is the Real-Time streaming connection workflow with the Authentication V2. 

One major benefit of Authentication V2 over V1 is the application/API does not need to renew the access token as long as the streaming channel (either WebSocket or RSSL connection) is actives. This makes the Real-Time connection workflow of Authentication V1 and V2 difference beside the changes between the HTTP operations with RDP Authentication service V1 and V2.

The workflows summaries are as follows:

### Authentication V2 Streaming Workflow

1. Send an HTTP Post message with a Client Credentials Grant to RDP API Authentication V2 service and obtain an access token and session expiration interval for your application 
2. Send HTTP Get message to with access token RDP API Service Discovery and obtain a list of  Real Time in Cloud Streaming servers.
3. Connect to the desired RTO Streaming server.
4. Send a login message to the RTO Streaming server with an access token.
5. Send item request messages to the RTO Streaming server.
6. Handle asynchronous responses and status changes when they occur, including “pings” and connection failures.
7. Once connected to the RTO Streaming server, there is no need to renew the Access Token. The login session to the RTO Streaming Server will remain valid (even if the connection time passes the Access Token expiry period) until the consumer disconnects or is disconnected from the RTO Streaming server. The consumer application only need re-request an Access Token in the following scenarios:
    * When the consumer disconnects and goes into a reconnection state.
    * If the *streaming channel stays in reconnection* long enough to get close to the expiry time of the Access Token.
8. Re-authenticate with RDP API Authentication V2 service using a Client Credentials Grant.
9. Attempt re-connect/re-issue login request to RTO Streaming server endpoint with the access token from ```step 8``` 
10. If the streaming channel reconnection process (```step 9```) gets close to the expires_in time, back to ```step 8``` get a new token for reconnection use.

Summary: 
* As long as the streaming connection (either RSSL or WebSocket) is alive, the application API does not need to re-new Access Token (both RDP HTTP and RSSL/WebSocket streaming).
* Every time the application is disconnected, an application must request a new token to reconnect. If the reconnect attempt fails, then the same token can be re-used until Expires_in to avoid asking for a new token if RTO is down.

![figure-3](images/03_auth_v2_streaming.png "Authentication V2 Streaming Workflow")

### Comparing with the Authentication V1

Unlike the V2, the Authentication V1 needs to re-authentication the RDP Authentication Service version 1 (HTTP REST) before the token is expired. And then re-issue a OMM Login request message (RSSL/WebSocket) with a new access token to keep the streaming channel open.

If you are using the Refinitiv Real-Time SDK, the API automatically handles this token renewal process for you. However, the Authentication V2 streaming workflow is much simpler for the WebSocket API developers because application does not need to re-new access token as long as it's streaming channel is active.

That covers the Real-Time Streaming connection overview with the RDP Authentication Service version 2.

## <a id="prerequisite"></a>Prerequisite
The RTO Authentication V2 requires the following dependencies.
1. RTO Authentication V2 access credentials (*client_id* and *client_secret*).
2. RTSDK Developers: RTSDK version 2.0.5 (EMA/ETA API version 3.6.5) and above.
3. WebSocket API Developers: The latest version of the WebSocket API examples.
4. Internet connection.

Please contact your Refinitiv representative to help you to access the RTO account and services. 

## <a id="v2_rtsdk"></a>How to use Authentication V2 with Refinitiv Real-Time SDK

Now let me turn to using the Authentication V2 with the Refinitiv Real-Time SDK (RTSDK) developers. The RTSDK [C/C++](https://developers.refinitiv.com/en/api-catalog/refinitiv-real-time-opnsrc/rt-sdk-cc) and [Java](https://developers.refinitiv.com/en/api-catalog/refinitiv-real-time-opnsrc/rt-sdk-java) editions already support the Authentication V2 since version 2.0.5 (EMA/ETA API version 3.6.5). 

This article is based on RTSDK version 2.0.7.L1 (EMA/ETA API version 3.6.7).

### <a id="v2_ema"></a>EMA API

The EMA API supports Authentication V2 since the API version **3.6.5.0** (RTSDK version **2.0.5**). The demo and example code is available in the following EMA example applications:
* ```ex113_MP_SessionMgmt``` (both V1 and V2), ```ex450_MP_QueryServiceDiscovery``` (both V1 and V2), and ```ex451_MP_OAuth2Callback_V2``` (V2 only) examples of EMA Java API
* ```Cons113``` (both V1 and V2), ```Cons450``` (both V1 and V2), and  ```Cons451``` (V2 only) examples of EMA C/C++ API. 

The EMA API automatically operates the HTTP and streaming connections workflow for the application. However, developers need to pass the V2 client_id and client_secret credentials to the API, and use newly introduce methods/interfaces for Authentication V2.

The rest of the code logic is the same.

For more detail about using the Authentication V2 with the Enterprise Message API, please check the upcoming *Getting Started with Authentication V2 using Enterprise Message API* article (TBD).

#### EMA API Authentication V2 - Quick Start

My next point is the EMA examples quick start. This article is demonstrating with the ```ex450_MP_QueryServiceDiscovery``` and ```Cons450``` examples.

EMA Java ```ex450_MP_QueryServiceDiscovery```:
``` Bash
$>gradlew.bat runconsumer450 --args="-clientId <ClientID> -clientSecret <ClientSecret> -itemName <RIC name>"
```

![figure-4](images/04_emaj_450_run_result.gif "EMA Java example 450 result")

EMA C/C++ ```Cons450```:
``` Bash
$>Cons450 -clientId <ClientID> -clientSecret <ClientSecret> -itemName <RIC name>
```

![figure-5](images/05_emacpp_450_run_result.gif)

## <a id="references"></a>References

For further details, please check out the following resources:
* [OAuth 2.0 - Client Credentials](https://www.oauth.com/oauth2-servers/access-tokens/client-credentials/) page.
* [OAuth 2.0 - Access Token Response](https://www.oauth.com/oauth2-servers/access-tokens/access-token-response/) page.
* [OAuth 2.0 - Password Grant](https://www.oauth.com/oauth2-servers/access-tokens/password-grant/) page.
* [OAuth 2.0 - Client Credentials Grant](https://oauth.net/2/grant-types/client-credentials/) page.