---
layout: post
title: Jest Testing - XMLHttpRequest
keywords: [Jest, Test, Node.js, JavaScript]
date: 2024-04-05
subtitle: How to writing test case of unit testing for XMLHttpRequest calls.
description: How to writing test case of unit testing for XMLHttpRequest calls.
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
--- 
## XMLHttpRequest 如何模擬  

在被測試方法中，如果使用到了 XMLHttpRequest 物件來發送 HTTP 請求，因為測試的重點在於方法內部邏輯的實作，所以並不需要在測試中實際發送 HTTP 請求。  

- 假設有一個被測試方法使用到 XMLHttpRequest 物件來發送 HTTP 請求。  
    ```Javascript
	const XMLHttpRequest = require("xmlhttprequest").XMLHttpRequest;

	function fetchData(url, callback) {
	  const xhr = new XMLHttpRequest();
	  xhr.open('GET', url);
	  xhr.onload = function() {
		if (xhr.status === 200) {
		  callback(null, JSON.parse(xhr.responseText));
		} else {
		  callback(new Error('Failed to load data: ' + xhr.status));
		}
	  };
	  xhr.onerror = function() {
		callback(new Error('Network error'));
	  };
	  xhr.send();
	}

	module.exports = fetchData;
    ```
- 其中他會透過 XMLHttpRequest 來發送 HTTP 請求，從指定的 URL 獲取數據。  
![XMLHttpRequest](https://hackmd.io/_uploads/SJdRndMwC.png)  

### ─ 模擬方法  

#### ── nock 模擬  
- 這邊可以透過 nock 來模擬 HTTP 請求，這樣可以在不真正發送網絡請求的情況下測試程式碼。  
<br>
    - 範例 :  
        ```Javascript
		const nock = require('nock');
		const fetchData = require('./fetchData');

		// 測試 fetchData 函數
		test('測試 fetchData 是否能從 API 正確回傳數據', done => {
		  nock('https://api.example.com')
			.get('/data')
			.reply(200, JSON.stringify({ id: 1, name: 'Jensen' }));

		  fetchData('https://api.example.com/data', (err, data) => {
			expect(err).toBeNull();
			expect(data).toEqual({ id: 1, name: 'Jensen' });
			done(); // 使用 done 來處理異步測試
		  });
		});
        ```

- 當你有設置 nock 攔截之後，每當你的測試程式有請求被發送時，其中有對應到你所設置的 URL 和路徑上，就會被攔截並根據 .reply() 方法中你所預期的內容來回應。  
![nock](https://hackmd.io/_uploads/SJZcpdzwC.png)  


---

## Reference  
- [nock](https://www.npmjs.com/package/nock?activeTab=readme)  
- [Jest Docs](https://jestjs.io/docs/getting-started)  
- [Jest Expect](https://jestjs.io/docs/expect)  
- [Jest Methods](https://jestjs.io/docs/api#methods)  
- [Jest Mock Functions](https://jestjs.io/docs/mock-functions)  