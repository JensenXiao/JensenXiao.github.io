---
layout: post
title: SQL Percentage Sign Escaping
date: 2024-02-23
subtitle: How to escape a percentage sign in SQL
author: Jensen Hsiao
banner:
  image: 'assets/images/banners/Oracle-SQLServer.png'
  opacity: 0.9
categories: Markdown
tags:
  - SQL
  - C Sharp
--- 
## SQL 語法將 ％ 字元視為萬用字元  

- 系統在做模糊查詢時，如果要查詢的字串包含有 ％ 字元，會被 SQL 認為是萬用字元  

- 所以針對使用者輸入的變數，檢查是否有 % 字元，將其使用跳脫符號避免結果與預期不同  
    - **Ms sql**  
        - 使用 [] 跳脫  
        
        - 將 % 替換成 [%]  
        
    - **Oracle**  
        - 需要在 LIKE 參數後面使用 ESCAPE 關鍵字，定義符號 \ 為跳脫符號  
        
        - 將 % 替換成 \%  

- 修改 C# 中組合 SQL 語句處，找尋 param 中有含 % 字元的，針對不同 DBType 做SQL語法處理  

    ```csharp
    //...

    // 如果值包含 '%' 字元，則對 SQL 語法處理
    if (param.ContainsKey(name) && param[name].Contains("%"))
    {
        var escapedVal = "";

        switch (DBType)
        {
            case MsSqlServer:
                // 在 SQL Server 中將 % 替換為 [%] 跳脫
                escapedValue = param[name].Replace("%", "[%]");

                // 替換 SQL 語法中的參數值
                Sql = Sql.Replace(m.Groups["param"].Value, "'" + escapedVal + "'");
                break;

            case Oracle:
                // 在 Oracle 中將 % 替換為 \% 跳脫
                escapedValue = param[name].Replace("%", @"\%");

                // 如果是 LIKE '%'||'param' 子句
                // 替換 SQL 語法中的值，以及加入 ESCAPE '\'
                if (string.IsNullOrEmpty(m.Groups["escape"].Value))
                {
                    Sql = Sql.Replace(m.Groups["param"].Value, "'" + escapedVal + "' ESCAPE '\\'");
                }
                else
                {
                    // 如果是 LIKE '%'||'param%'||'%' 子句
                    // 則替換 SQL 語法中的值，和替換後面的 '%' 子句，加入 ESCAPE '\'
                    Sql = Sql.Replace(m.Groups["param"].Value, "'" + escapedVal + "'");
                    Sql = Sql.Replace(m.Groups["escape"].Value, "||'%' ESCAPE '\\'");
                }
                break;
        }
    }
    ```
