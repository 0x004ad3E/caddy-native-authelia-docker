# caddy-native-authelia-docker
this is for configuring caddy-native with authelia-docker

- Domains configured in orange cloudflare
- Trusted cloudflare proxies. Access only via cloudflare proxy
- Cloudflare dns challenge for ssl certificates
- Use authelia for authentication
- Sites with Foward auth
- Sites without Forward auth
- Sites with oidc for immich(docker), nextcloud(docker) [see other repos for details]
- Forward auth for Jellyfin with custom headers

Note: Generate API key from Cloudflare for dns challenge [see other repo for details]

* See authelia/docker-compose.yml for authelia docker
* See authelia/users.yml for adding users for authentication
* See authelia/configuration.yml for authelia configuration to support forward auth and oidc

Start authelia
--------------
* Copy authelia/docker-compose.yml to /opt/authelia/docker-compose.yml
* Copy users_database.yml and configuration.yml to the bind mount location (see docker-compose.yml)
* Start authelia docker (see docker-compose.yml)

Start Caddy
-----------
* Install caddy
* Copy the Caddyfile to  /etc/caddy/Caddyfile
* Modify the Caddyfile as required
* Start / Restart caddy - sudo systemctl restart/start caddy


Caddy configuration
-------------------
The Caddyfile is an easy way to configure your Caddy web server.

 To use your own domain name (with automatic HTTPS), first make
 sure your domain's A/AAAA DNS records are properly pointed to
 this machine's public IP

 Caddy Reverse Proxy Configuration Notes
----------------------------------------

<img width="495" height="381" alt="grafik" src="https://github.com/user-attachments/assets/845658d4-9828-4b45-9481-ccfb501c8529" />


 Authentication model:

   Authelia:
       - Controls whether a user is allowed to access a service.
       - Provides SSO/session-based authentication.
       - Does NOT replace application-specific logins.

   Applications:
       - Continue to handle their own users, permissions,
         and API authentication.



 Authelia forward_auth notes
------------------------------

 The "forward_auth" directive works by:

   1. Client requests an application.
   2. Caddy sends a sub-request to Authelia:

          /api/authz/forward-auth

   3. Authelia checks the user's session cookie.
   4. If authorized, Caddy forwards the original request
      to the backend application.



 Jellyfin special handling
------------------------------

 Jellyfin cannot use the normal Authelia authentication
 snippet because Jellyfin uses its own HTTP Authorization
 scheme:

       Authorization: MediaBrowser ...

 This is NOT a standard Basic/Bearer authentication scheme.

 Without special handling:

<img width="598" height="301" alt="grafik" src="https://github.com/user-attachments/assets/84a7dffc-3d71-4edd-b53d-ab59b9afd00f" />


 Authelia attempts to parse the Authorization header and
 rejects the unknown "MediaBrowser" scheme before Jellyfin
 ever receives the request.


 Solution:

 A separate "(auth-jellyfin)" snippet is used.

 It removes the Authorization header ONLY from the request
 sent to Authelia for the access check:

       header_up -Authorization

 The original Authorization header is still passed to
 Jellyfin, allowing normal Jellyfin authentication to work.


 Result:

<img width="651" height="286" alt="grafik" src="https://github.com/user-attachments/assets/d6b981e7-665a-4afb-a2fa-7beb7d2c69b4" />




 Why there are two authentication snippets
 -----------------------------------------

 (auth)

   Used by normal applications.
   Keeps the Authorization header unchanged.


 (auth-jellyfin)

   Used only by Jellyfin.
   Removes Authorization header during Authelia check because
   Jellyfin uses the custom MediaBrowser authentication scheme.



 Maintenance notes
--------------------------

 - If Authelia, Caddy, or Jellyfin are upgraded, retest whether
   the Jellyfin-specific workaround is still required.

 - The header_up warnings from Caddy about X-Forwarded-For and
   X-Forwarded-Proto are harmless. Caddy already sets these
   headers automatically for reverse_proxy.

 - Keep application-specific reverse proxy settings close to
   each site block rather than modifying global snippets.

 Last reviewed:
 July 2026

 ============================================================ 
