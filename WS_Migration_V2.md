# Getting Started with V2 Authentication for Refinitiv Real-Time using Refinitiv WebSocket API with its Migration Guide
- version: Draft
- Last update: November 2022

**Note**: This is a draft version of the public articles on the Developer Portal. For internal use documents, please check the [Article.RTO.AuthenticationV2.Internal](https://github.com/Refinitiv-API-Samples/Article.RTO.AuthenticationV2.Internal) repository. 

## Introduction
[Refinitiv Data Platform (RDP)](https://developers.refinitiv.com/en/api-catalog/refinitiv-data-platform/refinitiv-data-platform-apis) gives you seamless and holistic access to all of the Refinitiv content (whether real-time or non-real-time, analytics or alternative datasets), commingled with your content, enriching, integrating, and distributing the data through a single interface, delivered wherever you need it. As part of the Refinitiv Data Platform, the **Refinitiv Real-Time - Optimized (RTO)** gives you access to best-in-class Real-Time market data delivered in the cloud.  Refinitiv Real-Time - Optimized is a new delivery mechanism for RDP, using the AWS (Amazon Web Services) cloud.

The RTO utilizes the RDP authentication service to obtain Access Token information. The RDP Version 2 Authentication is a newly introduced authentication service for RTO. It is based on industry-standard [OAuth 2.0 - Client Credentials model](https://www.oauth.com/oauth2-servers/access-tokens/client-credentials/) with a lot of updates, changes, and benefits over version 1 for the RTO users, such as, to generate an access token, instead of access and refresh tokens and once connected to the Refinitiv Real-Time Optimized with an access token, there is no need to renew the access token as the login session will remain valid until the application disconnects or is disconnected from RTO. While the V1 Authentication uses the oAuth2.0 [Password Grant](https://www.oauth.com/oauth2-servers/access-tokens/password-grant/) and [Refresh Token Grant](https://www.oauth.com/oauth2-servers/access-tokens/refreshing-access-tokens/).

The V2 Authentication is a new authentication service for Refinitiv Real-Time Optimized (RTO). This document aims for helping developers to understand how to use the WebSocket API with the V2 Authentication. For more detail about the V2 Authentication overview and concept, please check this [Getting Started with V2 Authentication](./README.md) document.


## V2 Authentication Summaries

The V2 Authentication is based on the [OAuth 2.0 - Client Credentials model](https://www.oauth.com/oauth2-servers/access-tokens/client-credentials/) model with the following details:

### API Endpoint ###

API URL endpoint: **http://api.refinitiv.com/auth/oauth2/v2/token** (please be noticed the **v2** API version)

### Request Parameters

The V2 Authentication does not use the **Password Grant/Refresh Grant** model, all connections (initial connection and re-new) use a ```grant_type``` of **client_credentials** to get access token information.

The V2 Authentication requires the following access credential information in the HTTP request parameters:
- **grant_type**: The grant_type parameter must be set to **client_credentials**.
- **client_id**:  The ```client_id``` is a public identifier for apps.
- **client_secret**:  The ```client_secret``` is a secret known only to the application and the authorization server. It is essential to have the application’s password associated with the client ID.
- **scope (optional)**: Limits the scope of the generated token so that the Access token is valid only for a specific data set

**Note**: The ```V2 client_id``` **is not the same value** as the ```V1 client_id``` which is an ```app key``` of the [V1 - Password Grant Model](https://www.oauth.com/oauth2-servers/access-tokens/password-grant/).

### Benefits of V2 Auth Over the V1 one
1. Longer Access Token time (V1: 10 minutes vs V2: 120 minutes).
2. Version 2 Authentication produces a single Access Token, which is easier to manage.
3. Simplify an Access Token renewal process
4. The application/API does not need to renew an Access Token (HTTP/RSSL-WebSocket) as long as the streaming channel (WebSocket/RSSL) is active, even if that session time passes the expires_in period. The consumer application only need re-request an Access Token in the following scenarios:
    * When the consumer disconnects and goes into a reconnection state.
    * If the *streaming channel stays in reconnection* long enough to get close to the expiry time of the Access Token.
5. Version 2 Authentication is a highly scalable and resilient infrastructure to support an increased user base.
6. Version 2 Authentication uses standard technologies and protocols.

## <a id="prerequisite"></a>Prerequisite
The V2 Authentication with Websocket API requires the following dependencies.
1. Authentication credential (*client_id* and *client_secret*).
2. Internet connection.
3. [WebSocket API - RTO examples](https://github.com/Refinitiv/websocket-api/tree/master/Applications/Examples/RDP)

*The purpose of these examples is to connect to Refinitiv Real-Time - Optimized (RTO) to retrieve JSON-formatted market content over a Websocket connection from a Refinitiv Real-Time Service after authenticating via Refinitiv Data Platform (RDP).*

Please contact your Refinitiv representative to help you to access the RTO account and services. 

## How to use with the WebSocket API with V2 Authentication

The [Websocket API for Pricing Streaming and Real-Time Services](https://developers.refinitiv.com/en/api-catalog/refinitiv-real-time-opnsrc/refinitiv-websocket-api) (WebSocket API) supports the V2 Authentication **manually**. The V2 Authentication examples are ```*MarketPrice_RdpGw_ClientCred_auth*``` example codes available on [Refinitiv/websocket-api GitHub](https://github.com/Refinitiv/websocket-api/tree/master/Applications/Examples/RDP) repository (in the ```/Applications/Examples/RDP/``` folder). 
The list of current examples for V2 Authentication is as follows:
- **Python**: market_price_rdpgw_client_cred_auth.py
- **C#**: MarketPriceRdpGwClientCredAuthExample/MarketPriceRdpGwClientCredAuthExample.cs
- **Java**: MarketPriceRdpGwClientCredAuth.java
- **Go**: market_price_rdpgw_client_cred_auth.go

Note: the examples use the Service Discovery feature by default.

The code explanation is based on the Python ```market_price_rdpgw_client_cred_auth.py``` file.

## Sample process of using WebSocket API with V2 Authentication

Here is a sample of how to use WebSocket API to retrieve an access token from the RDP V2 Authentication with Python programming language. Then, use it with the Service Discovery endpoint to discover an available RTO WebSocket Streaming Server endpoint to be connected to.

### 1) Requesting Access Token from the RDP V2 Authentication

First, set the RDP Authentication Service API endpoint to **v2**.

``` Python
auth_url = 'https://api.refinitiv.com/auth/oauth2/v2/token'
```

Then, send an HTTP POST request to the V2 Authentication service with the V2 - Client Credentials Grant.

``` Python
def get_auth_token(url=None):
    """
    Retrieves an authentication token.
    """

    if url is None:
        url = auth_url

    data = {'grant_type': 'client_credentials', 'scope': scope, 'client_id': clientid, 'client_secret': client_secret}

    print("\n" + str(datetime.now()) + \
            " Sending authentication request with client credentials to ", url, "...\n")
    try:
        # Request with auth for https protocol    
        r = requests.post(url,
                headers={'Accept' : 'application/json'},
                          data=data,
                          verify=True,
                          allow_redirects=False)

    except requests.exceptions.RequestException as e:
        print('Refinitiv Data Platform authentication exception failure:', e)
        return None, None
```

Next, get the Access Token and Expires_in information from the JSON response message.

``` Python
if r.status_code == 200:
  auth_json = r.json()
  print(str(datetime.now()) + " Refinitiv Data Platform Authentication succeeded. RECEIVED:")
  print(json.dumps(auth_json, sort_keys=True, indent=2, separators=(',', ':')))
  return auth_json['access_token'], auth_json['expires_in']
```
### 2) Requesting Service Discovery

The application can request the list of RTO Streaming servers from RDP Service Discovery (```/streaming/pricing/v1/``` endpoint) with the same logic as V1 Authentication.  This service accepts both Access Token V1 and V2. The source code does not need to change/update anything.

``` Python
try:
  r = requests.get(url, headers={"Authorization": "Bearer " + auth_token}, params={"transport": "websocket"}, allow_redirects=False)

except requests.exceptions.RequestException as e:
  print('Refinitiv Data Platform service discovery exception failure:', e)
  return False
```

### 3) Connecting to RTO WebSocket Streaming Server

This step is the same as the V1 Authentication. The consumer application initiates a WebSocket connection to the desire RTO WebSocket endpoints. Once the connection establishes successfully, the application needs to send the Access Token with the JSON Login message with a WebSocket connection.

The JSON Login message structure is as follows:
```JSON
{
    "ID": 1,
    "Domain": "Login",
    "Key": {
        "NameType": "AuthnToken",
        "Elements": {
            "ApplicationId": "<APP ID>",
            "Position": "<Position>",
            "AuthenticationToken": "<Access Token>"
        }
    }
}
```
The V2 Authentication Python code is shown below.

``` Python
def _send_login_request(self, authn_token):
  """
  Send login request with authentication token.
  Used both for the initial login and subsequent reissues to update the authentication token
  """
  login_json = {
    'ID': 1,
    'Domain': 'Login',
      'Key': {
        'NameType': 'AuthnToken',
        'Elements': {
          'ApplicationId': '',
          'Position': '',
          'AuthenticationToken': ''
          }
        }
      }

  login_json['Key']['Elements']['ApplicationId'] = app_id
  login_json['Key']['Elements']['Position'] = position
  login_json['Key']['Elements']['AuthenticationToken'] = authn_token

  self.web_socket_app.send(json.dumps(login_json))
```
### Requesting data from RTO WebSocket Streaming Server

This part has the same logic. Nothing changed.

### Access Token Renewal

As mentioned above about V2 Authentication, the application/API does not need to renew an Access Token (HTTP/RSSL-WebSocket) as long as the streaming channel (WebSocket/RSSL) is active.

However, if the application disconnects or is disconnected from the RTO Streaming server, the application needs to re-request an Access Token in the following scenarios:
  - When the consumer disconnects and goes into a reconnection state.
  - If the *streaming channel stays in reconnection* long enough to get close to the expiry time of the Access Token.

Example code:

``` Python
while True:
  # NOTE about connection recovery: When connecting or reconnecting 
  #   to the server, a valid token must be used. Upon being disconnecting, initial 
  #   reconnect attempt must be done with  a new token.
  #   If a successful reconnect takes longer than token expiration time, 
  #   a new token must be obtained proactively. 

  # Waiting a few seconds before checking for connection down and attempting reconnect
  time.sleep(4)
  if not session1.web_socket_open or ( session2 and not session2.web_socket_open ) :
    if session1.reconnecting or ( session2 and session2.reconnecting ) :
        curTS = time.time()
        if (int(expire_time) < 600):
          deltaTime = float(expire_time) * 0.90
        else:
          deltaTime = 300
        if (int(curTS) >= int(float(tokenTS) + float(deltaTime))):
          auth_token, expire_time = get_auth_token() 
          tokenTS = time.time()
    else:
      auth_token, expire_time = get_auth_token() 
      tokenTS = time.time()
    if auth_token is not None:
      if not session1.disconnected_by_user:
        session1.reconnecting = True
        session1.connect()
      if session2 and not session2.disconnected_by_user:
        session2.reconnecting = True
        session2.connect()
  else:
    print("Failed authentication with Refinitiv Data Platform. Exiting...")
    sys.exit(1) 
```
The code above checks if it is the initial disconnection, and re-request a new Access Token with the same Client Credential Grant request.

``` Python
if session1.reconnecting or ( session2 and session2.reconnecting ) :
 # Check Time Logic
else:
  auth_token, expire_time = get_auth_token() 
  tokenTS = time.time()
```

Then, it sets the application's state to the ```reconnection``` state and tries to re-connect to the RTO WebSocket streaming server using the new Access Token.

``` Python
else:
  auth_token, expire_time = get_auth_token() 
  tokenTS = time.time()
if auth_token is not None:
  if not session1.disconnected_by_user:
    session1.reconnecting = True
    session1.connect()
```

While the application's state is in the ```reconnection``` state, if it still cannot connect to the RTO WebSocket streaming server and the time gets close to the expiry time of the Access Token, the application must re-request the new Access Token for the next reconnection attempt.

``` Python
if session1.reconnecting or ( session2 and session2.reconnecting ) :
  curTS = time.time()
  if (int(expire_time) < 600):
    deltaTime = float(expire_time) * 0.90
  else:
    deltaTime = 300
  if (int(curTS) >= int(float(tokenTS) + float(deltaTime))):
    auth_token, expire_time = get_auth_token() 
    tokenTS = time.time()
else:
  # Re-request Access Token for an initial disconnect
```

The V1 Authentication workflow is different. It re-requests the Access Token every 8-9 minutes with the Refresh Grant, then re-sends the JSON Login message with that new Access token.

```Python
#  Continue using current token until 90% of initial time before it expires.
time.sleep(int(float(expire_time) * 0.90))

sts_token, refresh_token, expire_time = get_sts_token(refresh_token)
if not sts_token:
    sys.exit(1)
 
if int(expire_time) != int(original_expire_time):
  print('expire time changed from ' + str(original_expire_time) + ' sec to ' + str(expire_time) + ' sec; retry with password')
  sts_token, refresh_token, expire_time = get_sts_token(None)
  if not sts_token:
    sys.exit(1) 
  original_expire_time = expire_time 

# Update token.
if logged_in:
  send_login_request(sts_token, True)
```

### Running Examples
The V2 Authentication examples are ```*MarketPrice_RdpGw_ClientCred_auth*``` example codes available on [Refinitiv/websocket-api GitHub](https://github.com/Refinitiv/websocket-api/tree/master/Applications/Examples/RDP) repository (in the ```/Applications/Examples/RDP/``` folder). 

#### Python WebSocket - Running

``` Bash
$> python market_price_rdpgw_client_cred_auth.py --clientid <clientid> --clientsecret <clientsecret>
```

#### Example Run Result with Python

![figure-8](images/08_ws_python_run_result.gif "WebSocket API Python result")

#### Java WebSocket - Running

Firstly, build and compile the project with [Apache Ant](https://ant.apache.org/).
``` Bash
$>ant
```

Next, set the Java Classpath that Ant gave you

``` Bash
$>export CLASSPATH=lib/accessors-smart-1.1.jar:lib/asm-5.0.3.jar:lib/commons-cli-1.3.jar:lib/commons-codec-1.9.jar:lib/commons-logging-1.2.jar:lib/hamcrest-core-1.3.jar:lib/httpclient-4.5.3.jar:lib/httpcore-4.4.6.jar:lib/json-20160810.jar:lib/json-path-2.1.0.jar:lib/json-smart-2.2.jar:lib/junit-4.12.jar:lib/mockito-core-1.9.5.jar:lib/nv-websocket-client-1.30.jar:lib/objenesis-1.0.jar:lib/slf4j-api-1.7.13.jar:bin
```

![figure-9](images/09_ws_java_build_result_2.gif "WebSocket API Java Build result")

Then, run the example with the following command:

``` Bash
$>java MarketPriceRdpGwClientCredAuth --clientid <ClientID> --clientsecret <ClientSecret> --ric /EUR=
```
![figure-10](images/10_ws_java_run_result_1.gif "WebSocket API Java Run result")

## <a id="summary"></a>Summary
The RDP Version 2 Authentication simplifies the usage of access tokens when connecting to Refinitiv Real-Time Optimized. It uses the industry-standard [OAuth 2.0 - Client Credentials model] with a client ID, and client secret credentials instead of a username, password, and client ID (application key). The major advantage for real-time users is the applications don’t need to renew access tokens at every specific interval as the access token used by the application will remain valid until the application disconnects or is disconnected from Refinitiv Real-Time Optimized. 

To migrate applications to use the RDP Version 2 Authentication for WebSocket API developer, even though you need to manually update the application source code to use new HTTP and WebSocket connections, the V2 workflow is simple to operate when compared to the previous one.

However, in case you have any questions regarding WebSocket API usage and/or its migration to V2 Authentication, you are recommended to check our [Q&A Forum of WebSocket API](https://community.developers.refinitiv.com/spaces/152/index.html) or posting the question into this forum. We're willing to assist you regarding this.

If you are using the other APIs in the Refinitiv Real-Time SDK family, please see more detail about using it with the V2 Authentication from the following resources:
- [Getting Started with V2 Authentication for Refinitiv Real-Time using Enterprise Message C++ API](./EMA_Cpp_Migration_V2.md) article
- [Getting Started with Authentication V2 for Refinitiv Real-Time using Enterprise Message Java API](./EMA_Java_Migration_V2.md) article
- [ETA C: Refinitiv Real-Time Optimized Version 2 Authentication Migration Guide](./ETA_C_Migration_V2.md) article
- [ETA Java: Refinitiv Real-Time Optimized Version 2 Authentication Migration Guide](./ETA_Java_Migration_V2.md) article

## <a id="references"></a>References
For further details, please check out the following resources:
* [Refinitiv Real-Time SDK Family](https://developers.refinitiv.com/en/use-cases-catalog/refinitiv-real-time) page.
* [Refinitiv Real-Time SDK C/C++ page](https://developers.refinitiv.com/en/api-catalog/refinitiv-real-time-opnsrc/rt-sdk-cc) on the [Refinitiv Developer Community](https://developers.refinitiv.com/) website.
* [Refinitiv Real-Time SDK Java page](https://developers.refinitiv.com/en/api-catalog/refinitiv-real-time-opnsrc/rt-sdk-java).
* [Refinitiv WebSocket API page](https://developers.refinitiv.com/en/api-catalog/refinitiv-real-time-opnsrc/refinitiv-websocket-api).
* [ETA C: Refinitiv Real-Time Optimized Version 2 Authentication Migration Guide](./ETA_C_Migration_V2.md)
* [ETA Java: Refinitiv Real-Time Optimized Version 2 Authentication Migration Guide](./ETA_Java_Migration_V2.md)
* [Getting Started with Version 2 Authentication for Refinitiv Real-Time using Enterprise Message Java API](./EMA_Java_Migration_V2.md)
* [Getting Started with Version 2 Authentication for Refinitiv Real-Time using Enterprise Message C++ API](./EMA_Cpp_Migration_V2.md)
* [OAuth 2.0 - Client Credentials](https://www.oauth.com/oauth2-servers/access-tokens/client-credentials/) page.
* [OAuth 2.0 - Access Token Response](https://www.oauth.com/oauth2-servers/access-tokens/access-token-response/) page.
* [OAuth 2.0 - Password Grant](https://www.oauth.com/oauth2-servers/access-tokens/password-grant/) page.
* [OAuth 2.0 - Client Credentials Grant](https://oauth.net/2/grant-types/client-credentials/) page.
