---
layout: post
title: LDAP Injection
date: 2024-02-29
subtitle: How to Prevent LDAP Injection
author: Jensen Hsiao
banner:
  image: 'assets/images/banners/LDAP-Injection.png'
  opacity: 0.9
categories: Security
tags:
  - Security
  - Checkmarx
  - C Sharp
--- 
## 弱點風險 
- Checkmarx 弱點掃描掃出，使用 DirectorySearcher 建立 LDAP 查詢句，其中 search.Filter 使用沒有被處理過不受信任的字串，可能會有被注入攻擊的機會。  
    ```csharp
    DirectorySearcher search = new DirectorySearcher(directoryEntry);

    // 搜尋 AD 帳號名稱為 USER_ID 的項目
    search.Filter = "(SAMAccountName=" + Obj.USER_ID + ")";
    ```
  
- 這邊在 [AntiXSS 4.0 Released](https://learn.microsoft.com/zh-tw/archive/blogs/securitytools/antixss-4-0-released) 程式庫中有提供 LDAP Encoding changes 的方法，可以透過 Encode 的方式將字串轉義，來避免 LDAP Injection。  
<br>
- 開啟 NuGet 套件管理員，搜尋並安裝 AntiXSS 。  
![AntiXSS](https://hackmd.io/_uploads/rkFE0gs2T.png)  
<br>
- 其中針對 LDAP 有提供了三個方法  
    - Encoder.LdapFilterEncode(string)：  
        - 用於編碼 LDAP 查詢過濾器中的值，如 (、)、*、\、/、NUL 等特殊字符。  
        - 會對這些特殊字符進行轉義，轉義為 LDAP 查詢語法中的安全表示形式。  
<br>
    - Encoder.LdapDistinguishedNameEncode(string)：  
        - Distinguished Name（DN）在創建或修改LDAP條目時使用的屬性值。  
        - 用於編碼LDAP分類名稱（DN）中的值，如 ,、=、+、<、>、#、;、\。  
        - 會對這些特殊字符進行轉義，轉義成為LDAP DN中的安全表示形式。  
<br>
    - Encoder.LdapDistinguishedNameEncode(string, bool, bool)：  
        - 這個重載方法允許你指定額外的參數來控制編碼的行為。  
        - 第一個參數是要編碼的字符串。  
        - 第二個參數，用於指示是否編碼域（即 @ 後面的部分）中的特殊字符。  
            - 設置為 true，則會對域中的特殊字符進行轉義，默認為 false。  
        - 第三個參數，用於指示是否編碼空格。  
            - 設置為 true，則會對空格進行轉義，默認為 true。  
<br>
- 通過使用 Encoder.LdapFilterEncode 方法對變數進行編碼，可以有效地解決潛在的弱點風險。  
    ```csharp
    using Microsoft.Security.Application;

    DirectorySearcher search = new DirectorySearcher(directoryEntry);

    // 對字串編碼
    string filteredUserId = Encoder.LdapFilterEncode(Obj.USER_ID);

    // 搜尋 AD 帳號名稱為 USER_ID 的項目
    search.Filter = "(SAMAccountName=" + filteredUserId + ")";
    ```

## Reference  
- [LDAP Injection Prevention](https://cheatsheetseries.owasp.org/cheatsheets/LDAP_Injection_Prevention_Cheat_Sheet.html)  
- [AntiXSS 4.0 Released](https://learn.microsoft.com/zh-tw/archive/blogs/securitytools/antixss-4-0-released)  
