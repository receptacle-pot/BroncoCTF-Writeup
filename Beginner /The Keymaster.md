## The Keymaster has split a flag into 8 keys and hid them in plain sight.

## Quite literally, as they're on our advertisement page!

## Ready your cursor-pointers, pull out your trusty inspection panel, and find them quickly, detective!

## https://broncosec.com/BroncoCTF

In the view page source i got a comment at very top of HTML file : <!--g9AnrGROc7LGTm2csS2Oo-->

Also in the source the title contains a part of flag as :
<a title="3 - 0und_th3" href="https://broncoctf.ctfd.io/">

- Part 3 : 3 - 0und_th3

The challenge says: 
- "split a flag into 8 keys and hid them in plain sight. Ready your cursor-pointers..."

So i checked more in inpect and view file source in HTml, CSS and JavaScript file too.

I also noticed this interesting link: <a href="/7.txt" download="7.txt">2026</a> downloadable files.

In the source page view searched for : bronco{ 

And got he 1st part od the flag :

- Part 1 : 1 - bronco{h

nothing more got in the view page source so, i fetched : curl https://broncosec.com/7.txt

┌──(kali㉿kali)-[~/Downloads] 
└─$ curl https://broncosec.com/7.txt
7 - _w0rr135

-Part 7 : 7 - _w0rr135

Downloaded the the JavaScript bundles:

mkdir js
cd js

┌──(kali㉿kali)-[~/Downloads/js]
└─$ curl -s https://broncosec.com/BroncoCTF | grep -o '/_next/static/chunks/[^"]*\.js' | sort -u

output : 
/_next/static/chunks/0ff423a9fcc0186e.js
/_next/static/chunks/247eb132b7f7b574.js
/_next/static/chunks/8296bf97416a5ebf.js
/_next/static/chunks/8c4bb65ca9f95eb5.js
/_next/static/chunks/a6dad97d9634a72d.js
/_next/static/chunks/cbd55ab9639e1e66.js
/_next/static/chunks/e785679bf8074938.js
/_next/static/chunks/f31cf569852813cb.js
/_next/static/chunks/ff1a16fafef87110.js
/_next/static/chunks/turbopack-1a1407eeed6b9166.js

Downloaded all the .js files by putting this all files url in a single chunk.txt file using :
Also grep and search for flag part from 1-8 and flag 
┌──(kali㉿kali)-[~/Downloads/js]
└─$ wget -i chunks.txt | grep -RniE '[1-8] *-|cookie|flag|0und|w0rr' .

This revels other parts of the flag inside the javascript file.

<img width="1918" height="968" alt="Screenshot_2026-07-13_12_23_39" src="https://github.com/user-attachments/assets/4ffc73da-6d01-48b4-9863-749140ae6a58" />

- Part 2 : 2 - 3y_y0u_f
- Part 4 : 4 - m_4ll_w1
- Part 5 : 5 - th_4b501
- Part 6 : 6 - ut31y_n0
- Part 8 : 8 - _4t_411}

After assembling each part of the flag 

Flag : bronco{h3y_y0u_f0und_th3m_4ll_w1th_4b501ut31y_n0_w0rr135_4t_411}


