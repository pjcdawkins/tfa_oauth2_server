# TFA OAuth2 Server

A Drupal module to integrate [TFA](https://www.drupal.org/project/tfa) with [OAuth2 Server]
(https://www.drupal.org/project/oauth2_server).

This interrupts the Password Credentials ('password') grant type for users who have enabled TFA.

**Currently, this requires the latest dev version of OAuth2 Server**

Or apply this patch:
https://www.drupal.org/files/issues/oauth2_server-token_hook-2571265-2.patch

## Flow
The client requests a token using password credentials, as normal:
```
POST /oauth2/token
Accept: application/json
Authorization: Basic <base64-encoded client credentials>
Content-Type: application/x-www-form-urlencoded

grant_type=password&username=alice&password=passw0rd
```

If the password is incorrect, then the normal OAuth 2.0 error is sent.

If the password is correct, and the user has TFA enabled, then a 403 response is sent:
```
401 Unauthorized
Content-Type: application/json
X-Drupal-TFA: required, schemes=totp

{"error":"invalid_grant", "error_description": "Two-factor authentication is required"}
```

The initial request should be repeated, with the current TOTP code included:
```
POST /oauth2/token
Accept: application/json
Authorization: Basic <base64-encoded client credentials>
Content-Type: application/x-www-form-urlencoded
X-Drupal-TFA: 123456

grant_type=password&username=alice&password=passw0rd
```

The TOTP code is validated, and an OAuth 2.0 access token is sent as normal if it succeeds.
