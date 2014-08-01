
* **store_loot()**: Used to store both stolen files (both text and binary) and "screencaps" of commands such as a ```ps -ef``` and ```ifconfig```. The file itself need not be of forensic-level integrity -- they may be parsed by a post module to extract only the relevant information for a penetration tester.

* **report_auth_info()**: Used to store working credentials that are immediately reusable by another module. For example, a module dumping the local SMB hashes would use this, as would a module which reads username:password combinations for a specific host and service. Specifically, merely "likely" usernames and passwords should use store_loot() instead.

* **report_vuln()**: Auxiliary and post modules that exercise a particular vulnerability should report_vuln() upon success. Note that exploit modules automatically report_vuln() as part of opening a session (there is no need to call it especially).

* **report_note()**: Modules should make an effort to avoid report_note() when one of the above methods would be a better fit, but there are often cases where "loot" or "cred" or "vuln" classifications are not immediately appropriate. report_note() calls should always set a OID-style dotted :type, such as domain.hosts, so other modules may easily find them in the database.

* **report_host()** -

* **report_service()** - 

* **report_client()** - 

* **report_artifact()** -

* **report_web_site()** - 

* **report_web_page()** - 

* **report_web_form()** - 

* **report_web_vuln()** - 

* **report_loot()** -

### Reference

https://github.com/rapid7/metasploit-framework/wiki/Guidelines-for-Accepting-Modules-and-Enhancements

https://github.com/rapid7/metasploit-framework/blob/master/lib/msf/core/auxiliary/report.rb