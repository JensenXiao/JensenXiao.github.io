---
layout: post
title: Self-signed certificate
keywords: [Security, TLS/SSL, OpenSSL, Self-signed, Certificate]
date: 2023-6-21
subtitle: Generating a self-signed certificate using OpenSSL
description: Generating a self-signed certificate using OpenSSL
author: Jensen Hsiao
banner:
  image: 'assets/images/banners/OpenSSL_logo.png'
  opacity: 0.9
image: 'assets/images/banners/OpenSSL_logo.png'
categories: Security
tags:
  - Security
  - TLS/SSL
tags: [Security , TLS/SSL]
--- 

## Generate Root CA 

- 產生 RootCA.key & RootCA.pem  
    - CN 主機名稱  
    - days 憑證期限  
    
    ```
openssl req -x509 -nodes -new -sha256 -days 1024 -newkey rsa:2048 -keyout RootCA.key -out RootCA.pem -subj "/C=TW/CN=MockRoot-CA"
    ```
  
- 將 RootCA.pem 轉為 RootCA.crt 格式  
```
openssl x509 -outform pem -in RootCA.pem -out RootCA.crt
```

---  

## Generate a Server Certificate  

- 產生 Server-Private.key & Server.csr  
    - CN 主機名稱  
    
    ```
    openssl req -new -nodes -newkey rsa:2048 -keyout  Server-Private.key -out Server.csr -subj "/CN=MockServer"
    ```
- 新增 "domains.ext" 檔案，裡面設定所需資料  
    - DNS 設定主機別名，可設定多個 DNS.1、DNS.2、DNS.3...  
    - IP 設定主機 IP 位置，可設定多個 IP.1、IP.2、IP.3...  
    ![domains.ext](https://hackmd.io/_uploads/BkDkcfC_6.png)  
    
- 使用 Server.csr & RootCA.pem & RootCA.key & domains.ext 生成 Server-Certificate.crt  
``` 
openssl x509 -req -sha256 -days 1024 -in Server.csr -CA RootCA.pem -CAkey RootCA.key -CAcreateserial -extfile domains.ext -out Server-Certificate.crt
```
- 所有產生出的檔案  
![Server憑證生成](https://hackmd.io/_uploads/H1l9DGAOp.png)  

---  

## Generate a Client Certificate  

- 生成 client-key  
```
openssl genrsa -out client-key.pem
```

- 使用 client-key 設定主體名稱生成 signingrequest  
    - CN 主機名稱  
    ```
openssl req -new -key client-key.pem -out signingrequest.csr -subj "CN=MockClient"
    ```
    
- 使用 signingrequest & RootCA.crt & RootCA.Key 生成 client-cert  
```
openssl x509 -req -days 1024 -in signingrequest.csr -CA RootCA.pem -CAkey RootCA.key -CAcreateserial -out client-cert.pem
```

- 將 client-cert.pem 格式轉換為 client-cert.crt  
```
openssl x509 -in client-cert.pem -out client-cert.crt
```

- 合併 client-cert.crt & client-key.pem 為 pfx 格式  
    - pass : 設定密碼(可留空)  
    ```
openssl pkcs12 -export -in client-cert.crt -inkey client-key.pem -out client.pfx -passout pass:
    ``` 

- 所有產生出的檔案  
![Client憑證](https://hackmd.io/_uploads/SyvHNST_a.png)  

---  

## Reference  
- 憑證認證方式  
    - 當產生憑證時，CA 會提供我們憑證（certificate.crt）。  
    - 通常還會提供一個包含根憑證和中間憑證的 CA Bundle File（ca_bundle.crt）。  
    - 而在驗證時，當接收到一個 SSL 憑證它會使用 CA bundle 來驗證該憑證的有效性，所以 Server 與 Client 兩端使用的 Certificate & Private Key 不同沒關係，只要是同一個CA 發行的憑證都可以認證通過。
