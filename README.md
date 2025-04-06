# Gemini API

![Vite](https://badges.aleen42.com/src/vitejs.svg) ![Vite](https://badges.aleen42.com/src/vue.svg)

 串接 Google AI SDK for JavaScript API ，與 Google AI 模型進行對談

## 1.取得 API 金鑰

1. 前往 [Google AI Studio](<https://aistudio.google.com/>)
2. 登入 Google 帳號
3. 建立 [API 金鑰](<https://aistudio.google.com/app/apikey>)

## 2.設定開發環境

1. 建議將 Node.js 切換至 v20.9.0，其他版本可能無法執行

   ```code
   nvm install 22.14.0
   ```

   ```code
   nvm use 22.14.0
   ```

## 3.啟動專案

1. 安裝專案原始碼安裝至指定位置，你可以採取下列其中一種方法
      1. 點選右上方綠色『Code』按鈕，選擇『Download ZIP』下載ZIP檔，並解壓縮至指定資料夾中

      2. 開啟『終端機 Terminal.app』  
       輸入指令，移動到指定資料夾位置  

      ```code
      cd 「專案資料夾路徑」 (資料夾名稱間的空格要用「 / 」隔開)
      ```

      輸入指令將後端專案 clone至指定資料夾

      ```code
      git clone https://github.com/Pudding1989/Gemini-API-Sample.git
      ```

2. 安裝專案使用的套件

   ```code
    npm install
    ```

3. 輸入執行指令，啟動專案

```text
  npm run dev
```

- 當終端機顯示下列訊息，表示已成功開啟應用程式伺服器

  ```text
  VITE v6.2.x ready in xxx ms:
    -> Local:   http://localhost:5173/
    -> Network: http://192.168.0.1:5173/
    -> press h + enter to show help
  ```

## 4.開發工具

開發環境

- Node.js v22.14.0
- Vite: 6.2.1

前端框架

- Vue: 3.5.13

  - Vue Router: 4.5.0
  - Pinia: 3.0.1

前端套件

- Sass: 1.86
- Tailwind CSS: 4.0.17
