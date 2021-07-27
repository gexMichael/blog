## SQL Server 2008中Service Broker基礎應用
* 導讀：本文主要涉及Service Broker的基本概念及建立一個Service Broker應用程式的基本步驟。
* 來源: http://www.cnblogs.com/downmoon/archive/2011/04/05/2005900.html

一、前言：
Service Broker為SQL Server提供訊息佇列，這提供了從資料庫中發送非同步異動訊息佇列的方法。Service Broker消息可以保證以適當的順序或原始的發送順序不重複地一次性接收。並且因為內建在SQL Server中，這些消息在資料庫發生故障時是可以恢復的，也可以隨資料庫一起備份。在SQL Server 2008中，還引入了使用Create Broker Priority命令對會話設定優先順序，可以對重要的或不重要的會話進行優先順序設定，以保證消息合理地處理。
本文假定一個線上資料庫BookStore中存儲了一些業務訂單。我們使用Service Broker應用程式將消息發送到另一個資料庫BookDistribution，該資料庫是分離的應用程式調用，該應用程式控制倉庫入庫和出庫交付， 並返回消息給BookStore。
創建Service Broker應用程式大體步驟如下：
1、定義希望應用程式執行的非同步任務。
2、確定Service Broker的發起方服務和目標服務是否創建在同一個SQL Server實例中。如果是兩個實例，實例間的通信還需要創建經過證書認證或NT安全的身份認證，並且要創建端點、路由以及對話安全模式。
3、如果沒有啟用，則在多方參與的資料庫中使用Alter Database命令設置Enable_broker以及Truseworthy資料庫選項。
4、為所有多方參與的資料庫創建資料庫主金鑰。
5、創建希望在服務之間發送的訊息類型。
6、創建契約（Contract）來定義可以由發起方發送的各種消息以及由目標發送的訊息類型的種類。
7、同時在兩方參與的資料庫中創建用於保存消息的佇列。
8、同時在綁定特定約定到特定佇列的多方參與的資料庫中創建服務。
二、實例
下面我們通過一個示例來實現以上步驟：
（一）、啟用資料庫的Service Broker活動
-- Enabling Databases for Service Broker Activity
 

USE master
GO
IF NOT EXISTS (SELECT name FROM sys.databases WHERE name = 'BookStore')
CREATE DATABASE BookStore
GO
IF NOT EXISTS (SELECT name FROM sys.databases WHERE name = 'BookDistribution')
CREATE DATABASE BookDistribution
GO
ALTER DATABASE BookStore SET ENABLE_BROKER
GO
ALTER DATABASE BookStore SET TRUSTWORTHY ON
GO
ALTER DATABASE BookDistribution SET ENABLE_BROKER
GO
ALTER DATABASE BookDistribution SET TRUSTWORTHY ON



（二）、創建資料庫主金鑰
-- Creating the DatabaseMaster Key for Encryption
 

USE BookStore
GO
CREATE MASTER KEY
ENCRYPTION BY PASSWORD = 'I5Q7w1d3'
GO
 

USE BookDistribution
GO
CREATE MASTER KEY
ENCRYPTION BY PASSWORD = 'D1J3q5z8X6y4'
GO



（三）、管理訊息類型
使用CREATE MESSAGE TYPE（HTTP://msdn.microsoft.com/en-us/library/ms187744.aspx）命令,
-- Managing Message Types
 

