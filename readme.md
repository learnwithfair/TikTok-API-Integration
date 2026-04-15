# TikTok API Integration Documentation (OAuth 2.0 + PKCE)
[![Youtube][youtube-shield]][youtube-url]
[![Facebook][facebook-shield]][facebook-url]
[![Instagram][instagram-shield]][instagram-url]
[![LinkedIn][linkedin-shield]][linkedin-url]

Thanks for visiting my GitHub account!

## 1. Overview

This document explains how to integrate TikTok Login Kit API using OAuth 2.0 with PKCE, including:

* Developer account setup
* App creation (Sandbox & Production)
* OAuth login flow
* Redirect configuration
* Postman testing
* Access token & refresh token handling
* User profile API
* Video list API
* Video analytics approach

---

# 2. Prerequisites

* TikTok Developer Account
* Web application (Laravel / Node / etc.)
* Public HTTPS domain (or localhost for testing with tunneling tools)
* Postman for API testing

Official portal:
TikTok Developer Portal

---

# 3. Create TikTok Developer Account

## Step 1: Register

* Go to TikTok Developer Portal
* Sign in using TikTok account
* Complete developer profile

## Step 2: Verify Email / Phone

* Required for app creation access

---

# 4. Create Application (Sandbox Mode)

## Step 1: Create App

* Navigate to “My Apps”
* Click “Create App”
* Select **Web/Desktop App**

## Step 2: App Configuration

You will get:

* Client Key
* Client Secret

Store these securely:

```
CLIENT_KEY = xxxxx
CLIENT_SECRET = xxxxx
```

---

# 5. Sandbox Configuration

## Step 1: Enable Sandbox

* Activate Sandbox mode in app settings

## Step 2: Add Sandbox Users

* Add TikTok username(s)
* Only these users can test login

---

# 6. App Settings Configuration

## 6.1 Web/Desktop URL

This is your base application URL:

```
https://your-domain.com
```

NOT a callback URL.

---

## 6.2 Redirect URI (VERY IMPORTANT)

Example:

```
https://your-domain.com/tiktok/callback
```

Rules:

* Must match EXACTLY
* HTTPS recommended
* No trailing slash mismatch

---

# 7. OAuth Login Flow (PKCE)

## Step 1: Generate Code Verifier

Example:

```
random_string_123456789
```

---

## Step 2: Generate Code Challenge

SHA256 + Base64URL encoded version of verifier:

```
code_challenge = BASE64URL(SHA256(code_verifier))
```

```
Example: ZtNPunH49FDWg0z8F7hFhF38sZ4N2VN6iEOXcX4Jb9k
```

---

## Step 3: Redirect User to TikTok Login

```
https://www.tiktok.com/v2/auth/authorize/
?client_key=YOUR_CLIENT_KEY
&response_type=code
&scope=user.info.basic,video.list
&redirect_uri=https://your-domain.com/tiktok/callback
&state=xyz123
&code_challenge=YOUR_CODE_CHALLENGE
&code_challenge_method=S256
```

#### Example:
```
https://www.tiktok.com/v2/auth/authorize/
?client_key=YOUR_CLIENT_KEY
&response_type=code
&scope=user.info.basic,user.info.profile,user.info.stats,video.list
&redirect_uri=https://djakaridja.thewarriors.team/tiktok/callback
&state=test123
&code_challenge=ZtNPunH49FDWg0z8F7hFhF38sZ4N2VN6iEOXcX4Jb9k
&code_challenge_method=S256
```

---

## Step 4: User Login Result

After login, TikTok redirects:

```
https://your-domain.com/tiktok/callback?code=AUTH_CODE&state=xyz
```

Save:

* `code`

---

# 8. Exchange Code for Access Token (Postman)

## Endpoint

```
POST https://open.tiktokapis.com/v2/oauth/token/
```

## Headers

```
Content-Type: application/x-www-form-urlencoded
```

## Body

```
client_key=YOUR_CLIENT_KEY
client_secret=YOUR_CLIENT_SECRET
code=AUTH_CODE
grant_type=authorization_code
redirect_uri=https://your-domain.com/tiktok/callback
code_verifier=YOUR_CODE_VERIFIER // myrandomstring123456789
```

---

## Response

```
access_token
refresh_token
expires_in
open_id
scope
```

---

# 9. Refresh Token Flow

## Endpoint

```
POST https://open.tiktokapis.com/v2/oauth/token/
```

