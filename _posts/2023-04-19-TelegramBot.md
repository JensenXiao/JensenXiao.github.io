---
layout: post
title: Telegram Bot
keywords: [Message, Node.js, Node, Telegram Bot, Telegram]
date: 2023-04-19
subtitle: How to use Telegram Bot in Node.js
description: How to use Telegram Bot in Node.js
author: Jensen Hsiao
banner:
  image: 'assets/images/banners/Telegram-Coin-Logo.jpg'
  opacity: 0.9
image: 'assets/images/banners/Telegram-Coin-Logo.jpg'
categories: Markdown
tags:
  - Message
  - Node.js
--- 

## Creating a bot using BotFather  
- 打開 Telegram，在搜尋框輸入 @BotFather，點擊第一項的搜尋結果。  
    ![BotFather](https://hackmd.io/_uploads/Bk5linasp.png)  

- 輸入 /start 會出現支援的指令說明。  
    ![start](https://hackmd.io/_uploads/rJ1_nhToa.png)  

- 輸入 /newbot 建立機器人。  
    ![newbot](https://hackmd.io/_uploads/SJ_Manpsa.png)  
    
- 輸入機器人的名稱  

    - 必須以bot結尾。  
    ![命名規則1](https://hackmd.io/_uploads/ByN41TTsp.png)  
	
    - 若有人重複使用會請你重新設置。  
    ![命名規則2](https://hackmd.io/_uploads/B13E1pTj6.png)  
	

- 創建成功會取得 **機器人名稱** & **token**。  
    ![Done](https://hackmd.io/_uploads/rJkie6Ti6.png)  

- 在 Telegram 搜尋機器人就可以找到你所創建的機器人。  

---  

## Find Group ChatID  
- 首先需要將創建好的機器人加入聊天群組。  

- 在瀏覽器中輸入 https://api.telegram.org/bot[your_token]/getUpdates  
    - [your_token] 改為在創建時獲得的token  

- 會回傳一段 JSON。  
    - "chat" object 中 group_ID 即是群組的 ChatID  
```
{
	"update_id":8393,
	"message":
    {
			"message_id":3,
			"from":{"id":7474,"first_name":"AAA"},
			"chat":{"id":<group_ID>,"title":""},
			"date":25497,
			"new_chat_participant":{"id":71,"first_name":"Name","username":"YourBotName"}
    }
}
```
    - If you created the new group with the bot and you only get {“ok”:true,“result”:[]}。  
```
{
	"ok": true,
    "result": []
}
```
    - remove and add the bot again to the group。  

---  

## Using in Node.js  
- cd 至專案路徑。  
    ``` js
    $ cd Testbot
    ```
- npm init，會建立 package.json 方便管理 npm 的 library。  
    ``` js
     npm init
    ```
- 安裝Telegram Bot API。  
    ``` js
    npm install node-telegram-bot-api --save

    or

    npm install --save-dev @types/node-telegram-bot-api
    ```
    
- 在程式中應用  

	``` js
	var TelegramBot = require('node-telegram-bot-api');
	var token = 'your_token';
	var groupId = 'your_groupId' ;
	
	//使用Long Polling的方式與Telegram伺服器建立連線
	//括號裡面的內容需要改為在創建時獲得的token
	var bot = new TelegramBot(token, {polling: true});
	
	//收到Start訊息時會觸發這段程式
	bot.onText(/\/start/, function (msg) {
		var chatId = msg.chat.id; //用戶的ID
		var resp = '你好'; //回應內容
		bot.sendMessage(chatId, resp); //發送訊息的function
	});
	 
	//收到/cal開頭的訊息時會觸發這段程式
	//接收用戶發送的數學表達式，計算其結果，然後將結果回傳給用戶。
	bot.onText(/\/cal (.+)/, function (msg, match) {
		var fromId = msg.from.id; //用戶的ID
		var resp = match[1].replace(/[^-()\d/*+.]/g, '');
		resp = '計算結果為: ' + eval(resp);
		bot.sendMessage(fromId, resp);
	});

	setInterval(() => {
		bot.sendMessage( groupId, "測試發送" );
	}, 5000);
	```

---  

## Reference  
- [Node.js Telegram Bot API](https://github.com/yagop/node-telegram-bot-api)  
- [API Reference](https://github.com/yagop/node-telegram-bot-api/blob/master/doc/api.md)  
