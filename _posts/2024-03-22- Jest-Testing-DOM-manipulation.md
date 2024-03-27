---
layout: post
title: Jest Testing for DOM manipulation
date: 2024-03-22
subtitle: How to write test case of unit testing for DOM manipulation.
author: Jensen Hsiao
banner:
  image: 'assets/images/banners/Jest.png'
  opacity: 0.9
categories: Testing
tags:
  - Test
  - Node.js
  - JavaScript
--- 
## Jest 外部依賴 ＆ 模擬

- 在 Jest 測試框架中，不具備網頁環境。  

- 對於網頁操作需要用到的 Library ，需要用引入模組的方式使其能夠正常運作。  

- 而若需要對網頁做 DOM manipulation ，需要用模擬的方式創建 DOM 環境。  

### ── 引入模組  

-  若是環境相關的引用可以另外寫一隻 setupTests.js 在裡面做全域宣告（global.）的方式，並在上一篇 [Jest Testing](https://jensenxiao.github.io/testing/2024/03/15/Jest-Testing.html) 中提到的 package.json 裡設定，能夠在執行每個測試檔案前先執行 setupTests.js 做 initial。  

- Jest 測試框架，需要使用的模組需要透過 npm 安裝  
    - 例如：jquery、kendo-ui、i18next  
<br>
- **jquery**  
    - Jest 使用 Node.js，而在 Node.js 中並沒有內建的 ﹩ 符號，它不會自動將 ﹩ 視為 jQuery。所以，為了在 Jest 測試中使用 ﹩ 符號來調用 jQuery 函數，你需要將 ﹩ 指派給 jQuery。  
    
    ```JavaScript
    // setupTests.js
    // 載入 jQuery 並指派 $ 使用 jquery 模組引用
    const jquery = require('jquery');
    global.jQuery = jquery;
    global.$ = jquery;
    ```
<br>
- **kendo-ui**  
    - 要在 Jest 測試中使用 Kendo UI 提供的相關功能進行測試，需要引用 @progress/kendo-ui，而在 kendo 模組用會用到 TextEncoder 與 TextDecoder，所以也要引入。  
    
    ```JavaScript
    // setupTests.js
    global.TextEncoder = require('util').TextEncoder;
    global.TextDecoder = require('util').TextDecoder;
    global.kendo = require('@progress/kendo-ui');
    ```
<br>
- **i18next**  
    - 將 i18next 函數引用並全域宣告  
    
    ```JavaScript
    // setupTests.js
    const i18next = require('i18next');
    global.i18next = i18next;
    ```
    

### ── 引入外部依賴  

- 使用 require 引入模組  
<br>
    - require 是 Node.js 中常見的引入模組的方式。  

    - 要將 JavaScript 檔案變成一個模組，需要將其中的功能進行封裝，然後透過 module.exports 將這些功能暴露給其他程式碼。
    
    - 這樣其他程式碼就可以通過 require 或 import 的方式引入你的模組並使用其中的功能。
    
    - 以下舉例，定義了一個函數 myFunction，並使用 module.exports 將這個函數暴露出來。
    
        ```JavaScript
        // Module.js
        function TestFunction() {
          console.log('This is TestFunction');
        }

        module.exports = {
          TestFunction: TestFunction
        };
        ```
<br>
   - 可以在測試的檔案中引入 Module.js 並使用 TestFunction。  
   
        ```JavaScript
        // Module.test.js
        const Module = require('./Module');

        Module.TestFunction(); // 調用 TestFunction 函數
        ```
<br>
   - 或是在 setupTests.js 進入點全域載入。  
   
        ```JavaScript
        // setupTests.js
        global.Module = require('./Module');

        // Module.test.js
        Module.TestFunction(); // 調用 TestFunction 函數
        ```
<br>
   - 而要讓測試檔案可以正確找到模組，需要在 package.json 文件中添加 moduleDirectories 設置，模組放置的路徑。  
   
       ```XML
        "jest": {
            "moduleDirectories": [
              "../path/folder"
            ]
        },
       ```
<br>
- 或是使用 fs.readFileSync 與 eval 讀取並執行檔案 (這個方式無法追蹤覆蓋率)。  
<br>
    - 不包成模組可以用 Node.js 的 fs 模組來讀取指定路徑的 JavaScript，並使用 eval 函數將其執行。  
    
        ```JavaScript
        // setupTests.js
        const path = require('path');
        const fs = require('fs');
        global.Module = fs.readFileSync(path.resolve(__dirname, '../path/folder/Module.js'), 'utf-8');
        eval(global.Module);
        ```
   
    
### ── 模擬 DOM 環境  

- Jest 使用 Node.js 環境來執行測試，在測試運行時沒有真正的 DOM 環境可供測試 DOM 相關的程式碼。  

- 將 "testEnvironment": "jest-environment-jsdom" 添加到 package.json 配置中，Jest 將會使用 JSDOM 來模擬一個瀏覽器環境。  

    - jest-environment-jsdom 提供了一個基於 jsdom 的模擬 DOM 環境，用於在 Node.js 中模擬瀏覽器環境進行測試。可以在執行測試時訪問 DOM 元素、執行 DOM 操作以及測試與 DOM 相關的程式碼，而無需實際打開瀏覽器。  

        ```XML
        "jest": {
            "testEnvironment": "jest-environment-jsdom"
        },
        ```
<br>
- 引入 jsdom ，並 mock 環境以便使用 DOM API。  

    ```JavaScript
    // setupTests.js
    const jsdom = require('jsdom');
    const { JSDOM } = jsdom;

    // 模擬 DOM 環境
    const dom = new JSDOM('<!doctype html><html><body></body></html>');
    global.window = dom.window;
    global.document = dom.window.document;
    global.navigator = dom.window.navigator;
    global.HTMLElement = dom.window.HTMLElement || class {};
    ```


### ── 模擬 window  

- 在測試中有些會用到 window 方法，這邊用模擬的 window 方法來代替真實的 window 方法。  
<br>
    - 利用 Jest 提供的 jest.fn()，建立一個模擬函式。  
    
    - 設置 global.window.close 為一個模擬函式，當在測試中呼叫 window.close() 時，實際上是呼叫了這個模擬函式。  

        ```JavaScript
        // setupTests.js
        global.window.close = jest.fn();
        global.window.alert = jest.fn();
        global.window.confirm = jest.fn();
        Object.defineProperty(window, 'location', {
          value: { href: 'http://example.com' },
          writable: true,
        });
        ```

---

## Reference  
- [Jest Docs](https://jestjs.io/docs/getting-started)  
- [Jest Methods](https://jestjs.io/docs/api#methods)  
