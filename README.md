# knotfun.duckdns.org static site

Hosts the universal-link landing page and the Apple app-site-association
file for Knot Fun shared knots (`https://knotfun.duckdns.org/k/<code>`).

## Files

- `k/index.html` — landing page for shared-knot links. Served for every
  `/k/<code>` path (see rewrite rule below). Shows the code, routes to the
  App Store, and teaches the install-then-retap flow.
- `.well-known/apple-app-site-association` — tells iOS which paths the app
  handles. **Replace `TEAM_ID`** with the Apple Developer team ID before
  deploying.

## Deploy requirements

1. **HTTPS with a valid cert** (Let's Encrypt works; duckdns.org supports it
   via certbot/dehydrated on your host). Universal links are ignored over
   plain HTTP.
2. Serve `/.well-known/apple-app-site-association` with:
   - Content-Type `application/json`
   - **No redirect** (including no http→https redirect on that exact file —
     Apple's CDN fetches it directly over HTTPS)
3. Rewrite `/k/*` (except real files) to `/k/index.html`. nginx example:

   ```nginx
   location /k/ {
       try_files $uri /k/index.html;
   }
   ```

4. Fill in the placeholders:
   - `TEAM_ID` in `.well-known/apple-app-site-association`
   - `APP_STORE_ID` (twice) in `k/index.html`

## Verify after deploy

- `curl -i https://knotfun.duckdns.org/.well-known/apple-app-site-association`
  → 200, `content-type: application/json`, no 3xx.
- Apple's CDN caches the AASA on first app install; after changing it,
  reinstall the app on the test device to refetch.
- Search "app-site-association" in the device console (Console.app) for
  `swcd` errors if links don't hand off.
