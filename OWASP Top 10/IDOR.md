### Insecure Direct Object Reference (IDOR)

- an access control vulnerability where you can access resources you wouldn't ordinarily be able to see

-  This occurs when the programmer exposes a Direct Object Reference, which is just an identifier that refers to specific objects within the server

- By object, we could mean a file, a user, a bank account in a banking application, or anything really

- For example: `https://bank.thm/account?id=111111`

- There is, however, a potentially huge problem here, anyone may be able to change the `id` parameter to something else like `222222`, and if the site is incorrectly configured, then he would have access to someone else's bank information.

- The application exposes a direct object reference through the `id` parameter in the URL, which points to specific accounts. Since the application isn't checking if the logged-in user owns the referenced account, an attacker can get sensitive information from other users because of the IDOR vulnerability. Notice that direct object references aren't the problem, but rather that the application doesn't validate if the logged-in user should have access to the requested account.