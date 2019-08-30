
This module can be useful if you need to test the security of your server and your
website behind a solution Cloud based. By discovering the origin IP address of the
targeted host.

More precisely, I use multiple data sources (in order ViewDNS.info, DNS enumeration and Censys)
to collect assigned (or have been assigned) IP addresses from the targeted site or domain
that uses the following:
  Amazon Cloudflare, Amazon CloudFront, Imperva Incapsula, InGen Security (BinarySec EasyWAF)
  and Sucuri.

## Verification Steps

  1. Install the module as usual
  2. Start msfconsole
  3. Do: `use auxiliary/gather/cloud_lookup`
  4. Do: `set hostname www.zataz.com`
  5. Do: `run`

## Options

  **CENSYS_SECRET**

  Your Censys API SECRET.

  **CENSYS_UID**

  Your Censys API UID.

  **COMPSTR**

  You can use a custom string to perform the comparison.

  **HOSTNAME**

  This is the hostname [fqdn] on which the website responds. But this can also be a domain.

    msf5 auxiliary(gather/cloud_lookup) > set hostname www.zataz.com
    --or--
    msf5 auxiliary(gather/cloud_lookup) > set hostname discordapp.com

  **Proxies**

  A proxy chain of format type:host:port[,type:host:port][...]. It's optional.

  **RPORT**

  The target TCP port on which the protected website responds. Default: 443

  **SSL**

  Negotiate SSL/TLS for outgoing connections. Default: true

  **THREADS**

  Number of concurent threads needed for DNS enumeration. Default: 8

  **URIPATH**

  The URI path on which to perform the page comparison. Default: '/'

  **WORDLIST**

  Name list required for DNS enumeration. Default: ~/metasploit-framework/data/wordlists/namelist.txt

## Advanced options

  **DNSENUM**

  Set DNS enumeration as optional. Default: true

  **NS**

  Specify the nameserver to use for queries. Default: is system DNS

  **REPORT_LEAKS**

  Set to write leaked ip addresses in notes. Default: false

  **USERAGENT**

  Specify a personalized User-Agent header in HTTP requests. Default: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:56.0) Gecko/20100101 Firefox/56.0

  **TAG**

  Specify the HTML tag in which you want to find the fingerprint. Default: title
  Useful when combined with the CMPSTR option.

  **TIMEOUT**

  HTTP(s) request timeout. Default: 5

  **VERBOSE**

  You can also enable the verbose mode to have more information displayed in the console.

## Scenarios

