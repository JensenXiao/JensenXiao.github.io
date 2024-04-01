---
layout: post
title: Jest Testing - Introduction
date: 2024-03-15
subtitle: How to create a test cases in Jest
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
## Jest 前端測試框架  
 
- 目前在專案中希望能夠配合 CI/CD，以進行自動化測試。除了對後端進行自動化測試外，也希望能針對前端進行測試。  

- 因此，我寫了一個使用 Jest 框架來建立前端測試環境的專案。未來各專案可以 clone 這份專案，並與 CI/CD 配合進行前端測試的開發。  

- 在這個專案中，順手記錄了一些 Jest 框架應用，以及前端測試可能會用到的方法。  


### ─ 專案結構  

以下是 Jest 專案常見的結構。  
- <span class="red">test_project/</span>  
    - <span class="red">src/</span>  
        - components/  
            - Component1.js  
            - Component2.js  
        - utils/  
            - util1.js  
            - util2.js  
    - <span class="red">tests/</span>  
        - unit/  
            - component1.test.js  
            - component2.test.js  
        - integration/  
            - integration_test1.test.js  
            - integration_test2.test.js  
    - <span class="red">coverage/</span>  
        - lcov-report/  
            - index.html  
    - node_modules/  
    - <span class="red">package.json</span>  
    - setupTests.js  
<br>
- src/：這裡存放需要被測試的原始碼文件。  
- tests/：這裡存放針對 src/ 內程式碼撰寫的測試案例文件。  
- coverage/：這裡存放測試後生成的程式碼覆蓋率報告文件。  
- package.json：這個檔案包含了與測試相關的配置設定。  


### ─ 測試相關配置  

- Jest 的 package.json 可以設定測試的相關配置，這些配置通常包括測試的入口文件、測試覆蓋率報告、以及其他一些自定義配置。  

```json
{
  "name": "your-project-name",
  "version": "1.0.0",
  "description": "Description of your project",
  "scripts": {
  "test": "jest"
  },
  // 專案運行時所需的環境依賴
  "devDependencies": {
    "jest": "^29.7.0",
    "jsdom": "^24.0.0",
    "jest-mock-axios": "^4.7.3",
    "jest-environment-jsdom": "^29.7.0"
  },
  // 專案運行時所需的外部依賴
  "dependencies": {
    "jquery": "^3.7.1"
  },
  // 定義 Jest 測試 option
  "jest": {
    "rootDir": "../",
    "roots": [
      "<rootDir>/test_project"
    ],
    // 指定在執行測試之前要執行的腳本文件
    "setupFiles": [
      "<rootDir>/test_project/setupTests.js"
    ],
    // 指定 Jest 測試的環境
    "testEnvironment": "jsdom",
    // 指定 Jest 測試文件的匹配模式
    "testMatch": [
      "**/__tests__/**/*.js", 
      "**/?(*.)+(spec|test).js"
    ],
    // 定義覆蓋率的閾值
    "coverageThreshold": {
      "global": {
        "branches": 80,
        "functions": 80,
        "lines": 80,
        "statements": 80
      }
    },
    // 指定哪些文件需要收集覆蓋率訊息
    "collectCoverageFrom": [
      "src/**/*.js"
    ],
    // 指定生成覆蓋率報告的格式
    "coverageReporters": [
      "json",
      "lcov",
      "text",
      "clover"
    ],
    // 指定覆蓋率報告的輸出目錄
    "coverageDirectory": "coverage",
  }
}
```

- **devDependencies** : 在這個節點中，列出了專案運行時所需的環境依賴。  
    - 這邊可以使用 npm install --save-dev jest 或 yarn add --dev jest 來安裝 Jest。  
<br>
- **dependencies** : 在這個節點中，列出了專案運行時所需的外部依賴。  
    - 這邊可以使用 npm install 來安裝第三方 module。  
