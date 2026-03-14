
prometheus.yml
基本上就是吃 traefik 產出的資料
http://localhost:9100/metrics , 可以打開就沒問題

1. 透過 Prometheus UI 查詢原始數據
這是最直接的方法。請打開瀏覽器前往 http://localhost:9090，在 Expression 欄位輸入以下任一指令並按 Execute：

traefik_config_reloads_total：如果數值大於 0，表示 Traefik 至少成功加載過設定。

traefik_entrypoint_requests_total：這會顯示各個入口（web, metrics）收到的請求總數。

up{job="traefik"}：如果回傳值是 1，代表抓取週期正常。

切換到 Graph 頁籤，如果你看到線條正在波動，那就代表資料持續在寫入。