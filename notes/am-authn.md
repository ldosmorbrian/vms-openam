# AM Authentication


Chain: certChain

certModule (Certificate) --> Sufficient --> Pass
|
continue
|
V
DataStore (Data Store) --> Sufficient --> Pass
|
continue
|
V
???? Requires at least one Pass, No Fail


----
Global Configure --> Authentication --> Core -->
(tab) Post Auth Processing: Default Success Login URL --> /openam/console

(tab) User Profile: Required

---
GC --> AUthentication --> Certificate
Subject DN ... : CN
Issuer DN Attr: CN

Cert field used to access User Profile: subject CN

LDAP Server Where Certs are Stored: idam hosted LDAP (hostname:port)



