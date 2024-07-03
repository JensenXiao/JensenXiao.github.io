---
layout: post
title: Jest Testing - Asynchronous 
keywords: [Jest, Test, Node.js, JavaScript, Asynchronous]
date: 2024-04-12
subtitle: How to writing test case of unit testing for Callbacks & Promise calls.
description: How to writing test case of unit testing for Callbacks & Promise calls.
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
  - Asynchronous 
--- 
## Asynchronous 非同步程式測試  

- 在測試方法時內部可能有 Callbacks 或 Promise 之類可能是非同步情況下，有可能會出現測試通過但報告卻顯示沒有被測試到。  

- 假設有一個被測試檔案中有非同步的函數方法。  
    ```Javascript
	// asynchronous.js
	const Asynchronous = {
		// 使用 Callbacks 測試的非同步函數
		fetchDataWithCallback(callback) {
			setTimeout(() => {
				callback('This is a callback');
			}, 3000);
		  },

		// 使用 Promise 測試的非同步函數
		fetchDataWithPromise() {
			return new Promise((resolve, reject) => {
				setTimeout(() => {
					resolve('This is a Promise');
				}, 3000);
			});
		}
	};

	module.exports = Asynchronous;
    ```
<br>

- 接著寫了一個測試案例來呼叫被測試的方法，然後接著驗證方法回傳的資料是不是你所預期的。  
    ```Javascript
	// asynchronous.test.js
	const Asynchronous = require('asynchronous');

	describe('Asynchronous Tests', () => {
		// 測試 Callbacks
		test('Testing Callbacks', () => {
			Asynchronous.fetchDataWithCallback((data) => {
				expect(data).toBe('This is a callback');
			});
		});

		// 測試 Promise
		test('Testing Promise', () => {
			Asynchronous.fetchDataWithPromise().then((data) => {
				expect(data).toBe('This is a Promise');
			});
		});
	});
    ```
<br>

- 那這邊當你寫好測試之後實際去執行測試，也會看到測試如預期的通過了。  

![非同步測試通過](https://hackmd.io/_uploads/HkdjZ2vkC.png)  
<br>
 
- 但是這時候會發現測試顯示通過了，但是覆蓋率的數字卻沒有變化或是變化很少，這個時候如果你去看詳細的覆蓋率報告會看到，非同步相關的程式碼並沒有被測試覆蓋到。  

![非同步測試](https://hackmd.io/_uploads/Hk5sjtzvR.png)  
<br>  

- 這是因為 Jest 的測試運行機制，在執行測試時它 **不會** 等待測試函數內的所有非同步操作完成，而是認為測試已經完成，並測試通過，針對這種情況在測試時可以針對非同步的方法強制等待回傳值，並對其驗證。  


### ─ Callbacks  

- 在 test ( ) 內使用 done argument，不要用 empty argument，Jest 就會等到 done callback 被呼叫時才會結束測試。  
	```Javascript
	// asynchronous.test.js
	// 測試 Callbacks
	test('Testing Callbacks', (done) => {
		Asynchronous.fetchDataWithCallback((data) => {
			try {
				expect(data).toBe('This is a callback');
				done(); // 在測試完成時調用 done
			} catch (error) {
				done(error); // 如果出現錯誤，也調用 done 並傳遞錯誤
			}
		});
	}, 5000); // 設置測試超時時間為 5000 毫秒（5 秒）
	```


### ─ Promise  

- 在 Promise 多種方式可以使用  
	1. 使用 return 等待回傳的 promise 被 resolve 才會結束測試。  
	2. 使用 async / await 來測試 Promise。  
	3. 在 expect 中使用 .resolves / .rejects matcher。  
	
	```Javascript
	// asynchronous.test.js
	
	// Using return
	test('Testing Promis', () => {
		return Asynchronous.fetchDataWithPromise().then((data) => {
			expect(data).toBe('This is a Promise');
	});

	// Using async/await
	test('Testing Promise using async/await', async () => {
		const data = await Asynchronous.fetchDataWithPromise();
		expect(data).toBe('This is a Promise');
	});
		
	// Using .resolves matcher
	test('Testing Promise with return', () => {
		return expect(Asynchronous.fetchDataWithPromise()).resolves.toBe('This is a Promise');
	});
	```
<br>

- 也可以把不同的方法搭配一起使用。  
	- 例如 async/await & .resolves matcher  
		```Javascript
		// asynchronous.test.js
		test('Testing Promise using async/await & .resolves matcher', async () => {
			await expect(Asynchronous.fetchDataWithPromise()).resolves.toBe('This is a Promise');
		});
		```


### ─ Result  

- 當使用前面提到的任何一種方法來修改測試案例之後，再去查看覆蓋率報告，就可以發現那些非同步的程式碼，都有確實被測試覆蓋到了。  

![非同步測試成功](https://hackmd.io/_uploads/B1F1JqGwA.png)  


---

## Reference  
- [Jest Docs](https://jestjs.io/docs/getting-started)  
- [Jest Expect](https://jestjs.io/docs/expect)  
- [Jest Methods](https://jestjs.io/docs/api#methods)  
- [Jest Mock Functions](https://jestjs.io/docs/mock-functions)  
- [Jest Testing Asynchronous](https://jestjs.io/docs/asynchronous)  