<br>
- **jest** : 在這個節點中，定義了 Jest 測試的配置選項：  
    - setupFiles: 指定在執行測試之前要執行的腳本文件。  
        - 可以在執行測試之前，預先執行一些全域設置、初始化或配置測試環境等。  
        ![全域設置](https://hackmd.io/_uploads/S1WphCvk0.png)  
    - testEnvironment: 指定 Jest 測試的環境。  
    - testMatch: 指定 Jest 測試文件的匹配模式。  
    - coverageThreshold: 定義代碼覆蓋率的閾值。  
    - collectCoverageFrom: 指定哪些文件需要收集代碼覆蓋率訊息。  
    - coverageReporters: 指定生成覆蓋率報告的格式。  
    - coverageDirectory: 指定覆蓋率報告的輸出目錄。  
<br>
- 上述是 package.json 配置中的一些常見選項，根據具體需求，可以在加入其他配置。  


### ─ 新增測試檔案  

1. 假設在 src/ 路徑下，有一個 calculator.js 需要被測試的檔案。  

    - test_project/  
        - **src/**  
            - <span class="red">calculator.js</span>  
        - tests/  
<br>
2. 需要將 calculator.js 內部方法使用 module.exports 導出。  
    ```JavaScript
    // calculator.js
    const calculator = {
        function() {
            //...
        }
    };

    module.exports = calculator;
    ```
    - [(測試檔案如何使用被測試原始碼的內部方法)]({{ site.baseurl }}/testing/2024/03/22/Jest-Testing-Importing-and-Customizing-Modules.html#h--引入外部依賴)  
<br>
3. 再 tests/ 路徑下，新增名為 calculator.test.js 的測試檔案。  
    - test_project/  
        - src/  
            - calculator.js  
        - **tests/**  
            - <span class="red">calculator.test.js</span>  
<br>
4. 在 calculator.test.js 測試檔案檔案內，引入被測試檔案。  
    ```JavaScript
    // calculator.test.js
    const Calculator = require('calculator');
    ```

5. 針對被測試檔案內部方法撰寫測試。  
    ```JavaScript
    // calculator.test.js
    const Calculator = require('calculator');

    describe('testModule', () => {
      it('TestFunction', () => {
        //...
        });

      it('innerFunction', () => {
        //...
      });
    });
    ```
    - [(如何撰寫測試案例)](#-測試案例)  
    - [(如何驗證測試結果)](#-監視--斷言)  


### ─ 如何運行測試  

- 在 Command-line (CMD) 運行測試  
<br>
    - 打開（CMD）使用 cd 命令進入你的專案目錄。  
<br>
    - 如果在 package.json 中設置了測試腳本，一旦進入了專案目錄，可以直接運行測試。  
    
        ```bash
        cd path/to/your/project
        ```
    - 或是輸入指令進行測試，這將會運行所有的測試並輸出結果到終端。  
        ```bash
        npm test
        ```
        ```bash
        npm test -- --coverage        # --coverage 會顯示覆蓋率
        ```
    - 預設情況下，Jest 會找：**tests** 資料夾內的 .js, .jsx, .ts, .tsx 以 .test 或 .spec 結尾的檔案。  
        - 例如 component1.test.js  

---  

## 測試方法  

### ─ 測試案例  

- **describe**  
    - 組織測試案例：  
        - 使用 describe 函數來將測試案例分組，使其更有組織性和易讀性。  
    <br>
    - 提供描述標題：  
        - 每個 describe 區塊有一個描述性的標題，可以清楚地說明這組測試案例的目的或主題。  
    <br>
    - 隔離測試案例：  
        - 使用 describe 函數將相關的測試案例分組，這樣可以更好地隔離測試案例之間的影響。  
    <br>

- **it / test**  
    - 提供測試案例的描述：  
        - 通過標題清楚地描述測試的目的或要驗證的功能。    
    <br>
    - 提供可讀性和可維護性：  
        - 通過清晰的描述和組織良好的測試案例，提高測試的可讀性和可維護性。  
    <br>
    
```JavaScript
describe('testModule', () => {
    it('TestFunction', () => {
        //...
    });
    
    it('innerFunction', () => {
        //...
    });
});
```


### ─ HOOK  

- 在 Jest 測試框架中有提供鉤子函數，beforeEach & afterEach。  
    - beforeEach & afterEach 的觸發會**限制在 describe 群組內**。  
<br>
- **beforeEach**  
    - 函數會在每個測試運行**之前**運行一次。  
    - 通常用於設置測試的前置條件。  
    - 例如：初始化測試數據、創建模擬對象等  
    ```JavaScript
    describe('testModule', () => {
            beforeEach(() => {
                // 在每個測試運行之前執行的操作
                // 初始化測試數據、創建模擬對象等
            });
            
            it('TestFunction', () => {
                //...
            });
    });
    ```
<br>
- **afterEach**  
    - 函數會在每個測試運行**之後**運行一次。  
    - 通常用於清理測試過程中可能產生的副作用。  
    - 例如：重置測試數據、銷毀模擬對象等。  
    ```JavaScript
    describe('testModule', () => {
            afterEach(() => {
                // 在每個測試運行之後執行的操作
                // 清理測試過程中可能產生的副作用、重置測試數據、銷毀模擬對象等
            });
        
            it('TestFunction', () => {
                //...
            });
    });
    ```


### ─ 監視 & 斷言  

- 而該怎麼驗證測試是否正確，Jest 提供 jest.spyOn() 函式。  

    - jest.spyOn() 用於創建一個對特定物件的方法進行模擬的 Spy（間諜）。  

    - jest.spyOn() 也可以監視一個物件的特定方法，記錄它的呼叫情況以及接收到的參數。  
    
        ```JavaScript
        // Module.test.js
        // 引入被測試的模組
        const Module = require('./Module');

        // 使用 jest.spyOn 監視 TestFunction 方法
        const spy = jest.spyOn(Module, 'TestFunction');

        // 呼叫 TestFunction 方法
        Module.TestFunction();

        // 斷言 TestFunction 方法被呼叫過
        expect(spy).toHaveBeenCalled();

        // 清除監視，恢復原始方法
        spy.mockRestore();
        ```
<br>
- 在上述 jest.spyOn 監視物件方法後，可以使用 Jest 提供的 expect() 斷言函式來進行測試  
    - **返回值的斷言方法**
        1. expect().toBe(value): 斷言實際值是否與預期值完全相等。  
        2. expect().toEqual(value): 斷言兩個對象是否在值上相等（所有屬性和屬性值相等）。  
        3. expect().toMatch(pattern): 斷言字符串是否與正則表達式匹配。  
        4. expect().toContain(item): 斷言某個集合（數組、Set）是否包含指定的元素。  
        5. [more](https://jestjs.io/docs/expect)...  
<br>
    - **監視的斷言方法**
        1. toHaveBeenCalled(): 斷言被監視的方法是否被呼叫過。  
        2. toHaveBeenCalledWith(arg1, arg2, ...): 斷言被監視的方法是否被指定的參數呼叫過。  
        3. toHaveReturned(): 斷言被監視的方法是否有返回值。  
        4. toHaveReturnedWith(value): 斷言被監視的方法是否有指定的返回值。  
        5. toHaveBeenCalledTimes(number): 斷言監視方法被呼叫的次數是否符合指定的數量。  
        6. [more](https://jestjs.io/docs/expect)...  


### ─ 模擬方法行為  

- **mockReturnValue**  
    - mockReturnValue 用於設置 mock 函數的返回值。它將覆蓋原始函數的返回值，無論函數的實際執行結果是什麼，都將返回被設置的值。  
    - 通常與 jest.fn() 一起使用，用於創建一個新的 mock 函數並設置其返回值。  
    - 當你想要模擬函數的返回值，但不關心函數的實際實現時，可以使用 mockReturnValue。  
<br>
- **mockImplementation**  
    - mockImplementation 用於設置 mock 函數的實現。它將覆蓋原始函數的實現，無論原始函數的實際代碼是什麼，都將執行被設置的函數實現。  
    - 通常與 jest.fn() 一起使用，用於創建一個新的 mock 函數並設置其實現。  
    - 當你想要模擬函數的實現，並指定特定的行為或邏輯時，可以使用 mockImplementation。  
<br>
- 當測試某個方法時，該方法內部調用了其他方法並依賴其回傳值時，使用模擬回傳值的方法來測試。  
    - 隔離被測試方法的行為，專注於測試它的功能，而不需要依賴其他方法的實際實現。  
    - 可以使測試更加獨立和可靠，並且在測試失敗時更容易定位問題。  
<br>

    ```JavaScript
    // Module.js
    function TestFunction() {
      // 假設 TestFunction 內部調用了 innerFunction 方法
      const value = innerFunction();
      return value;
    }

    function innerFunction() {
      //...內部處理邏輯
    }

    module.exports = {
      TestFunction: TestFunction,
      innerFunction: innerFunction
    };
    ```
    
    ```JavaScript
    // Module.test.js
    const Module = require('./Module');

    // 模擬 innerFunction 的返回值
    const innerSpy = jest.spyOn(Module, 'innerFunction').mockReturnValue('jensen');

    // 使用 jest.spyOn 監視 TestFunction 方法
    const spy = jest.spyOn(Module, 'TestFunction');

    // 調用 TestFunction 方法
    const result = Module.TestFunction();

    // 斷言方法的返回值是否正確
    expect(result).toBe('jensen');

    // 清除監視
    spy.mockRestore();
    innerSpy.mockRestore();
    ```


### ─ 清除 & 恢復  
- 在測試結束後要清除監視，恢復原始方法，避免引響其他測試，確保測試獨立性。  

    1. jest.clearAllMocks(): 清除所有模擬函數的調用次數、傳入的參數等，但不還原模擬函數的行為。  
    2. jest.clearAllTimers(): 清除所有計時器的設置，包括 setTimeout、setInterval 等。  
    3. jest.resetAllMocks(): 重置所有模擬函數的狀態，包括還原它們的行為以及清除調用信息。  
    4. jest.resetModules(): 重置所有模組的狀態，將它們從快取中卸載並重新加載，使它們回到初始狀態。  
    5. jest.restoreAllMocks(): 還原所有使用 jest.spyOn 創建的模擬函數的原始實現。  
    6. jest.runOnlyPendingTimers(): 立即執行所有處於待定狀態的計時器，而不需要等待真實的時間。  
    7. jest.advanceTimersByTime(ms): 快進指定時間(ms)以觸發計時器。  

---

## 覆蓋率分析  

### ─ Command  
- 當執行測試之後在在 Terminal 畫面上會顯示目前被測程式，測試到的覆蓋率 %。  
![覆蓋率](https://hackmd.io/_uploads/B1RkOZOy0.png)  

### ─ HTML  
- 執行測試完畢的同時在 Coverage 資料夾下的 lcov-report 裡，也會生成一個 index.html。  
![index網頁](https://hackmd.io/_uploads/Hk9LUPZkR.png)  

- 裡面提供了更詳細的圖表分析報告。  
![總分析報告](https://hackmd.io/_uploads/Hy8w_bdy0.png)  

- 可以針對個別檔案點擊，查看詳細的內容。  
    - 內容會以<span class="red">**紅色**</span>標註指出未覆蓋到的 Statements。  
    
    - 而在判斷處會指出其中 **if** 或 **else** 沒有被測試到。  
    
    - 能夠更精確的撰寫測試案例，以便覆蓋到所有 Statements。  

    ![個別分析報告](https://hackmd.io/_uploads/rJ6kdvWkA.png)  

---

## Reference  
- [Jest Docs](https://jestjs.io/docs/getting-started)  
- [Jest Expect](https://jestjs.io/docs/expect)  
- [Jest Methods](https://jestjs.io/docs/api#methods)  
- [Jest Mock Functions](https://jestjs.io/docs/mock-functions)  



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
