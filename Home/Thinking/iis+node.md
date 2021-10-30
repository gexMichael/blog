
終於測試成功將 node.js + IIS
基本上網上的教學是 ok 的，目前在 GCP 已測試成功
之前一直失敗是因為正確的呼叫方式要含 port 5050

http://api.gex.com.tw:5050
在 IIS 的網站設定, 仍必須設為 80
然後在 api 的目錄也必須有 web.config, 內容主要是指定 xxx.js(如 index.js)

先安裝 iisnode 及 rewrite
其他就和一般設定相同
