# Action Pack 

由三個模組組成

* ActionDispatch

負責Route，將request轉給適當的controller 

* ActionController

controller，負責處理request，產生正確的response

* ActionView

View，幫助controller產生適當的回應

# 分派request

路由規則：可以透過資源檔定義，或是設定pattern

Rails會處理url encoding

## REST

server與client之間透過stateless的通訊協定溝通，所有狀態透過request及response的編碼來傳遞

client透過一組預先定義好的資源識別碼（URL）來存取伺服器上的資源

常見的REST是透過http動詞來表達CRUD指令

* GET

* POST

* PUT

* PATCH

* DELETE

目標之一，讓展示層與資料層解耦合

## 處理request

符合route規則的controller收到需求之後，會從public method找符合的action

如果找到，會invoke該method

如果沒找到，會invoke controller.method_missing()

如果controller沒有實作method_missing()，會把request傳給下一個符合route的controller

直到所有controller都不符合，就丟出AbstractController::ActionNotFound例外

### 回應

* 套用template產出html，回應給browser（view）

* 不透過template，直接回應純文字

* 完全不回應（ajax的狀況）

* 或是回應html以外的媒體資源（串流或檔案）

template習慣上會放在views/controllers/路徑底下，檔名與action同名，副檔名則是request的類型，例如html、xml等...

render()

send_data()

### redirect

告訴client端你的需求不歸我管，去找別人，redirect回應內會告訴client端該找誰。

redirect_to()


## Object and Operation that span Requests

### session

類似hash table的結構，跨request。與cookies的差異是session適合用來保存web application的狀態

#### 放在哪裡？

* rails預設是比照cookies放到client端

* 或是放在server端，需要較多設定。

1. rails記錄session內容，算出一組session ID

2. rails把session的內容放在一個資料結構中，用session ID做索引。當request進來，rails用session ID檢查是否有對應的session

#### 可以放什麼？

* 只要能夠serialize的物件都可以放進去（透過Marshal()）

* 如果要放model，必須加上model 定義

* 不要放太多東西到session（效能考量）

* session適合放與使用者相關的資訊，至於全域資訊例如庫存總量，如果放在tom的session中，當david更新的時候，tom的session不會更新

* 不適合放重要資訊，因為session可能過期，cookies可能被刪除

每次request進來，應用程式都會去讀取session（如果有存的話），並且試著把session裡面存的物件還原回來。假若此時程式有更新，有可能產生session的內容與新的程式邏輯不相容。解決方案：

1. 物件內容放在資料庫裡面，連同session ID一起，每次request進來就去資料庫用session ID找對應的物件

2. 更新程式的時候直接清掉server上的session

3. 在session裡面加上版本資訊，當程式發現拿到的session版本與目前程式版本不一致，就丟掉session重新產生

session_store()

怎麼存完全看需求

#### session期限

session會占資源，因此最好設定一個期限，超過就丟掉

### flash

暫存用，在request之間如果有一些暫時性，偶發性的資料。懶得為它建model，可以用flash來傳遞，flash的內容只會保留在當下的request。當下一個request進來，前面的內容就會被清除掉

適合放例如錯誤訊息、提示訊息之類的

經驗是不應該濫用，會造成維護上的困難

### Callback

放在action執行前後，例如驗證、Log等等....

rails支援三種：before、after、arround
