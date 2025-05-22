
# M05-Unit 6 - 使用 Azure Portal 建立高可用性的 Front Door 架構

## 一、實驗模擬

在本練習中，你將設定 Azure Front Door，串接兩個部署於不同 Azure 區域的 Web App 實例。該架構會根據延遲將流量導向最近的站點，並具備自動容錯切換機制。

Contoso 公司希望能建立一個部署於多區域的 Web 應用，並透過 Azure Front Door 提供自動流量導向與容錯能力（Failover）。

![架構圖](../image/m5u6/task1_4.png)

## 二、架構目標

**本實驗將：** 

**1. 任務 1：建立兩個 Web App 實例** 建立兩個部署在不同地區的 Web App 實例（Active/Active 模式）

**2. 任務 2：登入 Azure Portal** 建立 Azure Front Door，根據延遲進行導流

**3. 任務 3：** 測試自動容錯切換機制

- 約 30 分鐘
---

## 三、任務流程

### (一) Task1: 建立兩個 Web App 實例

此練習需建立兩個 Web App 實例，分別部署於不同 Azure 區域，並以 Active/Active 模式運行。

1. 登入 Azure Portal  
   前往 https://portal.azure.com 登入

2. 建立第一個 Web App  
   (1) 在首頁搜尋 WebApp 並點選 App Services  
   (2) 點選「+ Create」建立 Web App    
   (3) 設定如下：  

   | 設定項目       | 值                              |
   |----------------|--------------------------------|
   | 訂閱           | 選擇你的訂閱                    |
   | 資源群組       | ContosoResourceGroup            |
   | 名稱           | WebAppContoso-1                 |
   | 發佈方式       | Code                            |
   | 執行堆疊       | .NET 8 (LTS)                    |
   | 作業系統       | Windows                         |
   | 區域           | Central US                      |
   | App Service Plan | myAppServicePlanCentralUS      |
   | 價格方案       | Standard S1（100 ACU，1.75 GB）|

   (4) 點選「Review + create」並建立

   ![Task1圖](az-700/vnet/image/m5u6\task1_2.png)
   ![Task1圖](az-700/vnet/image/m5u6\task1_3.png)

3. 建立第二個 Web App  
   (1) 重複上述步驟，改以下設定：

   | 設定項目       | 值                            |
   |----------------|-------------------------------|
   | 名稱           | WebAppContoso-2               |
   | 區域           | East US                       |
   | App Service Plan | myAppServicePlanEastUS      |

    ![Task1圖](az-700/vnet/image/m5u6\task1_4.png)

    (2) 備註：若出現部署錯誤，請確認是否為區域配額限制，可更換區域後重試。
     ![Task1圖](az-700/vnet/image/m5u6\task1_5.png)
    - Solution: 把區域原為- East US改為 East US 2，就可設定成功
     ![Task1圖](az-700/vnet/image/m5u6\task1_6.png)
      ![Task1圖](az-700/vnet/image/m5u6\task1_7.png)

---

### (二) 建立 Azure Front Door

使用 Azure Front Door 根據延遲自動導引流量至最近的 Web App。

1. 建立 Front Door  
   - 在 Azure Portal 中搜尋「Front Door and CDN profiles」  
   - 點選「Create Front Door and CDN profiles」  
   - 選擇「Quick create」，進入建立畫面


2. 填寫以下基本設定：

   | 設定項目           | 值                              |
   |--------------------|----------------------------------|
   | 訂閱               | 選擇你的訂閱                    |
   | 資源群組           | ContosoResourceGroup            |
   | 名稱               | FrontDoor-[你的縮寫]           |
   | 層級               | Standard                        |
   | Endpoint 名稱      | FDendpoint                      |
   | 原始類型           | App Service                     |
   | 原始主機名稱       | WebAppContoso-1（第一個 WebApp）|

   (1) 點選「Review and Create」並部署  
   (2) 部署完成後點選「Go to Resource」

     ![Task2圖](az-700/vnet/image/m5u6\task2_10.png)
     ![Task2圖](az-700/vnet/image/m5u6\task2_11.png)

    2.1 出現異常訊息: 沒有microsft.cdn沒有註冊
   (1) 解決方式：手動註冊 Microsoft.Cdn
    - 請照以下步驟操作即可：
    - 方法一：使用 Azure Portal GUI 註冊
    - 回到 Azure Portal 主畫面
    - 搜尋「Subscriptions」並點進你正在使用的訂閱（如 Lagelab corp）
    - 在左側選單選「Resource providers」
    - 在搜尋框輸入 Microsoft.Cdn
    - 點選後方的「Register」按鈕
    - 等候數十秒至 1 分鐘，狀態會從 NotRegistered 變成 Registered

    ![Task2圖](az-700/vnet/image/m5u6\task2_8.png)
    ![Task2圖](az-700/vnet/image/m5u6\task2_9.png)
    

