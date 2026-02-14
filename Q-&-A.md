# OAuth2 Authorization & API Consumption Summary

## 1. The Difference: Access, Bearer, and Refresh Tokens
* **Access Token:** The actual string (often a JWT) that contains permissions and expiration (A Movie Ticket with a specific seat and time.).
* **Bearer Token:** A category of token. It means ""whoever bears (holds) this token gets access. (No extra ID or signature is needed from the sender.","A Physical Ticket. If you drop it and I pick it up, I can enter the movie.).
* **Refresh Token:** A long-lived credential used to get a new Access Token without asking the user for a password again. (A Voucher to get a new ticket if the first one expires.).

## 2. First Request vs. Second Request
First Request vs. Second Request
The interviewer wants to know if you understand Caching.

## The First Request (The "Cold" Start)
* **Check:** Spring looks for a token in the AuthorizedClientService. It finds null.
* **Exchange:** Spring calls the token-uri using your client_id and client_secret.
* **Store:** It receives the JWT and stores it in memory (along with its expiration time).
* **Execute:** It adds the header Authorization: Bearer <token> and hits the API.

## The Second Request (The "Warm" Start)
* **Check:** Spring looks in the cache. It finds the token from the first request.
* **Validate:** It compares the current time to the token’s expires_at timestamp.
* **Optimization:** If the token is still valid (e.g., only 5 minutes have passed), it skips the authentication step entirely.
* **Execute:** It uses the same token string from the first request to call the API.
* **The Difference:** The first request has the overhead of an extra network call to the Auth server. The second request is much faster because it only makes one network call (to the API).

| Step | First Request | Second Request (Within 1hr) |
| :--- | :--- | :--- |
| **Token Cache** | Empty | Found (Valid) |
| **Auth Server Call** | Yes (Fetch Token) | No (Skipped) |
| **API Call** | Yes | Yes |
| **Performance** | Slower (2 Network calls) | Faster (1 Network call) |

## 3. Configuration & Hot Reloading
* **Persistence:** Spring Boot caches tokens in-memory by default.
* **Hot Reload/Restart:** Using tools like DevTools or restarting the app **clears the cache**. 
* **Secret Updates:** If the `client_secret` is changed to an invalid value and the app reloads, the next API call will fail because the "First Request" logic will trigger and the Auth Server will reject the wrong credentials.

## 4. Implementation in Spring Boot
Use `spring-boot-starter-oauth2-client` with `WebClient`. This removes the need to manually:
1.  Split or decode JWTs to check expiration.
2.  Write `if/else` logic for 401 Unauthorized errors.
3.  Manually manage token headers.
