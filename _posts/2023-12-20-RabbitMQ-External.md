---
layout: post
title: RabbitMQ with External Auth
date: 2023-12-20
subtitle: How to use External Authentication on RabbitMQ 
author: Jensen Hsiao
banner:
  image: 'assets/images/banners/RabbitMQ_logo.png'
  opacity: 0.9
categories: Security
tags:
  - Security
  - Message
  - TLS/SSL
  - C Sharp
--- 
## Introduction
- 使用純文字帳號密碼驗證機制，現在會被軟體掃出中風險如下  
    - The remote Advanced Message Queuing Protocol (AMQP) service supports
one or more authentication mechanisms that allow credentials to be
sent in the clear.  

- 建議修改方式  
    - Disable cleartext authentication mechanisms in the AMQP configuration.  
    - 將連線與認證方始改用 External 機制，強制使用 SSL 認證連線  

---  

## RabbitMQ Server  
### - 啟用 External 機制  

- 開啟cmd  
    cd至rabbit_server底下的sbin資料夾  
    ```
    cd C:\Program Files\RabbitMQ Server\rabbitmq_server-3.9.4\sbin (參考依照實際安裝路徑)
    ```
- 檢查是否有rabbitmq-auth-mechanism-ssl plugin ?  
    ```
    rabbitmq-plugins list
    ```