## Body

```
client_key=YOUR_CLIENT_KEY
client_secret=YOUR_CLIENT_SECRET
grant_type=refresh_token
refresh_token=YOUR_REFRESH_TOKEN
```

---

# 10. Get User Information

## Endpoint

```
GET https://open.tiktokapis.com/v2/user/info/
```

## Headers

```
Authorization: Bearer ACCESS_TOKEN
```

## Fields Example

```
?fields=open_id,display_name,avatar_url
```

---

## Example Request

```
GET https://open.tiktokapis.com/v2/user/info/?fields=open_id,display_name,avatar_url
```

---

# 11. Get Video List

## Endpoint

```
POST https://open.tiktokapis.com/v2/video/list/
```

## Headers

```
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

## Body

```
{
  "max_count": 20,
  "cursor": 0
}
```

## Fields (IMPORTANT)

```
?fields=id,title,view_count,like_count,comment_count,share_count
```

---

## Example Full Request

```
POST https://open.tiktokapis.com/v2/video/list/?fields=id,title,view_count,like_count,comment_count,share_count
```

---

# 12. Get Specific Video Analytics

## Important Limitation

TikTok does NOT provide direct single-video analytics endpoint.

## Correct Method

### Step 1: Fetch video list

### Step 2: Filter by video_id in backend

Example:

```
video_id = 7628864861979577608
```

---

## Backend Example

### Laravel / PHP

```php
$video = collect($videos)->firstWhere('id', $videoId);
```

### JavaScript

```js
const video = videos.find(v => v.id === videoId);
```

---

# 13. Video URL / Share Link

From `video/list` response:

You may get:

* share_url
* embed_html

Example:

```
https://www.tiktok.com/@user/video/123456
```

---

# 14. Analytics Limitations

## Available (basic scope)

* views
* likes
* comments
* shares

## Requires approval

* watch time
* engagement rate
* video insights
* audience data

---

# 15. Required Scopes

## Basic

```
user.info.basic
video.list
```

## Advanced

```
video.insights
user.info.stats
```

---

# 16. Common Errors

## invalid_grant

* code expired

## redirect_uri mismatch

* exact URL mismatch

## scope_not_authorized

* user did not approve permission

## 404 endpoint

* invalid API route

---

# 17. Production Architecture (Recommended)

## Flow

1. OAuth login
2. Store tokens in DB
3. Sync video list periodically
4. Store analytics in database
5. Build dashboard

---

## Database Tables

```
users
tiktok_tokens
tiktok_videos
tiktok_video_stats
```

---

# 18. Summary Flow

```
Login → Code → Token → User Info → Video List → Filter → Analytics Display
```


## Author

Developed and maintained by [MD. Rahatul Rabbi](https://github.com/learnwithfair).

---

## Follow

[<img src='https://cdn.jsdelivr.net/npm/simple-icons@3.0.1/icons/github.svg' alt='github' height='30'>](https://github.com/learnwithfair)
[<img src='https://cdn.jsdelivr.net/npm/simple-icons@3.0.1/icons/facebook.svg' alt='facebook' height='30'>](https://www.facebook.com/learnwithfair/)
[<img src='https://cdn.jsdelivr.net/npm/simple-icons@3.0.1/icons/instagram.svg' alt='instagram' height='30'>](https://www.instagram.com/learnwithfair/)
[<img src='https://cdn.jsdelivr.net/npm/simple-icons@3.0.1/icons/youtube.svg' alt='YouTube' height='30'>](https://www.youtube.com/@learnwithfair)

---

<!-- MARKDOWN LINKS -->
[youtube-shield]: https://img.shields.io/badge/-Youtube-black.svg?style=flat-square&logo=youtube&color=555&logoColor=white
[youtube-url]: https://youtube.com/@learnwithfair
[facebook-shield]: https://img.shields.io/badge/-Facebook-black.svg?style=flat-square&logo=facebook&color=555&logoColor=white
[facebook-url]: https://facebook.com/learnwithfair
[instagram-shield]: https://img.shields.io/badge/-Instagram-black.svg?style=flat-square&logo=instagram&color=555&logoColor=white
[instagram-url]: https://instagram.com/learnwithfair
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=flat-square&logo=linkedin&colorB=555
[linkedin-url]: https://linkedin.com/company/learnwithfair

#learnwithfair #rahtulrabbi #rahatul-rabbi #learn-with-fair
