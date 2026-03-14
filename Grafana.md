第一步：登入 Grafana
打開瀏覽器，前往 http://localhost:3000。

預設帳號：admin / 密碼：admin（根據你的 yml 設定）。

第一次登入會要求修改密碼，你可以直接跳過或修改。

第二步：新增資料源 (Data Source)
Grafana 需要知道去哪裡抓 Prometheus 的資料。

點擊左側選單的 Connections -> Data sources。

點擊 Add data source。

選擇 Prometheus。

在 Connection 的 Prometheus server URL 欄位輸入：

http://prometheus:9090

注意：這裡要用 prometheus (服務名稱)，因為它們在同一個 Docker 網路內。

捲動到最下方，點擊 Save & test。

如果出現綠色的「Successfully queried the Prometheus API」，代表連線成功。

第三步：導入 Traefik 官方儀表板 (Dashboard)
不用自己手動畫圖，Traefik 社群已經有做好的專業面板。

點擊左側選單的 Dashboards。

點擊右側的 New -> Import。

在 Import via grafana.com 欄位輸入 ID：17346 (這是 Traefik v3 官方推薦的面板)。

點擊 Load。

在最後一頁的底部的 Prometheus 下拉選單中，選擇你剛剛建立的 Data Source。

點擊 Import。

第四步：確認數據顯示
現在你應該能看到一個非常華麗的介面，包含：

Entrypoints Statistics: 進入 Traefik 的流量總量。

HTTP Requests Response Code: 200, 404, 500 等狀態碼分佈。

Response Time: 服務回應的延遲時間。