### For auditing purpose

  If successful, you must be able to obtain the IP(s) address of the website as follows:
  ```
msf5 auxiliary(gather/cloud_lookup) > set verbose true
verbose => true
msf5 auxiliary(gather/cloud_lookup) > run

[*] Selected action: Amazon CloudFlare
[*] Passive gathering information...
[*]  * ViewDNS.info: 17 IP address found(s).
[*]  * DNS Enumeration: 6 IP address found(s).
[*] Clean Amazon CloudFlare server(s)...
[*]  * TOTAL: 10 IP address found(s) after cleaning.
[*]
[*] Bypass Automatic is in progress...
[*]  * Initial request to the original server for <title> comparison
[*]  * Trying: http://XXX.XXX.XXX.XXX:80/
[+] A direct-connect IP address was found: http://XXX.XXX.XXX.XXX:80/
[*]  * Trying: https://XXX.XXX.XXX.XXX:443/
      --> responded with an unhandled HTTP status code: 504
[*]  * Trying: http://XXX.XXX.XXX.XXX:80/
[*]  * Trying: https://XXX.XXX.XXX.XXX:443/
[*]  * Trying: http://XXX.XXX.XXX.XXX:80/
[+] A direct-connect IP address was found: http://XXX.XXX.XXX.XXX:80/
[*]  * Trying: https://XXX.XXX.XXX.XXX:443/
      --> responded with an unhandled HTTP status code: 504
[*]  * Trying: http://XXX.XXX.XXX.XXX:80/
[+] A direct-connect IP address was found: http://XXX.XXX.XXX.XXX:80/
[*]  * Trying: https://XXX.XXX.XXX.XXX:443/
      --> responded with an unhandled HTTP status code: 403
[*] Auxiliary module execution completed
  ```
  In this case 'A direct-connect IP address was found' is reported.

  However, some disreputable administrators used a simple redircetion (301 and 302)
  to force the passage through the WAF. This makes the IP address leak in the 'location'
  parameter of the HTTP header.

  For exemple:
  ```
  msf5 auxiliary(gather/cloud_lookup) > set hostname www.exodata.fr
  hostname => www.exodata.fr
  msf5 auxiliary(gather/cloud_lookup) > run

  [*] Selected action: Amazon CloudFlare
  [*] Passive gathering information...
  [*]  * ViewDNS.info: 3 IP address found(s).
  [*]  * DNS Enumeration: 12 IP address found(s).
  [*] Clean Amazon CloudFlare server(s)...
  [*]  * TOTAL: 4 IP address found(s) after cleaning.
  [*]
  [*] Bypass Automatic is in progress...
  [*]  * Initial request to the original server for <title> comparison
  [*]  * Trying: http://41.213.135.13:80/
  [*]  * Trying: https://41.213.135.13:443/
        --> responded with HTTP status code: 302 to http://www.exodata.fr/
  [!] A leaked IP address was found: https://41.213.135.13:443/
  [*]  * Trying: http://185.161.8.26:80/
        --> responded with HTTP status code: 302 to https://www.exodata.fr/
  [!] A leaked IP address was found: http://185.161.8.26:80/
  [*]  * Trying: https://185.161.8.26:443/
  [-] No direct-connect IP address found :-(
  [*] Auxiliary module execution completed
  ```
  --or--
  ```
  msf5 auxiliary(gather/cloud_lookup) > set verbose false
  verbose => false
  msf5 auxiliary(gather/cloud_lookup) > set hostname www.ingensecurity.com
  hostname => www.ingensecurity.com
  msf5 auxiliary(gather/cloud_lookup) > run

  [*] Passive gathering information...
  [*]  * ViewDNS.info: 2 IP address found(s).
  [*]  * DNS Enumeration: 8 IP address found(s).
  [*] Clean InGen Security (BinarySec EasyWAF) server(s)...
  [*]  * TOTAL: 4 IP address found(s) after cleaning.
  [*]
  [*] Bypass Automatic is in progress...
  [*]  * Initial request to the original server for <title> comparison
  [!] A leaked IP address was found: http://188.165.33.235:80/
  [-] No direct-connect IP address found :-(
  [*] Auxiliary module execution completed
  ```
  In this case 'A leaked IP address was found' is displayed but the bypass is NOT effective.

  You can also use the 'REPORT_LEAKS' option for write that in the notes.

  For some reason you may need to change the URI path to interoperate with other than the index page.
  To do this specific thing.

  For example:
  ```
  msf5 > use auxiliary/gather/cloud_lookup
  msf5 auxiliary(gather/cloud_lookup) > set HOSTNAME www.zataz.com
  hostname => www.zataz.com
  msf5 auxiliary(gather/cloud_lookup) > set URIPATH /contacter/
  uripath => /contacter/
  msf5 auxiliary(gather/cloud_lookup) > set compstr Contacter ZATAZ
  compstr => Contacter ZATAZ
  msf5 auxiliary(gather/cloud_lookup) > run
  ...
  ```
  --or--
  ```
  msf5 > use auxiliary/gather/cloud_lookup
  msf5 auxiliary(gather/cloud_lookup) > set HOSTNAME www.zataz.com
  hostname => www.zataz.com
  msf5 auxiliary(gather/cloud_lookup) > set URIPATH /contacter/
  uripath => /contacter/
  msf5 auxiliary(gather/cloud_lookup) > set compstr Contacter ZATAZ
  compstr => Contacter ZATAZ
  msf5 auxiliary(gather/cloud_lookup) > set tag html
  tag => html
  msf5 auxiliary(gather/cloud_lookup) > run
  ...
  ```

## References

  1. <https://citadelo.com/en/blog/cloudflare-how-to-do-it-right-and-do-not-reveal-your-real-ip/>