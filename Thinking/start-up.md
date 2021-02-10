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
* 利用工具(或Wizard) 協助客戶將任何一套現有系統的資料轉為報表
  * 乍聽和 BI Tools 很像，主要差別是學習 Canva 的極簡做法，讓設計報表簡化到極簡(或是要針對無Web系統的ERP，如 chi等，提供基礎及客制方案)
    * 這個聽起來不容易，因為要寫報表必須先了解 Table Schema，這個問題應該無法簡化(或是要針對性的研究特定系統)

#### 行業別的雲端解決方案 
* 社區管理系統
  * 雲管家 http://www.56168.com.tw/product.html
  
#### 寫書搭配工具推廣
* 擁抱 Open Source, MIT, 不必擔心沒有收入
  * [免費的開源軟體到底怎麼賺錢？](https://www.juduo.cc/tech/458802.html)
  * [開源軟體的商業模式分析 (二)](https://www.bnext.com.tw/article/52848/open-source--business-model-2)
  * [為什麼「開源」是新世界的創業方式？](http://goodwork.hfcc.com.tw/356132611838678303402603221109269893693921205/why-startup-open-source)
* Windows ERP(gex) 如何開創市場? ==> 證明不是空口說白話，也一併樹立標竿
  * 免費下載使用([官網](http://www.gex.com.tw) 要重新整理並上線)，附安裝說明，手冊另行購買。
  * 收費方式：
    * 使用手冊=?99(電子書)
    * 維護合約(年)
    * 計次服務(上課)
* Web 版 ERP 推廣方式? 免費雖然好用，但也不是萬靈丹，再仔細想一下
* 賣書搭配工具
  * 買 [玩轉SQL] 加購+?99 送 ORM程式
  * 買 [ERP開發勝經] 加購+??99 送 Wizard 程式(或 WinERP ...)
* 加購工具類(工具若要服務或說明，請購買顧問時數)
  * ORM
  * Security(Permissions) 標準機制含 Tables
  * Wizard
  * Node.js 後台
  * 薪資計算Tools
  * WorkFlow
  * SAP 報表產生器(含 Delphi7 Source)
  * RWD 網站模板
  * 官網模版(已改為 RWD) ==> 想辦法改成依資料庫設定顯示，並準備多些大小圖方便設定。
  * Wizard 前端網頁產生器(Thinking)
  * 其他繼續思考中(可以多寫一些小工具或程式) ==> 可以試著由ERP系統拆出小模組(Thinking)
* 付款方式(非書籍類)
  * 加入[蝦皮賣家](https://agirls.aotter.net/post/54780)，利用電商平台處理金流及發票事宜(個人賣家)

#### 參考好文
* [How side projects saved our startup(thenextweb.com)](https://thenextweb.com/entrepreneur/2014/10/23/side-projects-saved-our-startup/1/)