- 啟用rabbitmq-auth-mechanism-ssl  
    - 使用 **Administrator** 帳號登入  
        ```
        rabbitmq-plugins enable rabbitmq_auth_mechanism_ssl
        ```
    - 一般帳號登入至以下路徑找到 **enabled_plugins** 檔案  
        ```
        C:\Users\Administrator\AppData\Roaming\RabbitMQ\enabled_plugins(參考)
        ```
        直接修改檔案內容將要開啟的 plugin 加入  
        ![enabled_plugins ](https://hackmd.io/_uploads/ryJJZzCO6.png)  

    - 以上設定完成後**重啟服務**  
        
    - 正確啟用後可以再次檢查會有"E"標記  
    ![螢幕擷取畫面 2024-01-09 114942](https://hackmd.io/_uploads/H1VqMH9Op.png)  


### - 設定 Server 雙向驗證  

- 伺服器端需要有以下三個檔案 (需要<span class="red">**安裝到系統**</span>中)  
    - Certificate Authority  
    - Certificate  
    - Private Key  
    ![Server憑證](https://hackmd.io/_uploads/r1nzcN6_p.png)  
    
- 修改 **advanced.config** (若無此檔可自行新增)  
    ![AdvancedConfig](https://hackmd.io/_uploads/ry3wcNTuT.png)  
    - {ssl, [{versions, ['tlsv1.2', ~~'tlsv1.1'~~]}]}, 啟用的tls版本   
        - (使用<span class="red"> tlsv1.2 </span>以上版本， tlsv1.1 協定已棄用，會被掃出中風險)  
              
    - {auth_mechanisms,[~~'PLAIN'~~, 'EXTERNAL']}, 啟用的驗證方式  
        - (身份驗證使用<span class="red"> EXTERNAL</span>，PLAIN 明文身份驗證，會被掃出中風險)  
              
    - {ssl_listeners, [5671]}, 指定 ssl port  
    
    - {ssl_options, []} 設定ssl連線憑證  
        - cacertfile 設定Certificate Authority檔案路徑  
        - certfile 設定Certificate 檔案路徑  
        - keyfile 設定Private Key 檔案路徑  
        - {verify, verify_peer} 雙向驗證  
        - {verify, verify_none} 不驗證Client端憑證  
        - {fail_if_no_peer_cert, true} Client端沒有提供憑證不可連線  
        - {fail_if_no_peer_cert, false} Client端沒有提供憑證可連線  

- Config 正確設定後可以檢查會有"EXTERNAL"標記  
    ![螢幕擷取畫面 2024-01-09 131408](https://hackmd.io/_uploads/ryK48U5_a.png)  

---  

## RabbitMQ Client  
### - 設定 Client 雙向驗證  

- Client端也需要有以下三個檔案 (需要<span class="red">**安裝到系統**</span>中)  
    - Certificate Authority  
    - Certificate  
    - Private Key  
    ![Server憑證](https://hackmd.io/_uploads/r1nzcN6_p.png)  
    
- 在 C# 中使用 RabbitMQ.Client 設定Ssl連線時，Ssl.CertPath 所讀取的檔案格式為 **PKCS#12** format   ([參考](https://www.rabbitmq.com/ssl.html))  

- 所以需要將上述 **Certificate** & **Private Key** <span class="red">合併</span>為 pfx 檔案。  
    ```
    //合併 certificate.crt & private.key 為 pfx 格式 (pass:設定密碼也可留空)
    openssl pkcs12 -export -in certificate.crt -inkey private.key -out client.pfx -passout pass:
    ```
    
- 或是使用伺服器中 Certificate Authority (CA.crt & CA.Key)產生 Client 端專用憑證。  
    ```
    //生成 client-key
    openssl genrsa -out client-key.pem
    
    //使用 client-key 設定主體名稱生成 signingrequest
    openssl req -new -key client-key.pem -out signingrequest.csr -subj "/CN=MockClient"
    
    //使用 signingrequest & CA.crt & CA.Key 生成 client-cert
    openssl x509 -req -days 1024 -in signingrequest.csr -CA RootCA.pem -CAkey RootCA.key -CAcreateserial -out client-cert.pem
    
    //將 client-cert.pem 格式轉換為 client-cert.crt
    openssl x509 -in client-cert.pem -out client-cert.crt
    
    //合併 client-cert.crt & client-key.pem 為 pfx 格式 (pass:設定密碼也可留空)
    openssl pkcs12 -export -in client-cert.crt -inkey client-key.pem -out client.pfx -passout pass:
    ```
    ![Client憑證](https://hackmd.io/_uploads/SyvHNST_a.png)  

- C# appsettings.json 調整新增參數  
```
  //CERT_PATH & CERT_PASSPHRASE 有多個，可以在同一個""內使用;區隔
  //支援執行多個host連線驗證
  "RABBITMQ_ENDPOINTS": {
    "EnableSSL": true,               //是否要啟用Ssl認證(bool)
    "CERT_PATH": "path/client.pfx",  //設定上述生成的 pfx 檔案路徑
    "CERT_PASSPHRASE": "",           //生成 pfx 檔案時設定的密碼
  }
```

- C# Client 端連線程式調整  
    - 使用 appsettings 中設定 bool 值，切換連線方式保留設定彈性。  
    - 增加 SslOption 設定。  

    ```csharp
    lock (_lock)
    {
        //...
        
        if (_env.EnableSSL)
        {
            var certPathList = _env.CertPath.Split(';');
            var certPassphraseList = _env.CertPassphrase.Split(';');
            
            factory = new ConnectionFactory
            {
                DispatchConsumersAsync = true,
                AutomaticRecoveryEnabled = true,
                AuthMechanisms = new AuthMechanismFactory[] { new ExternalMechanismFactory() },
            };
            
            foreach (var item in hostList)
            {
                var endPoint = new AmqpTcpEndpoint
                {
                    HostName = item,
                    Port = Convert.ToInt32(portList[index]),
                    Ssl = new SslOption
                    { 
                        Enabled = true,
                        ServerName = item,
                        CertPath = certPathList[index],
                        CertPassphrase = certPassphraseList[index++],
                        Version = SslProtocols.Tls12
                    }
                };

                addressList.Add(endPoint);
            }
            
            //...
            
        }
        else
        {
            //...
        } 
    } // lock
    ```

---  

## RabbitMQ Management  

- 當Server & Client 都設定好之後。  
- 需要再RabbitMQ Admin中添加生成憑證時設定的主機名，供登入時驗證用。  
![Admin](https://hackmd.io/_uploads/rJviZXAua.png)  

---  

## 驗證  

- 當以上都正確設定完成後，在 RabbitMQ Management 中可以確認，連線已啟用TLS/SSL協議  
  ![Connections](https://hackmd.io/_uploads/rkY6xQ0ua.png)  

- 並且 Authentication 確認是使用 EXTERNAL 機制  
  ![EXTERNAL](https://hackmd.io/_uploads/B1zNW7COa.png)  
  ![Ssl](https://hackmd.io/_uploads/HkpXM7Aup.png)  

- 使用 Swagger 測試  
    - 在網頁測試模擬 Publish 發送訊息  
    ![Swagger測試](https://hackmd.io/_uploads/Hk4zYXNna.png)
 
    
    - 後端 Subscribe 接收  
    ![模擬接收](https://hackmd.io/_uploads/HkK_DQVn6.png)  
  
 ---  
  
## Reference  
- [Self-signed certificate](https://jensenxiao.github.io/Self-signed-certificate)  
- [RabbitMQ - TLS Support](https://www.rabbitmq.com/ssl.html)  
- [RabbitMQ - google groups](https://groups.google.com/g/rabbitmq-users/c/Sxm0tNWCJuM?pli=1)  


<style>
    .red {
      color: red;
    }
    .Orange {
      color: Darkorange ;   
    }
    .Brown {
      color: SandyBrown;   
    }
    .yellow {
      color: Gold;   
    }
</style>
