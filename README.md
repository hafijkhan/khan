# :octocat: find available github username

## 📜 Table of Contents

- [Usage](#usage)
  - [v1 script](#v1-script)
  - [v2 script](#v2-script)
  - [Combining both versions](#combining-both-versions)
- [Project Background](#project-background)
- [FAQs](#faqs)
  - [Dictionary file name meaning?](#dictionary-file-name-meaning)
  - [What is the difference between full dictionary and `easy-to-remember`?](#what-is-the-difference-between-full-dictionary-and-easy-to-remember)
  - [How do I generate a dictionary?](#how-do-i-generate-a-dictionary)
  - [How to generate `easy-to-remember` dictionaries from the full dictionary?](#how-to-generate-easy-to-remember-dictionaries-from-the-full-dictionary)
  - [How can I exclude those `easy-to-remember` usernames from the full dictionary?](#how-can-i-exclude-those-easy-to-remember-usernames-from-the-full-dictionary)
  - [Why are some usernames unavailable checked via `v1.sh`](#why-are-some-usernames-unavailable-checked-via-v1sh)
  - [How to check GitHub API remaining available requests?](#how-to-check-github-api-remaining-available-requests)
  - [Is it possible to use it under Windows?](#is-it-possible-to-use-it-under-windows)
  - [How do I find my current username?](#how-do-i-find-my-current-username)
  - [Is further support available?](#is-further-support-available)
- [License](#license)

## Usage

Clone the repository

```sh
git clone https://github.com/oood/find-available-github-usernames.git && cd find-available-github-usernames
```

### v1 script

---

|Conditions|Requests per Hour|
|-|-|
|Username only|60|
|Username and token|5000|

---

1) **Generate a Personal Access Token**:
   - Register on GitHub and go to `Settings > Developer Settings > Personal Access Token` to create your token.
  
2) Edit the script and fill in your current GitHub username and token

```sh
USER="" # your_current_username
TOKEN="" # your_api_token
```

3) Run script

````sh
./v1.sh <path to dictionary.txt> <threads>
````

**To stop script, just press ```^C (Ctrl+C)``` or wait until it stops**

**To get information about api limits:**

````sh
./v1.sh --api
````

Response example:

```sh
> ./v1.sh --api
TOKEN is empty
Limit: 60
Used: 46
Remaining: 14
Reset time: 2024-10-21 20:11:20
```

**Notes:**


The working principle of the script is very simple, import the first line of the dictionary, check whether the username is available, if got `HTTP: 404`, add it to the `v1_found.txt`, if encounter an error, print and output to the `v1.log`, and finally delete all checked lines in the dictionary.


The reason for deleting the lines in the dictionary is to ensure that the task can be terminated at any time by pressing `^c` (Ctrl+C) in the terminal and don't have to start from the beginning on the next run.


The first time the script is run, a backup file with a `.bak` suffix is generated for the dictionary.

### v2 script

---

~4200-6000 Requests Per Hour

---

1) Find tokens from GitHub web version.
    - Go to `Settings` > `Account` > `Change username`
    - Open `Web DevTools` > `Network`
    - Type something in `Choose a new username form`
    - Click on `rename_check?suggest_usernames=true` in `Web DevTools`
    - Look at Headers and Payload
  
2) Edit the script and fill in your current tokens

```sh
# copy only <token>

# in headers starts with 
# _octo=<token>
TOKEN_1=""

# in headers starts with 
# boundary=----<token>
TOKEN_2="" # updates if you are making a lot of requests

# in headers starts with 
# user_session=<token> 
# or 
# __Host-user_session_same_site=<token>
TOKEN_3=""

# in payload starts with 
# authenticity_token: <token>
TOKEN_4="" # updates if you are making a lot of requests
```

3) Run script

````sh
./v2.sh <path to dictionary.txt> <threads>
````

GitHub allows you to make 70-100 requests per minute so you can run:

```sh
./auto-v2.sh <path to dictionary.txt>
```

it just runs `v2.sh` every minute with 100 threads

**To stop script, just press ```^C (Ctrl+C)``` or wait until it stops**

### Combining both versions

Use `v1.sh` and after what `auto-v2.sh` with result of `v1.sh`

```sh
./v1 dictionaries/4-characters_easy-to-remember_AAAA-ZZZZ.txt 100
```

And next

```sh
./auto-v2.sh v1_found.txt
```

result will be in `v2_found.txt`

---

## Project Background

GitHub allows everyone to check user pages, I initially wrote a script and created some dictionaries with character combinations to check all URLs `https://github.com/$username`. this worked fine at first, but unfortunately after a few minutes I triggered an `Error: 429 Too many requests`, so I started searching to see if there was a better way, and I found that GitHub provides an API that can be used to retrieve registered username and available username, but I don't have an account how can I get an API token?


There are two ways: First, register an account with any username, then get the API token and then find the appropriate username and modify it. Or bypass the restriction with proxies :shipit:, which I know is rude, so I chose the first way.


GitHub serves 5,000 requests per hour for users using the API, which seems a bit low considering the short usernames that are still available from millions of usernames, but compared to only 60 requests per IP per hour without the API, that's a huge difference.


I generated the 2-character dictionary and found some "available" usernames, but I found out that they were just reserved usernames, then I tried the 3-character dictionary again, this time I didn't intend to check all 3-characters, because I didn't want to wait too long, and all combinations were already well over the limit of 5,000 per hour, I generated `easy-to-remember` dictionaries from all combinations, then I didn't find a username that worked for me, and finally I found my current username in a 4-character `easy-to-remember` dictionary.

## FAQs

### Dictionary file name meaning?

---

`2-characters_00-99_AA-ZZ.txt` Consists of 2 characters, including all combinations of letters and digits.

`3-characters_AAA-ZZZ.txt` and `4-characters_AAAA-ZZZZ.txt` Contains all combinations of all 3 and 4 letters, no digits.

`XX-characters_easy-to-remember_XX.txt` Usernames with repeated characters.

### What is the difference between full dictionary and `easy-to-remember`?

---

I recommend trying the `easy-to-remember` ones in my dictionaries first, as those that have some kind of regularity, like multiple occurrences of the same character, and easier to type. other dictionaries are full dictionaries, which means they may contain thousands of usernames.

### How do I generate a dictionary?

---

For 2 characters:

```sh
printf "%s\n" {0,1,2,3,4,5,6,7,8,9,a,b,c,d,e,f,g,h,i,j,k,l,m,n,o,p,q,r,s,t,u,v,w,x,y,z}{0,1,2,3,4,5,6,7,8,9,a,b,c,d,e,f,g,h,i,j,k,l,m,n,o,p,q,r,s,t,u,v,w,x,y,z} > ./2-characters.txt
```

For 4 characters:

```sh
echo {a..z}{a..z}{a..z}{a..z} | tr ' ' '\n' > ./4-characters.txt
```

### How to generate `easy-to-remember` dictionaries from the full dictionary?

---

```sh
grep '\(.\).*\1' 3-characters_AAA-ZZZ.txt > ./3-characters_easy-to-remember_AAA-ZZZ.txt
```

```sh
grep '\(.\).*\1.*\1' 4-characters_AAAA-ZZZZ.txt > ./4-characters_easy-to-remember_AAAA-ZZZZ.txt
```

This will extract usernames that have the same letter repeated multiple times, like `aaa`, `aab`, `aba`...

### How can I exclude those `easy-to-remember` usernames from the full dictionary?

---

```sh
comm -23 ./dictionary.txt ./easy-to-remember.txt > ./easy-to-remember-excluded.txt
```

### Why are some usernames unavailable checked via `v1.sh`?

---

GitHub may for some reason keep some usernames that are not open for registration, such as `47`, `fr`, `ccc`, although I found a lot of 2-character and 3-character usernames when I tried to register, but they were not available, but that doesn't mean you shouldn't try those 2 and 3-character usernames, because at any time a user could delete their account or change their username, so you might have better luck than me.

The `v2.sh` script utilizes the  `https://github.com/account/rename_check` endpoint to verify the availability of these usernames.

### How to check GitHub API remaining available requests?

---

```sh
./v1.sh --api
```

or

```sh
curl -i -u "$USER:$TOKEN" -H "Accept: application/vnd.github.v3+json" https://api.github.com/rate_limit
```

Is it possible to use it under Windows?
---------------------------------------
It should work fine with [WSL](https://docs.microsoft.com/en-us/windows/wsl/install) and some 3rd party Linux tools ([Cygwin](https://github.com/cygwin/cygwin) or [Git bash](https://github.com/git-for-windows/git)).

How do I find my current username?
----------------------------------
1. In the GitHub Desktop menu, click Preferences.

2. In the Preferences window, verify the following:
    - To view your GitHub username, click Accounts.

Is further support available?
-----------------------------
No, I'm not going to use my time to continue developing and contributing to this repository, it's just a sharing of some experiences, if you're interested you can create something better!

License
=======
This project is available under the [MIT License](LICENSE), allowing anyone to use, modify, and distribute the work without restrictions.

[![CC0](https://licensebuttons.net/p/zero/1.0/88x31.png)](https://creativecommons.org/publicdomain/zero/1.0/)

<div class="BlobLicenseBanner-module__Box--ZTbkr"><div class="blob-license-banner-outer BlobLicenseBanner-module__Box_1--fLkDE"><div class="BlobLicenseBanner-module__Box_2--tcP0E"><div class="BlobLicenseBanner-module__Box_3--hdf26"><svg aria-hidden="true" focusable="false" class="octicon octicon-law Octicon__StyledOcticon-sc-jtj3m8-0" viewBox="0 0 24 24" width="32" height="32" fill="currentColor" display="inline-block" overflow="visible" style="vertical-align: text-bottom;"><path d="M12.75 2.75V4.5h1.975c.351 0 .694.106.984.303l1.697 1.154c.041.028.09.043.14.043h4.102a.75.75 0 0 1 0 1.5H20.07l3.366 7.68a.749.749 0 0 1-.23.896c-.1.074-.203.143-.31.206a6.296 6.296 0 0 1-.79.399 7.349 7.349 0 0 1-2.856.569 7.343 7.343 0 0 1-2.855-.568 6.205 6.205 0 0 1-.79-.4 3.205 3.205 0 0 1-.307-.202l-.005-.004a.749.749 0 0 1-.23-.896l3.368-7.68h-.886c-.351 0-.694-.106-.984-.303l-1.697-1.154a.246.246 0 0 0-.14-.043H12.75v14.5h4.487a.75.75 0 0 1 0 1.5H6.763a.75.75 0 0 1 0-1.5h4.487V6H9.275a.249.249 0 0 0-.14.043L7.439 7.197c-.29.197-.633.303-.984.303h-.886l3.368 7.68a.75.75 0 0 1-.209.878c-.08.065-.16.126-.31.223a6.077 6.077 0 0 1-.792.433 6.924 6.924 0 0 1-2.876.62 6.913 6.913 0 0 1-2.876-.62 6.077 6.077 0 0 1-.792-.433 3.483 3.483 0 0 1-.309-.221.762.762 0 0 1-.21-.88L3.93 7.5H2.353a.75.75 0 0 1 0-1.5h4.102c.05 0 .099-.015.141-.043l1.695-1.154c.29-.198.634-.303.985-.303h1.974V2.75a.75.75 0 0 1 1.5 0ZM2.193 15.198a5.414 5.414 0 0 0 2.557.635 5.414 5.414 0 0 0 2.557-.635L4.75 9.368Zm14.51-.024c.082.04.174.083.275.126.53.223 1.305.45 2.272.45a5.847 5.847 0 0 0 2.547-.576L19.25 9.367Z"></path></svg><div class="BlobLicenseBanner-module__Box_4--tUGlM"><div class="BlobLicenseBanner-module__Box_5--pNkB6">hafijkhan/khan is licensed under  the</div><h3>MIT License</h3></div></div><div class="Box-sc-62in7e-0 BlobLicenseBanner-module__VerifiedHTMLBox--TIzSo">A short and simple permissive license with conditions only requiring preservation of copyright and license notices. Licensed works, modifications, and larger works may be distributed under different terms and without source code.</div></div><div class="BlobLicenseBanner-module__Box_6--RO_Q9"><div class="BlobLicenseBanner-module__Box_7--KUIEo"><h5 class="BlobLicenseBanner-module__Box_8--dxXDr">Permissions</h5><div class="BlobLicenseBanner-module__Box_9--ZuLtf"><svg aria-hidden="true" focusable="false" class="octicon octicon-check Octicon__StyledOcticon-sc-jtj3m8-0 jLpWtk BlobLicenseBanner-module__Octicon--ew2Qo" viewBox="0 0 16 16" width="13" height="13" fill="currentColor" display="inline-block" overflow="visible" style="vertical-align: text-bottom;"><path d="M13.78 4.22a.75.75 0 0 1 0 1.06l-7.25 7.25a.75.75 0 0 1-1.06 0L2.22 9.28a.751.751 0 0 1 .018-1.042.751.751 0 0 1 1.042-.018L6 10.94l6.72-6.72a.75.75 0 0 1 1.06 0Z"></path></svg>Commercial use</div><div class="BlobLicenseBanner-module__Box_9--ZuLtf"><svg aria-hidden="true" focusable="false" class="octicon octicon-check Octicon__StyledOcticon-sc-jtj3m8-0 jLpWtk BlobLicenseBanner-module__Octicon--ew2Qo" viewBox="0 0 16 16" width="13" height="13" fill="currentColor" display="inline-block" overflow="visible" style="vertical-align: text-bottom;"><path d="M13.78 4.22a.75.75 0 0 1 0 1.06l-7.25 7.25a.75.75 0 0 1-1.06 0L2.22 9.28a.751.751 0 0 1 .018-1.042.751.751 0 0 1 1.042-.018L6 10.94l6.72-6.72a.75.75 0 0 1 1.06 0Z"></path></svg>Modification</div><div class="BlobLicenseBanner-module__Box_9--ZuLtf"><svg aria-hidden="true" focusable="false" class="octicon octicon-check Octicon__StyledOcticon-sc-jtj3m8-0 jLpWtk BlobLicenseBanner-module__Octicon--ew2Qo" viewBox="0 0 16 16" width="13" height="13" fill="currentColor" display="inline-block" overflow="visible" style="vertical-align: text-bottom;"><path d="M13.78 4.22a.75.75 0 0 1 0 1.06l-7.25 7.25a.75.75 0 0 1-1.06 0L2.22 9.28a.751.751 0 0 1 .018-1.042.751.751 0 0 1 1.042-.018L6 10.94l6.72-6.72a.75.75 0 0 1 1.06 0Z"></path></svg>Distribution</div><div class="BlobLicenseBanner-module__Box_9--ZuLtf"><svg aria-hidden="true" focusable="false" class="octicon octicon-check Octicon__StyledOcticon-sc-jtj3m8-0 jLpWtk BlobLicenseBanner-module__Octicon--ew2Qo" viewBox="0 0 16 16" width="13" height="13" fill="currentColor" display="inline-block" overflow="visible" style="vertical-align: text-bottom;"><path d="M13.78 4.22a.75.75 0 0 1 0 1.06l-7.25 7.25a.75.75 0 0 1-1.06 0L2.22 9.28a.751.751 0 0 1 .018-1.042.751.751 0 0 1 1.042-.018L6 10.94l6.72-6.72a.75.75 0 0 1 1.06 0Z"></path></svg>Private use</div></div><div class="BlobLicenseBanner-module__Box_7--KUIEo"><h5 class="BlobLicenseBanner-module__Box_8--dxXDr">Limitations</h5><div class="BlobLicenseBanner-module__Box_9--ZuLtf"><svg aria-hidden="true" focusable="false" class="octicon octicon-x Octicon__StyledOcticon-sc-jtj3m8-0 fTVHHw BlobLicenseBanner-module__Octicon--ew2Qo" viewBox="0 0 12 12" width="13" height="13" fill="currentColor" display="inline-block" overflow="visible" style="vertical-align: text-bottom;"><path d="M2.22 2.22a.749.749 0 0 1 1.06 0L6 4.939 8.72 2.22a.749.749 0 1 1 1.06 1.06L7.061 6 9.78 8.72a.749.749 0 1 1-1.06 1.06L6 7.061 3.28 9.78a.749.749 0 1 1-1.06-1.06L4.939 6 2.22 3.28a.749.749 0 0 1 0-1.06Z"></path></svg>Liability</div><div class="BlobLicenseBanner-module__Box_9--ZuLtf"><svg aria-hidden="true" focusable="false" class="octicon octicon-x Octicon__StyledOcticon-sc-jtj3m8-0 fTVHHw BlobLicenseBanner-module__Octicon--ew2Qo" viewBox="0 0 12 12" width="13" height="13" fill="currentColor" display="inline-block" overflow="visible" style="vertical-align: text-bottom;"><path d="M2.22 2.22a.749.749 0 0 1 1.06 0L6 4.939 8.72 2.22a.749.749 0 1 1 1.06 1.06L7.061 6 9.78 8.72a.749.749 0 1 1-1.06 1.06L6 7.061 3.28 9.78a.749.749 0 1 1-1.06-1.06L4.939 6 2.22 3.28a.749.749 0 0 1 0-1.06Z"></path></svg>Warranty</div></div><div class="BlobLicenseBanner-module__Box_7--KUIEo"><h5 class="BlobLicenseBanner-module__Box_8--dxXDr">Conditions</h5><div class="BlobLicenseBanner-module__Box_9--ZuLtf"><svg aria-hidden="true" focusable="false" class="octicon octicon-info Octicon__StyledOcticon-sc-jtj3m8-0 gHqjiX BlobLicenseBanner-module__Octicon--ew2Qo" viewBox="0 0 16 16" width="13" height="13" fill="currentColor" display="inline-block" overflow="visible" style="vertical-align: text-bottom;"><path d="M0 8a8 8 0 1 1 16 0A8 8 0 0 1 0 8Zm8-6.5a6.5 6.5 0 1 0 0 13 6.5 6.5 0 0 0 0-13ZM6.5 7.75A.75.75 0 0 1 7.25 7h1a.75.75 0 0 1 .75.75v2.75h.25a.75.75 0 0 1 0 1.5h-2a.75.75 0 0 1 0-1.5h.25v-2h-.25a.75.75 0 0 1-.75-.75ZM8 6a1 1 0 1 1 0-2 1 1 0 0 1 0 2Z"></path></svg>License and copyright notice</div></div></div></div><div class="BlobLicenseBanner-module__Box_10--b6JCS">This is not legal advice.&nbsp;<a class="prc-Link-Link-9ZwDx" data-inline="true" href="https://docs.github.com/articles/licensing-a-repository/#disclaimer">Learn more about repository licenses</a></div></div>

<div class="footer">
  &copy; 2026 Hafij Khan. 
</div>
