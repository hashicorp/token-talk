# Tokens

```text
  ,d               88
  88               88
MM88MMM ,adPPYba,  88   ,d8  ,adPPYba, 8b,dPPYba,  ,adPPYba,
  88   a8"     "8a 88 ,a8"  a8P_____88 88P'   `"8a I8[    ""
  88   8b       d8 8888[    8PP""""""" 88       88  `"Y8ba,
  88,  "8a,   ,a8" 88`"Yba, "8b,   ,aa 88       88 aa    ]8I
  "Y888 `"YbbdP"'  88   `Y8a `"Ybbd8"' 88       88 `"YbbdP"'
```


# Agenda

Why? Then How.

1. Authn vs Authz
2. Stateless vs. Stateful
3. Tokens/JWTs
4. Access vs. Refresh Tokens
5. Example Flow


# Authn vs Authz

* Authentication: to verify the identity of who you are.
* Authorization: to verify that you should be granted access to a resource or action.


# Authn vs Authz

Example 1: Flying on a plane

* passport: authentication
* plane ticket: authorization


# Authn vs Authz

Example 2: Riding the bus

A transit token authorizes you to ride the bus for 90 minutes.
Proof of identity is not required.

```plaintext
        +------------------------------+
        |        transit ticket        |
        |                              |
        |                              |
        |                              |
        | expires: 90 minutes          |
        +------------------------------+
```


# Authn vs Authz

Transferring the token to someone else, authorizes them to ride the bus for 90 minutes.
You're not supposed to transfer your transit token but detection is difficult.

```plaintext
        +------------------------------+
        |        transit ticket        |
        |                              |
        |                              |
        |                              |
        | expires: 90 minutes          |
        +------------------------------+
```


# Tokens - Stateful vs Stateless

Stateful: A stateful token is one where the token
needs to be looked up in a database to honour it.

Stateless: A stateless token has all the information
encoded in the token.


# Tokens - Impact on Microservices (Stateful style)

Let's say you have an application that is a cluster of microservices that is authenticated via Stateful tokens (not JWTs).

A user sends a request (along with a token) to the Bingo service, and the following events take place:
1. The Bingo service will verify the provided user's token is valid by sending it to the authentication service. If the token is valid, the Bingo service then forwards the token and request data to the Papaya Service.
2. The Papaya service only allows certain users to access it, so it sends the user's token to an Authorization service to try and fetch the user's permissions. If the user has the necessary permissions, it will do some work, then call the Magic Baby service (MBS for short) to fetch the final data needed to process the user's request.
3. MBS needs to validate that the provided token has permissions to use it, so it makes yet another request to the Authorization service to check if the user has the necessary permissions.

Whew... that's a lot of requests to auth services...

```plaintext
┌───────────┐                      ┌───────────┐
│           │                      │           │
│   Authn   │                ┌─────►   Authz   ◄──────────┐
│           │                │     │           │          │
└──────▲────┘                │     └───────────┘          │
       │                     │                            │
       │                     │                            │
       │                     │                            │
 ┌─────┴─────┐          ┌────┴──────┐             ┌───────┴─────────────┐
 │           │          │           │             │                     │
 │   Bingo   ├──────────►  Papaya   ├─────────────►   Magic Baby (MBS)  │
 │           │          │           │             │                     │
 └───────────┘          └───────────┘             └─────────────────────┘
```

P.S. If you're confused about the example Service names, see here: https://www.youtube.com/watch?v=y8OnoxKotPQ 

# Tokens - Impact on Microservices (Stateless style)

To get a sense of why stateless tokens are amazing, let's reimagine our cluster of microservices, but with them using Stateless tokens.

A user sends a request (along with a token) to the Bingo service, and the following events take place:
1. The Bingo service can verify the user's provided stateful token by simply checking if the signature is valid, so no request to an Auth service is needed. Just like in the previous example though, it still needs to forward the request and token to the Papaya service.
2. The Papaya service examines the stateful token it recieved from the Bing service, in the stateful token's payload it sees the user has permission to use it so no request to any Auth services are needed. It forwards the request along to the MBS.
3. MBS, just like the Papaya service, reads the stateful token's payload and sees that it has all the required permissions, so the request can be processed without any calls to any Auth services.

Less requests, means less complexity, means more time can be spent coming up with more abstract names for microservices! Woohoo!

```plaintext
┌───────────┐          ┌───────────┐             ┌─────────────────────┐
│           │          │           │             │                     │
│   Bingo   ├──────────►  Papaya   ├─────────────►   Magic Baby (MBS)  │
│           │          │           │             │                     │
└───────────┘          └───────────┘             └─────────────────────┘
```

# Tokens - JWT

JSON web tokens allow us to create stateless
tokens that encode the necessary information into the token.

```plaintext
{header}.{body}.{signature}
```


# Tokens - JWT

Example Token

```plaintext
{header}.{body}.{signature}
```

```plaintext
eyJhbGciOiJSUzI1NiJ9.eyJleHAiOjE1NTMyMDYx<snip>.BrWtDArYiut47Oo76UTD<snip>
```


# Tokens - JWT

JSON Web Signature

```json
{
  "alg": "RS256"
}
```

JWT Claims
```json
{
  "exp": 1553206143,
  "iat": 1553119743,
  "iss": "https://auth.acme.test/metadata",
  "nbf": 1553119743,
  "jti": "30ee4f06-3e2b-4ef4-961e-5a1dfd530ca5",
  "sub": "users/42",
  "name": "taylor.swift"
}
```

# Tokens - Types

* Access Token
  * Used to access a resource.
  * Shorter expiration time.


* Refresh Token
  * Exchanged for a new access/refresh token.
  * Less likely to be stolen because it is in-transit less often.
  * Longer expiration time.


# Example Flow

* Client: Your service, web app, SPA, mobile app.
* Resource Server: The API
* Authorization Server: The server that can issue JWTs.

```plaintext
    +--------+                                           +---------------+
    |        |--(A)------- Authorization Grant --------->|               |
    |        |                                           |               |
    |        |<-(B)----------- Access Token -------------|               |
    |        |               & Refresh Token             |               |
    |        |                                           |               |
    |        |                            +----------+   |               |
    |        |--(C)---- Access Token ---->|          |   |               |
    |        |                            |          |   |               |
    |        |<-(D)- Protected Resource --| Resource |   | Authorization |
    | Client |                            |  Server  |   |     Server    |
    |        |--(E)---- Access Token ---->|          |   |               |
    |        |                            |          |   |               |
    |        |<-(F)- Invalid Token Error -|          |   |               |
    |        |                            +----------+   |               |
    |        |                                           |               |
    |        |--(G)----------- Refresh Token ----------->|               |
    |        |                                           |               |
    |        |<-(H)----------- Access Token -------------|               |
    +--------+           & Optional Refresh Token        +---------------+
```

# Thanks

References:

* https://jwt.io/
* https://tools.ietf.org/html/rfc6749
* https://tools.ietf.org/html/rfc7515
* https://tools.ietf.org/html/rfc7519
* https://tools.ietf.org/html/rfc7522
