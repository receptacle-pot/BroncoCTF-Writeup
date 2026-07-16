## Description
My friend tried to motivate me to review my derivatives by telling that me that I can unlock a top-secret image after I solve 500 challenges on this website. Unfortunately for her, I'm a firm believer in work smarter not harder, so I wonder if there's a way I can get the flag without actually doing any math?

[http://login.web.broncoctf.xyz](https://broncoctf-unblur-me.chals.io/)

## Flag :

```
bronco{1_WOULDNT_M@K3_YOU_DO_C@LCULUS}
```

### Solution :

firstly When I opened the challenge, I saw a webpage that looked like a calculus quiz.

It showed:

- A blurred image
- A progress counter (0 / 500)
- Random derivative questions
- An input box to submit answers

The page said I had to solve 500 derivative questions to remove the blur from the image.

At first, this looked like a math challenge, but since it was in the Web category, I suspected there might be a client-side vulnerability.

Inspecting the Source Code

I pressed F12 to open the browser's Developer Tools.

Then I looked at the page source and JavaScript.

I found this function:

```
function loadSecretImage() {
  fetch('/api/v1/internal/fetch-config-blob')
    .then(response => response.blob())
    .then(blob => {
      const blobUrl = URL.createObjectURL(blob);
      document.getElementById('flag-image').src = blobUrl;
    });
}
```

### I noticed that the image was downloaded as soon as the page loaded.

Understanding the Vulnerability

The code that checked whether we solved 500 questions looked like this:

if (correctCount >= 500) {
    flag.style.filter = "none";
}

This means:

The image is already downloaded.
The website only hides it using CSS blur.
Solving 500 questions only removes the blur effect.

The protection exists only in the browser.

Then, Downloading the Image

Since the browser already downloads the image, I opened the Network tab in Developer Tools.

After refreshing the page, I found the request:

/api/v1/internal/fetch-config-blob

Opening this request showed the original image.

No math was required.

So i did :
```
curl -o secret.bin https://broncoctf-unblur-me.chals.io/api/v1/internal/fetch-config-blob
```
As this link i have fpund in the network tab in the Request initiator chain.

And got the image which has flag in it:

<img width="824" height="464" alt="fetch-config-blob" src="https://github.com/user-attachments/assets/d119e13d-2d76-404b-bcd2-15a5f8e4d1db" />