3. 加入第二個 Origin  
   - 在 Front Door 資源總覽中，選擇左側「Origin Groups」  
   - 點選 `default-origin-group`  
   - 點選「Add an origin」，加入 WebAppContoso-2  
   - 點「Add」後點選「Update」完成設定

    ![Task2圖](az-700/vnet/image/m5u6\task2_12.png)
    ![Task2圖](az-700/vnet/image/m5u6\task2_13.png)

---

### (三) 驗證 Azure Front Door 運作

建立完成後等待全球部署幾分鐘，然後測試其導流與容錯功能。

1. 取得 Front Door Endpoint  
   - 回到 Front Door 總覽頁  
   - 找到 endpoint（例如：`fdendpoint-xxxx.azurefd.net`）並複製
    ![Task3圖](az-700/vnet/image/m5u6\task3_14.png)

2. 使用瀏覽器測試  
   - 貼上該網址應顯示 App Service 預設頁面

3. 測試自動容錯切換  
   - 停止其中一個 Web App（Azure Portal > App Services > Stop）  
   - 回瀏覽器刷新，應仍顯示頁面（自動導向另一 Web App）  
   - 若停止兩個 Web App，再刷新應顯示錯誤頁面

---

## 四、清除資源
1. 開啟 Azure Portal > Cloud Shell > PowerShell  
2. 執行以下指令刪除資源群組，避免額外費用：

```powershell
Remove-AzResourceGroup -Name 'ContosoResourceGroup' -Force -AsJob
```

## 五、延伸學習建議
1. Front Door 與 Application Gateway 差異？
  ![Takeaway圖](az-700/vnet/image/m5u6\task4_16.png)

Q. 那該使用哪一個？  
(1) 你只在「台灣」架一個網站：用 Application Gateway  
(2) 你在「台灣 + 日本 + 美國」都架了 Web App：用 Front Door  

2. Front Door 設定檢查清單？

(1)建立前：
- 你需要有兩個以上的 Web App，最好部署在不同區域
- 你的 Azure 訂閱要先註冊 Microsoft.Cdn，不然會出錯

(2) 建立時：
-設定名稱要唯一
- 要建立一組 origin group，把多個 Web App 加進去
- 設定健康檢查條件（檢查服務掛了會自動切換）

(3) 建立後：
- 測試 endpoint 能不能正常顯示畫面
- 測試把一個 Web App 停掉，流量有沒有自動導向另外一個
- 如果你有自己網址（像 xxx.com），要設定 HTTPS 憑證

3. Origin 與 Endpoint 有何不同？  
(1) Origin（來源）：就是後面處理你請求的東西，比如 Web App、Storage、API  
(2) Endpoint（入口）：是使用者打開的網址（像是門口），例如 fdendpoint-123.azurefd.net  

簡單說：
- 使用者先進到 endpoint
- Front Door 幫你把流量導到適合的 origin

4. 這個Lab的目的: 
(1) 企業「網站不中斷」的實戰做法：
- 現在大多數公司網站都要求：
    a. 全球使用者打開要夠快  
    b. 有一台壞掉也不能當  
    c. 不想被 DDoS 攻擊打死

- 這個 Lab 學習怎麼讓網站在不同區域部署，而且自動切換、不中斷

(2) 實際工作上會遇到的情境
a. 比如：
- 你公司網站有台灣版、日本版、美國版 → 你就會需要像 Front Door 這種「流量導引系統」
- 你要幫公司建立 CDN 加速，讓首頁或圖片載入更快
- 你要做容錯，讓主站掛了自動切去備援站
b. 如果你不知道怎麼設定 Front Door，你會：
- 讓網站只有單點 → 一掛就全炸
- 全球流量都走一個站 → 效能慢、體驗差

---

## 六、自學資源

- [Introduction to Azure Front Door]  
- [Load balance your web service traffic with Front Door]

---

## 七、重點回顧

- Azure Front Door 為全球應用傳遞與負載平衡服務  
- 使用 L7（第七層）負載平衡技術  
- 支援路由策略：延遲、加權、優先順序、Session Affinity  
- 提供健康檢查與容錯切換





