# Getting Started with Authentication V2 for Refinitiv Real-Time: Overview
- version: Beta
- Last update: October 2022

## Introduction

[Refinitiv Data Platform (RDP)](https://developers.refinitiv.com/en/api-catalog/refinitiv-data-platform/refinitiv-data-platform-apis) gives you seamless and holistic access to all of the Refinitiv content (whether real-time or non-real-time, analytics or alternative datasets), commingled with your content, enriching, integrating, and distributing the data through a single interface, delivered wherever you need it. As part of the Refinitiv Data Platform, the Refinitiv Real-Time - Optimized (RTO) gives you access to best in class Real Time market data delivered in the cloud.  Refinitiv Real-Time - Optimized is a new delivery mechanism for RDP, using the AWS (Amazon Web Services) cloud.

The RTO utilizes the RDP authentication service to obtain Access Token information. The RDP authentication service version 2 (or simply V1) is based on oAuth2.0 [Password Grant](https://www.oauth.com/oauth2-servers/access-tokens/password-grant/) and [Refresh Token Grant](https://www.oauth.com/oauth2-servers/access-tokens/refreshing-access-tokens/) models. 

The RDP authentication version 2 (or simply V2) is a newly introduced authentication service for RTO. It is based on industry standard [OAuth 2.0 - Client Credentials model](https://www.oauth.com/oauth2-servers/access-tokens/client-credentials/). The V2 has a lot of updates, changes, and benefits over V1 for the RTO users. This document aims for helping developers to understand the Authentication V2 overview and workflow in general. 

## <a id="references"></a>References

For further details, please check out the following resources:
* [OAuth 2.0 - Client Credentials](https://www.oauth.com/oauth2-servers/access-tokens/client-credentials/) page.
* [OAuth 2.0 - Access Token Response](https://www.oauth.com/oauth2-servers/access-tokens/access-token-response/) page.
* [OAuth 2.0 - Password Grant](https://www.oauth.com/oauth2-servers/access-tokens/password-grant/) page.
* [OAuth 2.0 - Client Credentials Grant](https://oauth.net/2/grant-types/client-credentials/) page.