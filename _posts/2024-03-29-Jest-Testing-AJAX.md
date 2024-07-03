---
layout: post
title: Jest Testing - AJAX
keywords: [Jest, Test, Node.js, JavaScript, Ajax]
date: 2024-03-29
subtitle: How to writing test case of unit testing for AJAX API calls.
description: How to writing test case of unit testing for AJAX API calls.
author: Jensen Hsiao
banner:
  image: 'assets/images/banners/Jest.png'
  opacity: 0.9
image: 'assets/images/banners/Jest.png'
categories: Testing
tags:
  - Jest
  - Test
  - Node.js
  - JavaScript
  - Ajax
--- 
## AJAX 方法如何測試  

在測試方法內部如果使用了 AJAX 請求，但因為測試的重點在於方法內部邏輯的實作，所以並不需要每次測試都實際呼叫 API。  

- 假設有一個被測試方法使用到 Axios 來發送 AJAX 請求。  
    ```Javascript
    // axios.js
    const axios = require('axios');

    async function MockAxios() {
        try {
            const response = await axios.get('https://api.example.com/data');
            return response.data;
        } 
        catch (error) {
            throw error;
        }
    }
        
    module.exports = MockAxios;
    ```

### ─ 模擬方法  
#### ── jest.mock 模擬  
- 使用 jest.mock 模擬 axios，然後使用 axios.get.mockResolvedValue() 模擬 axios 回傳值，或是使用 mockRejectedValue() 來模擬回傳錯誤。  
<br>
    - 範例 :  
        ```Javascript
        // axios.test.js
        const axios = require('axios');
        const MockAxios = require('../src/axios');

        jest.mock('axios');

        describe('MockAxios with jest.mock', () => {
            test('MockAxios returns data from API', async () => {
                const mockData = { data: 'mock data' };
                axios.get.mockResolvedValue(mockData);

                const data = await MockAxios();

                expect(data).toEqual('mock data');
            });

            test('MockAxios handles errors', async () => {
                const errorMessage = 'Error fetching data';
                axios.get.mockRejectedValue(new Error(errorMessage));

                await expect(MockAxios()).rejects.toThrow(errorMessage);
            });
        });
        ```
        
#### ── axios-mock-adapter 模擬  
- axios-mock-adapter 套件提供多種模擬 API 的行為。  
<br>
    - 模擬請求和回應：  
        - 可以使用 onGet、onPost、onPut、onDelete 等方法來模擬各種 HTTP 請求。  
        - 可以指定特定的 URL 和參數，以及對應的回傳數據或錯誤。  
<br>
    - 設置回應狀態碼：  
        - 可以模擬不同的 HTTP 狀態碼，例如成功（200）、重定向（3xx）、客戶端錯誤（4xx）和服務器錯誤（5xx）等等。  
          
    - [more](https://www.npmjs.com/package/axios-mock-adapter)...  

    - 範例 :  
        ```bash
        npm install axios-mock-adapter
        ```

        ```Javascript
        // axios.test.js
        const axios = require('axios');
        const MockAdapter = require('axios-mock-adapter');

        describe('MockAxios with axios-mock-adapter', () => {
            let mock;

            test('MockAxios returns data from API', async () => {
                mock = new MockAdapter(axios);

                const mockData = { data: 'mock data' };
                mock.onGet('https://api.example.com/data').reply(200, mockData);

                const data = await MockAxios();

                expect(data).toEqual('mock data');
                mock.restore();
            });
        });
        ```


---

## Reference  
- [Jest Docs](https://jestjs.io/docs/getting-started)  
- [Jest Expect](https://jestjs.io/docs/expect)  
- [Jest Methods](https://jestjs.io/docs/api#methods)  
- [Jest Mock Functions](https://jestjs.io/docs/mock-functions)  
- [axios-mock-adapter](https://www.npmjs.com/package/axios-mock-adapter)  
