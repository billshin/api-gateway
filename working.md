








🎯 所以 metrics「可以放什麼？」
✅ 適合放（低基數）

status（200 / 500）

method（GET / POST）

service name

endpoint（有限數量）

instance（機器數量）

❌ 不適合放（高基數）

user_id ❌

IP ❌

request_id ❌

session_id ❌

email ❌

👉 這些都應該丟給：
👉 Loki


👉 Traefik 分三層：

1️⃣ entrypoint
👉 「門（port）」

2️⃣ router
👉 「規則（path / host）」

3️⃣ service
👉 「真正 API」


👉 Prometheus 是「預先聚合」 - 事先設計好的統計資料
時間序列資料庫（TSDB)
👉 Loki 是「查詢時才聚合」 - 完整事件紀錄


👉 Prometheus
= 看「系統狀況」（數字、趨勢）

👉 Loki
= 查「發生什麼事」（每一筆請求）



Grafana + Loki 可以做到：

查誰被 429

查哪個 API 被打爆

查 request 細節

👉 Grafana + Prometheus 可以做到：

429 趨勢圖

QPS

latency





auth api 
> Prometheus 
> Promtail  > loki 




--------

當你系統長大，你會需要：

🔹 1️⃣ 多服務

auth-service

user-service

order-service

🔹 2️⃣ 多監控

Prometheus

Loki

Alertmanager

🔹 3️⃣ 告警系統

error rate 過高

latency 過高



### traefik  Load balance

http:
  services:
    auth-service:
      loadBalancer:
        servers:
          - url: "http://auth1:3000"
          - url: "http://auth2:3000"
        healthCheck:
          path: /health
          interval: 10s
          timeout: 3s
		retry:
		  attempts: 2

定期呼叫,健康才提供


### traefik 如何透過 auth api 進行認證

Client
  ↓
Traefik（ForwardAuth middleware）
  ↓
Auth API（你寫的）
  ↓
回應：200 / 401
  ↓
Traefik 決定要不要放行


*Auth API 不能太慢*

👉 因為：  每個 request 都會打一次

👉 建議： JWT（無狀態）Redis cache

如果Auth API 有單點壞掉的可能
可以讓Traefik 做一次LB , 並啟用healthy check, 自動避開壞掉的Auth API

middleware 在接到traefik 這節點即可

監控指標
https://ithelp.ithome.com.tw/m/articles/10349002
Promtail 
https://ithelp.ithome.com.tw/articles/10331661
