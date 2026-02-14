# Spring Boot OAuth2 Client Integration Summary

This document summarizes how the application consumes a protected API using OAuth2 Client Credentials.

**Automated Management:** Spring Security OAuth2 Client handles the token request, caching, and refresh logic automatically. You don't need to manually parse or "split" the JWT.

**Expiration Logic:** Spring uses the expires_in metadata from the token response to know when to fetch a new token before the current one dies.

**The Role of YAML:** Credentials (Client ID/Secret) are loaded into memory at startup.

**Hot Reload Impact:** If you change a secret to a "wrong" value while using a Hot Reload tool, the in-memory cache is wiped, forcing Spring to attempt a new (and failing) authentication with the incorrect credentials.

## 1. Authorization Architecture
The application acts as an **OAuth2 Client**. It uses the `client_id` and `client_secret` to obtain a JWT Access Token from the Provider's `token-uri`.

## 2. Token Lifecycle & Expiration
* **Token Validity:** 1 Hour (3600 seconds).
* **Storage:** Spring stores the token in the `OAuth2AuthorizedClientService`.
* **Automatic Refresh:** Before every API call, the `AuthorizedClientManager` checks the expiration timestamp. If the token is expired (or nearing expiration), it automatically performs a POST request to get a new token. **No manual JWT decoding is required.**

## 3. Configuration (application.yml)
The credentials are set in the configuration file:
```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          my-api:
            client-id: ${CLIENT_ID}
            client-secret: ${CLIENT_SECRET}
            authorization-grant-type: client_credentials
        provider:
          my-api:
            token-uri: [https://provider.com/oauth2/token](https://provider.com/oauth2/token)
