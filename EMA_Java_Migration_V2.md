# Getting Started with Authentication V2 for Refinitiv Real-Time using Enterprise Message Java API
- version: Draft
- Last update: Nov 2022

**Note**: This is a draft version of the public articles on the Developer Portal. For internal use documents, please check the [Article.RTO.AuthenticationV2.Internal](https://github.com/Refinitiv-API-Samples/Article.RTO.AuthenticationV2.Internal) repository. 

## <a id="intro"></a>Introduction

[Refinitiv Data Platform (RDP)](https://developers.refinitiv.com/en/api-catalog/refinitiv-data-platform/refinitiv-data-platform-apis) gives you seamless and holistic access to all of the Refinitiv content (whether real-time or non-real-time, analytics or alternative datasets), commingled with your content, enriching, integrating, and distributing the data through a single interface, delivered wherever you need it. As part of the Refinitiv Data Platform, the Refinitiv Real-Time - Optimized (RTO) gives you access to best-in-class Real-Time market data delivered in the cloud.  Refinitiv Real-Time - Optimized is a new delivery mechanism for RDP, using the AWS (Amazon Web Services) cloud.

The RTO utilizes the RDP authentication service to obtain Access Token information. The RDP authentication version 2 is a newly introduced authentication service for RTO. It is based on industry-standard [OAuth 2.0 - Client Credentials model](https://www.oauth.com/oauth2-servers/access-tokens/client-credentials/) with a lot of updates, changes, and benefits over version 1 for the RTO users. This document provides guidelines to migrate the EMA Java consumer applications to use RDP authentication version 2. 

For more detail about the Authentication V2 overview and concept, please check this [Getting Started with Authentication V2](https://github.com/Refinitiv-API-Samples/Article.RTO.AuthenticationV2) document.

This article is based on RTSDK Java version 2.0.7.L1 (EMA/ETA API version 3.6.7).

## RDP Authentication Service version 2 Summaries

The RDP Authentication Service version 2 (or simply V2) is based on the [OAuth 2.0 - Client Credentials model](https://www.oauth.com/oauth2-servers/access-tokens/client-credentials/) model. The Authentication V2 simplifies the usage of access tokens. The V2 2 will only generate an access token, not both access and refresh tokens. Once connected to the Refinitiv Real-Time Optimized with an access token, there is no need to renew the access Token. The login session will remain valid until the application disconnects or is disconnected from RTO.

### API Endpoint ###

API URL endpoint: **http://api.refinitiv.com/auth/oauth2/v2/token** (please be noticed the **v2** API version)

### Request Parameters

The authentication V2 does not use the **Password Grant/Refresh Grant** model, all connections (initial connection and re-new) use a ```grant_type``` of **client_credentials** to get access token information.

The authentication V2 requires the following access credential information in the HTTP request parameters:
- **grant_type**: The grant_type parameter must be set to **client_credentials**.
- **client_id**:  The ```client_id``` is a public identifier for apps.
- **client_secret**:  The ```client_secret``` is a secret known only to the application and the authorization server. It is essential to have the application’s password associated with the client ID.
- **scope (optional)**: Limits the scope of the generated token so that the Access token is valid only for a specific data se

**Note**: The ```V2 client_id``` **is not the same value** as the ```V1 client_id``` which is an ```app key``` of the [V1 - Password Grant Model](https://www.oauth.com/oauth2-servers/access-tokens/password-grant/).

## <a id="prerequisite"></a>Prerequisite
The Authentication V2 requires the following dependencies.
1. Authentication credential (*client_id* and *client_secret*).
2. RTSDK Developers: RTSDK version 2.0.5 (EMA/ETA API version 3.6.5) and above.
3. Internet connection.

Please contact your Refinitiv representative to help you to access the RTO account and services. 

## RDP Authentication Version 2 Refinitiv Real-Time SDK Java Code Migration 

The Refinitiv Real-Time SDK (RTSDK) [C/C++](https://developers.refinitiv.com/en/api-catalog/refinitiv-real-time-opnsrc/rt-sdk-cc) and [Java](https://developers.refinitiv.com/en/api-catalog/refinitiv-real-time-opnsrc/rt-sdk-java) editions already support the Authentication V2 since version 2.0.5 (EMA/ETA API version 3.6.5). 

The EMA Java API automatically operates the RTO HTTP and streaming connections workflow for the application. However, developers need to pass the V2 client_id and client_secret credentials to the API, and use newly introduced Authentication V2 methods/interfaces for connecting to the RTO. The following parts in the code must be modified to migrate the EMA Java applications to use RDP authentication version 2.

### 1. Setting RDP authentication version 2 credentials to the API

TBD


## <a id="references"></a>References

For further details, please check out the following resources:
* [Getting Started with Authentication V2](https://github.com/Refinitiv-API-Samples/Article.RTO.AuthenticationV2) document.
* [Refinitiv Real-Time SDK Family](https://developers.refinitiv.com/en/use-cases-catalog/refinitiv-real-time) page.
* [Refinitiv Real-Time SDK C/C++ page](https://developers.refinitiv.com/en/api-catalog/refinitiv-real-time-opnsrc/rt-sdk-cc) on the [Refinitiv Developer Community](https://developers.refinitiv.com/) website.
* [Refinitiv Real-Time SDK Java page](https://developers.refinitiv.com/en/api-catalog/refinitiv-real-time-opnsrc/rt-sdk-java).
* [Refinitiv WebSocket API page](https://developers.refinitiv.com/en/api-catalog/refinitiv-real-time-opnsrc/refinitiv-websocket-api).
* [OAuth 2.0 - Client Credentials](https://www.oauth.com/oauth2-servers/access-tokens/client-credentials/) page.
* [OAuth 2.0 - Access Token Response](https://www.oauth.com/oauth2-servers/access-tokens/access-token-response/) page.
* [OAuth 2.0 - Password Grant](https://www.oauth.com/oauth2-servers/access-tokens/password-grant/) page.
* [OAuth 2.0 - Client Credentials Grant](https://oauth.net/2/grant-types/client-credentials/) page.