Use BookStore
GO
-- 發送圖書訂單的訊息類型
CREATE MESSAGE TYPE [//SackConsulting/SendBookOrder]
VALIDATION = WELL_FORMED_XML
GO
 

--目標資料庫發送的訊息類型
CREATE MESSAGE TYPE [//SackConsulting/BookOrderReceived]
VALIDATION = WELL_FORMED_XML
GO
 

--執行同樣的定義
Use BookDistribution
GO
-- 發送圖書訂單的訊息類型
CREATE MESSAGE TYPE [//SackConsulting/SendBookOrder]
VALIDATION = WELL_FORMED_XML
GO
 

--目標資料庫發送的訊息類型
CREATE MESSAGE TYPE [//SackConsulting/BookOrderReceived]
VALIDATION = WELL_FORMED_XML
GO



--注意，此處沒有定義消息的內容。實際的消息是訊息類型的實例。
（四）、創建契約（Contract）
使用Create Contract（HTTP://msdn.microsoft.com/en-us/library/ms178528.aspx）
-- Creating Contracts
 

Use BookStore
GO
CREATE CONTRACT
[//SackConsulting/BookOrderContract]
( [//SackConsulting/SendBookOrder]
SENT BY INITIATOR,
[//SackConsulting/BookOrderReceived]
SENT BY TARGET
)
GO
 

USE BookDistribution
GO
CREATE CONTRACT
[//SackConsulting/BookOrderContract]
( [//SackConsulting/SendBookOrder]
SENT BY INITIATOR,
[//SackConsulting/BookOrderReceived]
SENT BY TARGET
)
GO



--發起方和目標的定義必須相同
（五）、創建佇列
佇列用來保存資料。使用命令Create queue（HTTP://msdn.microsoft.com/en-us/library/ms190495.aspx）
-- Creating Queues
 

Use BookStore
GO
--保存BookDistribution過來的消息
CREATE QUEUE BookStoreQueue
WITH STATUS=ON
GO
 

USE BookDistribution
GO
--保存BookStore過來的消息
CREATE QUEUE BookDistributionQueue
WITH STATUS=ON
GO



（六）、創建服務
服務定義端點，然後使用它來將訊息佇列綁定到一個或多個契約上。服務使用佇列和契約來定義一個或一組任務。有點拗口，是不是？
服務是消息的發起方和接收方強制約定的規則，並將消息路由到正確的序列。
使用Create Service（HTTP://msdn.microsoft.com/en-us/library/ms190332.aspx）命令。
-- Creating Services
 

Use BookStore
GO
CREATE SERVICE [//SackConsulting/BookOrderService]
ON QUEUE dbo.BookStoreQueue--指定的佇列綁定到契約
([//SackConsulting/BookOrderContract])
GO
 

USE BookDistribution
GO
CREATE SERVICE [//SackConsulting/BookDistributionService]
ON QUEUE dbo.BookDistributionQueue--指定的佇列綁定到契約
([//SackConsulting/BookOrderContract])
GO
 

（七）、啟動對話
對話會話（dialog conservation）是在服務之間進行消息交換的操作。
使用Begin Dialog Conversation（HTTP://msdn.microsoft.com/en-us/library/ms187377.aspx） 命令創建新的會話。使用Send（HTTP://msdn.microsoft.com/en-us/library/ms188407.aspx）來發送消息。使用End Conversation命令（HTTP://msdn.microsoft.com/en-us/library/ms177521.aspx）結束會話。
-- Initiating a Dialog
 

Use BookStore
GO
 

--保存會話控制碼和訂單資訊
DECLARE @Conv_Handler uniqueidentifier
DECLARE @OrderMsg xml;
 

BEGIN DIALOG CONVERSATION @Conv_Handler--創建會話
FROM SERVICE [//SackConsulting/BookOrderService]
TO SERVICE '//SackConsulting/BookDistributionService'
ON CONTRACT [//SackConsulting/BookOrderContract];
SET @OrderMsg =
'<order id="3439" customer="22" orderdate="2/15/2011">
<LineItem ItemNumber="1" ISBN="1-59059-592-0" Quantity="1" />
</order>';
SEND ON CONVERSATION @Conv_Handler--發送到BookDistribution資料庫的佇列中
MESSAGE TYPE [//SackConsulting/SendBookOrder]
(@OrderMsg);



（八）、查詢佇列中傳入的消息
-- Querying the Queue for IncomingMessages
 

USE BookDistribution
GO
SELECT message_type_name, CAST(message_body as xml) message,
queuing_order, conversation_handle, conversation_group_id
FROM dbo.BookDistributionQueue



查詢結果：
09262M051-0  

（九）、檢索並回應訊息
使用Receive語句（HTTP://msdn.microsoft.com/en-us/library/ms186963.aspx）從佇列中讀取行（消息），也可以刪除已經讀取的消息。Receive的結果可以填充到常規表中，也可以在區域變數中執行其他操作，或發送到其他service Broker消息。如果消息是XML資料類型的消息，則可以直接借助TSQL的XQuery來操作。
-- Receiving and Responding to aMessage
 

USE BookDistribution
GO
--創建一個表存放接收到的訂單資訊
CREATE TABLE dbo.BookOrderReceived
(BookOrderReceivedID int IDENTITY (1,1) NOT Null,
conversation_handle uniqueidentifier NOT Null,
conversation_group_id uniqueidentifier NOT Null,
message_body xml NOT Null)
GO

-- 聲明變數
DECLARE @Conv_Handler uniqueidentifier
DECLARE @Conv_Group uniqueidentifier
DECLARE @OrderMsg xml
DECLARE @TextResponseMsg Varchar(8000)
DECLARE @ResponseMsg xml
DECLARE @OrderID int;

--從佇列中獲取消息，將接收值賦于區域變數
RECEIVE TOP(1) @OrderMsg = message_body,--TOP指定最多一條消息
@Conv_Handler = conversation_handle,
@Conv_Group = conversation_group_id
FROM dbo.BookDistributionQueue;

-- 將變數值插入表中
INSERT dbo.BookOrderReceived
(conversation_handle, conversation_group_id, message_body)
VALUES
(@Conv_Handler,@Conv_Group, @OrderMsg )

-- 使用XQuery進行抽取以回應訊息訂單
SELECT @OrderID = @OrderMsg.value('(/order/@id)[1]', 'int' )
SELECT @TextResponseMsg =
'<orderreceived id= "' +
CAST(@OrderID as Varchar(10)) +
'"/>';
SELECT @ResponseMsg = CAST(@TextResponseMsg as xml);

-- 使用既有的會話控制碼，發送回應訊息到發起方
SEND ON CONVERSATION @Conv_Handler
MESSAGE TYPE [//SackConsulting/BookOrderReceived]

（十）、結束會話
-- Ending a Conversation

USE BookStore
GO
-- 創建訂單確認表
CREATE TABLE dbo.BookOrderConfirmation
(BookOrderConfirmationID int IDENTITY (1,1) NOT Null,
conversation_handle uniqueidentifier NOT Null,
DateReceived datetime NOT Null DEFAULT GETDATE(),
message_body xml NOT Null)

DECLARE @Conv_Handler uniqueidentifier
DECLARE @Conv_Group uniqueidentifier
DECLARE @OrderMsg xml
DECLARE @TextResponseMsg Varchar(8000);

RECEIVE TOP(1) @Conv_Handler = conversation_handle,
@OrderMsg = message_body
FROM dbo.BookStoreQueue

INSERT dbo.BookOrderConfirmation
(conversation_handle, message_body)
VALUES (@Conv_Handler,@OrderMsg );

END CONVERSATION @Conv_Handler;
GO

USE BookDistribution
GO
DECLARE @Conv_Handler uniqueidentifier
DECLARE @Conv_Group uniqueidentifier
DECLARE @OrderMsg xml
DECLARE @message_type_name Nvarchar(256);

RECEIVE TOP(1) @Conv_Handler = conversation_handle,
@OrderMsg = message_body,
@message_type_name = message_type_name
FROM dbo.BookDistributionQueue

-- 雙方必須都結束會話
IF
@message_type_name = 'HTTP://schemas.microsoft.com/SQL/ServiceBroker/EndDialog'
BEGIN
END CONVERSATION @Conv_Handler;
END


--查詢會話狀態
SELECT state_desc, conversation_handle
FROM sys.conversation_endpoints
09262H0T-1  

三、小結
本文通過一個實例演示了一個用來發送圖書訂單消息分發控制資料庫的簡單的消息交換應用程式。發起方發送圖書訂單，發回一個回應，並在兩個資料庫上使用END Conservation結束會話。現實場景中可以轉換為其他訊息類型、契約、服務和佇列。合理運用Service Broker應用程式的非同步特性可以防止因應用程式掛起而導致業務系統產生瓶頸。
本文參考：
1、SQL Server 2005 Service Broker 初探
HTTP://msdn.microsoft.com/zh-cn/library/ms345108%28v=sql.90%29.aspx
2、SQL Server 2008 Transact-SQL Recipes: A Problem-Solution Approach
HTTP://www.amazon.com/Server-2008-Transact-SQL-Recipes-Problem-Solution/dp/1590599802
 
 
本文繼續介紹Service Broker的設置會話優先順序，預存程序中實現。
一、Service Broker的設置會話優先順序
自SQL Server 2008起，對非常活躍的Service Broker應用程式，提供了設定優先權的命令CREATE BROKER PRIORITY（HTTP://msdn.microsoft.com/en-us/library/bb934170.aspx）。通過該命令，可以設置從1至10共10個等級的顆細微性來調試會話的優先順序，預設為5。在此之前，你必須得首先打開HONOR_BROKER_PRIORITY開關。
-- 設置會話優先順序

--啟用會話優先順序選項
ALTER DATABASE BookStore
SET HONOR_BROKER_PRIORITY ON

--啟用會話優先順序選項
ALTER DATABASE BOOKDistribution
SET HONOR_BROKER_PRIORITY ON

--查看設置結果
SELECT name, is_honor_broker_priority_on
FROM sys.databases
WHERE name IN ('BookStore', 'BookDistribution')

/*
name is_honor_broker_priority_on
BookStore 1
BookDistribution 1
*/

USE BookStore
GO
CREATE BROKER PRIORITY Conv_Priority_BookOrderContract_BookOrderService
FOR CONVERSATION
SET (CONTRACT_NAME = [//SackConsulting/BookOrderContract],--特定的契約
LOCAL_SERVICE_NAME = [//SackConsulting/BookOrderService],--本機服務
REMOTE_SERVICE_NAME = ANY,--遠端服務為ANY,即Service Broker端點的任何相關服務
PRIORITY_LEVEL = 10)--設定優先權為10
通過sys.conversation_priorities目錄檢視,查詢優先順序
SELECT name, priority, service_contract_id,
local_service_id,remote_service_name
FROM sys.conversation_priorities cp

/*
name priority service_contract_id local_service_id remote_service_name
Conv_Priority_BookOrderContract_BookOrderService 10 65536 65536 Null
*/
如果你希望包含服務和契約名稱，可以將服務和從sys.conversation_priorities（HTTP://msdn.microsoft.com/zh-cn/library/bb895280%28v=sql.100%29.aspx）返回的契約ID與sys.service_contracts（HTTP://msdn.microsoft.com/en-us/library/ms184378.aspx），sys.services（HTTP://msdn.microsoft.com/en-us/library/ms174429.aspx）目錄檢視關聯起來。
USE BookDistribution
GO
--創建目標服務的優先順序，與發起方的優先順序保持一致,
--以使會話的優先順序設置覆蓋整個會話的生命週期

CREATE BROKER PRIORITY Conv_Priority_BookOrderContract_BookDistributionService
FOR CONVERSATION
SET (CONTRACT_NAME = [//SackConsulting/BookOrderContract],
LOCAL_SERVICE_NAME = [//SackConsulting/BookDistributionService],
REMOTE_SERVICE_NAME = ANY,
PRIORITY_LEVEL = 10)

USE BookStore
GO
ALTER BROKER PRIORITY Conv_Priority_BookOrderContract_BookOrderService
FOR CONVERSATION
SET (REMOTE_SERVICE_NAME = '//SackConsulting/BookDistributionService')
--修改遠端服務名稱

ALTER BROKER PRIORITY Conv_Priority_BookOrderContract_BookOrderService
FOR CONVERSATION
SET (PRIORITY_LEVEL = 9)
--設定優先權

--刪除優先順序設置
DROP BROKER PRIORITY Conv_Priority_BookOrderContract_BookOrderService
二、Service Broker的預存程序實現
在上文中，我們使用的臨時T-SQL來演示Service broker的步驟，事實上， 我們完全可以通過預存程序或外部應用程式自動啟動並處理佇列中的消息。使用Create Queue（HTTP://msdn.microsoft.com/en-us/library/ms190495.aspx）和Alter Queue（HTTP://msdn.microsoft.com/en-us/library/ms189529.aspx）選項，也可以指定可以啟動並處理在同一佇列中傳入的消息的、同時執行的相同服務程式的數量。
繼續上文的示例：
-- Creating the Bookstore Stored Procedure

USE BookDistribution
GO
CREATE PROCEDURE dbo.usp_SB_ReceiveOrders
AS
DECLARE @Conv_Handler uniqueidentifier
DECLARE @Conv_Group uniqueidentifier
DECLARE @OrderMsg xml
DECLARE @TextResponseMsg Varchar(8000)
DECLARE @ResponseMsg xml
DECLARE @Message_Type_Name Nvarchar(256);
DECLARE @OrderID int;

-- XACT_ABORT automatically rolls back the transaction when a runtime error occurs
SET XACT_ABORT ON

BEGIN TRAN;

RECEIVE TOP(1) @OrderMsg = message_body,
@Conv_Handler = conversation_handle,
@Conv_Group = conversation_group_id,
@Message_Type_Name = message_type_name
FROM dbo.BookDistributionQueue;

IF @Message_Type_Name = '//SackConsulting/SendBookOrder'
BEGIN
INSERT dbo.BookOrderReceived
(conversation_handle, conversation_group_id, message_body)
VALUES
(@Conv_Handler,@Conv_Group, @OrderMsg )
SELECT @OrderID = @OrderMsg.value('(/order/@id)[1]', 'int' )
SELECT @TextResponseMsg =
'<orderreceived id= "' +
CAST(@OrderID as Varchar(10)) +
'"/>';
SELECT @ResponseMsg = CAST(@TextResponseMsg as xml);
SEND ON CONVERSATION @Conv_Handler
MESSAGE TYPE [//SackConsulting/BookOrderReceived]
(@ResponseMsg );
END

IF @Message_Type_Name = 'HTTP://schemas.microsoft.com/SQL/ServiceBroker/EndDialog'
BEGIN
END CONVERSATION @Conv_Handler;
END
COMMIT TRAN
GO
解析：該預存程序包含處理//SackConsulting/SendBookOrder和HTTP://schemas.microsoft.com/SQL/ServiceBroker/EndDialog訊息類型的邏輯。如果發送發後者，特定會話的控制碼的特定會話會結束。如果接收到圖書訂單訊息類型，它的消息將插入到表中，並且返回訂單確認資訊。
可以使用Alter Queue命令修改既有的佇列。這個命令使用與Create Queue相同的選項，它允許改變佇列的狀態與保持期、待啟動的預存程序、佇列讀取預存程序實例的最大數量以及過程的安全模式契約。
Alter Queue包括一個額外的參數Drop，它用來刪除佇列的所有預存程序啟動設置。
使用Alter Queue命令將預存程序綁定到既有的佇列：
----使用Alter Queue命令將預存程序綁定到既有的佇列
ALTER QUEUE dbo.BookDistributionQueue
WITH ACTI加值稅ION (STATUS = ON,
PROCEDURE_NAME = dbo.usp_SB_ReceiveOrders,
MAX_QUEUE_READERS = 2,--獨立處理佇列中不同消息的同一預存程序同時執行的最大數量
EXECUTE AS SELF)--即預存程序將以與執行Alter Queue命令的主體的相同的許可權來執行
為了測試BookStore資料庫的新服務程式，開始一個會話並設置新順序：
Use BookStore
GO

DECLARE @Conv_Handler uniqueidentifier
DECLARE @OrderMsg xml;

BEGIN DIALOG CONVERSATION @conv_handler
FROM SERVICE [//SackConsulting/BookOrderService]
TO SERVICE '//SackConsulting/BookDistributionService'
ON CONTRACT [//SackConsulting/BookOrderContract];

SET @OrderMsg =
'<order id="3490" customer="29" orderdate="7/22/2008">
<LineItem ItemNumber="1" ISBN="1-59059-592-0" Quantity="2" />
</order>';

SEND ON CONVERSATION @Conv_Handler
MESSAGE TYPE [//SackConsulting/SendBookOrder]
(@OrderMsg);
當佇列Status＝ON並且佇列中到到達新消息時，執行預存程序來處理傳入的消息。可以使用預存程序或外部程式，但使用預存程序的好處是，它們提供了處理消息、自動執行所有需要的回應和相關業務任務的簡單的封裝好的元件。
如果在目標佇列上有預存程序被執行，並且啟動新的接收到的消息，那麼應該已經有訂單確認訊息返回到dbo.BookStoreQueue:
SELECT conversation_handle, CAST(message_body as xml) message
FROM dbo.BookStoreQueue
/*
conversation_handle message
A7B7FA73-5B5F-E011-8B4E-001C23FA56DD <orderreceived id="3439" />
*/
小結：本文主要介紹Service Broker的設置會話優先順序，預存程序中實現。下文將介紹Service broker的遠端實現。
 
