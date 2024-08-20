# amis
### 規劃作業(待辦)
* 將 00000000 這個 DB 實際輸入我的實際工作內容，公司用 DE 代替，目的是透過真實資料，測試實際使用時的可能問題(優化)
  目前同時要先維護的 DB
  - a.eis_00000000 自用的 DB (刪除 00000000D 及 eis_00)
  - b.eis_xxx 汪哥公司用的 DB
  - c.eis_demo 開放給所有人的測試 DB
  - d.原生的 DB 架構
    - zenBusiness 分離 DB 架構的 ERP DB
    - zenWizard   分離 DB 架構的 SYS DB
* ~~改為本地主機, 行事曆的時數是否正常~~
* 讓 admin 可以直接改 json 程式(目前是唯讀)
```
  alter table sys_users add admin_user char(1);
  "disabledOn": "${ls:admin_user != 'Y'}",
```

### 持續完善 M-D
* ~~a.(ok)表頭修改存檔後沒有立刻 refresh 的 bug (更新到 2.4 就 ok)
    表頭無法修改客戶資料，但是業務資料可以，應該是傳回多欄位的問題~~
* ~~b.(ok)修改及查詢時，表身也應該要有合計顯示
    *** 參考 CHI 的寫法，在 Grid 下方新增 div 專門顯示金額、稅金及小計
    至於數量，可以仍放在 Grid 下方的加總列中~~
* ~~c.(ok)修改 detail 時，部分欄位(如品名)沒有顯示內容 isFalse
    --> 還存在很多問題, 難搞~~
* ~~d.(ok)新增 template 的編輯程式, 直接改 db 實在太 Low~~
* ~~e.(ok)修改時，新增一筆 detail，按 OK 後沒有顯示在 grid，但存檔會存起來
    應該 reload 可以解決, 但尚未找到正確的寫法 -> 還是得用 event~~
* f.修改表身後, 表頭合計未同步更新(DB 已更新), reload 後 domain data 未即時更新, 試了多個方法皆無效
    目前只能先新增一個 button 暫時解決
  
* 導入 Definitions, 降低 json 的代碼量 -> 在 zen_fields 測試失敗, 我是用 alert, 就報錯
```
  https://aisuda.bce.baidu.com/amis/zh-CN/docs/types/definitions
  {
  "definitions": {
    "aa": {
      "type": "input-text",
      "name": "jack",
      "value": "ref value",
      "labelRemark": "通?<code>\\$ref</code>引入的?件"
    }
  },
  "type": "page",
  "title": "引用",
  "body": [
    {
      "type": "form",
      "api": "api/xxx",
      "actions": [],
      "body": [
        {
          "$ref": "aa"
        }
      ]
    }
  ]
}
```
      
### 整理行銷方案
* (1).創建 fb 新社團(新速實簡的低代碼平台)
  - 新：創新－新產品、新發明或新思維，創新符合未來的需求
  - 速：快速反應、快速成果，優先投入必要資源，短期要有立竿見影的效果且有利於中長期的發展
  - 實：凡事講求實際，重視實用，不喜歡不切實際（如高談闊論）的東西
  - 簡：要求是既要實用又要好用，好用就是使用方法簡單，不要太複雜。 簡單，化繁為簡，包裝太多不符社會期待，要返璞歸真。
* (2).到外包網留資料(行銷文案)
* (3).整理可能合作的代理商名單或個人
* (4).整理舊客戶名單

### 利用 ChatGPT 將 ms-sql 的 sp 改寫為 mysql 版(測試可行，但不確定正確性)
* ~~1.先將 db 轉為 mysql 版 -> 用 DB2DB~~
* 2.逐步更新相關 sp、udf 及 trigger
* 3.全部重新測試 Wizard v2.0 及 amis api v2.0

### 個人管家(簡化為個人用版本)
原本是想要弄一個個人用的版本，但優先序不高，對推廣沒有直接的助益，先暫停

# ERPNext
### 開源商盟的2大支柱產品
* ERPNext(Frappe) 提供中小企業的 ERP 最優的整體擁有成本 Total Cost of Ownership (TCO)解決方案
* 智能總管(amis low code) 提供中小企業的非 ERP 最優的整體擁有成本 Total Cost of Ownership (TCO)解決方案

# 行銷相關
### 設定一個小目標
* 1.EN 推廣
  - a.三年內 100 家企業用戶
  - b.完善正體中文化
  - c.銷售促成物 
    - @使用手冊
    - @報表範例
    - @成功案例
  - d.技術手冊
  - e.進階開發
* 2.推廣方法
  - a.人脈：汪哥(Chi相關)、Jack、宣揚和訊光的舊同事
  - b.公司：卓爾、千奧(類似的這種撐著的公司)、正航的經銷商(和汪哥討論下)
* 3.先讓用戶多起來，賺日後維運的錢
* 4.極簡化的商業模式：
  - a.提供客戶價值: 不是以自己為出發點想像的客戶需求，而是真的勾出客戶真正的需求
  - b.利用社群媒體，用上一點的原則，勾出客戶的需求，重點是要想辦法【讓客戶找上門】，這才是優質客戶的來源
  - c.續上，建立個人、公司或品牌價值，目的都是【獲客】
  - d.可複製的商業模式，盡量不要把自己的時間綁死
  - e.盡可能自動化的被動收入模式
  - f.多寫一些分享的文章(建立顧問形象) > 台灣 EN 第一人
  
### 可以怎麼做?
* 1.出書(7/16) -> 7/25(moo 審核離譜的久)
* 2.入門方案：安裝+1天的輔導=1W
* 3.專業方案：服務包 Service Package=購買點數，每10點1W(問題排除)
* 4.企業方案：包輔導上線(依個案)
  
# Commercial Opensource Software Alliance (COSA) 
* 柯薩：商用開源軟體聯盟，原本隨便想的名字，竟然已經存在多年
* [中華民國開放系統協會](https://www.cosa.org.tw/)
* 加入協會，借助協會的力量推廣，這個是否有可行性 ?
* 盡量讓更多的中間商加入，擴大市場，但是這個真不容易
  - 能接受以服務為主要收入

### 多花心思將企業會經常使用的 Open Source 專案整合打包為一個解決方案(加上我自己的 amis 解決方案)
調整為全 Open Source + 僅收取培訓費用及顧問輔導費, 重點轉為立言, 順帶賺點生活費吧
* a.系統
  - (1)amis APP (最好也能提供 Linux Solution)
  - (2)Redash or metabase (BI)
  - (3)ReportServer (Query Report)
  - (4)Open Project (PMS) 或是 Redmind
  - (5)Odoo ERP (or ERPNext)
* b.教材
  - (1)使用手冊整理
  - (2)教學說明影片
  - (3)服務
