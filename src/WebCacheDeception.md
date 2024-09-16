# Web Cache Deception
- Vulnerability that enables an attacker to store in the cache a dynamic response by confusing it for a static resource.
- It is caused by discrepancies between how the cache server and the origin server handles requests

## Impact
- An attacker tricks a user into visiting a malicious link, that makes a call for dynamic resources confused as static content. The attacker can then visit the same URL to retrieve the results of the dynamic resource intended only for the victim user (ex: their profile page + including username + password)

### Cache rules
- determines what to cache and for how long
- usually stores static content (ex: .js, .css, .ico, robots.txt, /assets or /static folder contents)

## How to construct an attack?
- Find a url that ignores the contents after "/" (with wildcard mapping)
    - ex: /profile = /profile/abcd
- Add the file extension of a static resource => that will cause the request to be stored in the cache (ex: /profile/a.js)
- Send the link to the victim and wait for it to be clicked
- Visit the same page (/profile/a.js) to see the profile details of the victim user


### Types of web cache deception
- `Exploiting path mapping discrepancies`
    - ex: `/api/orders/123` - logical mapping
    - `/api/orders/123/abcd` - ignores abcd, anything after /
- `Delimiter discrepancies`
    - ex: `/profile;foo.css`
    - **Java Spring** uses `;` as delimiter to add matrix variables -> such a server running Java Spring would drop everything after *;*
    - ex2: Ruby on Rails uses `.(dot)` for delimiter: `/profile.css or /profile.ico`
    - ex3: `profile%00foo.js`
    - DELIMITER LIST WORDLIST
        - https://portswigger.net/web-security/web-cache-deception/wcd-lab-delimiter-list
- `Delimiter decoding discrepancies`
    - some parsers will url decode and then separe, others won't url decode. The cache won't decode them, if the backend decodes them and drops the a.js => returns dynamic data
    - try encoding separator characters(ex: #, ? + non-printable characters %00, %0A, %09)
- `Exploiting static directory cache rules!`
    - **/static, /assets, /scripts, /images**
- `Exploiting Normalization Discrepancies`
    - Use `encoded path traversal for this`:
        - ex: `/static/..%2fprofile`
        - *not all servers url decode chars and not all servers resolve dot segments*
        