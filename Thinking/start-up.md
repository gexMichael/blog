## 紀錄一些目前看起來是異想天開的想法
#### 資料庫虛擬化技術
* 建立一個通用GateWay, 讓現有的程式可以任意切換不同的 Database

#### AI 開發應用系統
* 用口語或規範化的文字定義，由AI根據後台已有的模板，快速產生可執行程式

#### Tools Coding by store procedure 
* 將 orm 架構(sp) open source(ms-sql 或 mariadb 版)，賣書或 Donate 皆可
* 通用後端 API 平台(以 Store Procedure 為核心基礎)
* App CodeGen wizard(思考中) ==> 將原有的 Wizard 繼續延續生命
* 續上，能否參考 code Smith 的萬用模板定義程式，將 Wizard 改為真正的萬用，不必不斷的配合新的工具就不斷的改寫 Wizard 程式(或 store procedure)
  * 沿用 Template 架構
  * 想辦法將 @var@ 變數能夠透過自定義，能任意產生替代的變數
  * https://www.cnblogs.com/linxin/p/6509897.html
  繼續完成林鑫之前的樣板 wizard 化，要想辦法做到不寫 Code，只靠設定檔(可能要新增很多設定檔)，就能搞定絕大多數的 Wizard 程式

#### 三位一體的整合方案
* app 為尖兵，通用資料存取為基礎，配合後端平台為驅動。以自行開發的 Wizard為技術核心，不處理複雜邏輯，但提供多個系統為範本，以移動整合平台為中心思想，提供快速解決方案。

#### 行業別的雲端解決方案 
* 社區管理系統
  * 雲管家 http://www.56168.com.tw/product.html
