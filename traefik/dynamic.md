

dynamic.yml 就像是你的 「路由清單」。

想加一個新網站？改 Routers。

想加一個密碼鎖？改 Middlewares。

想把流量導向另一台伺服器？改 Services。


```yaml
http:
  # 1. 路由：決定請求（域名/路徑）要給誰處理
  routers:
    my-app-router:
      rule: "Host(`api.example.com`) && PathPrefix(`/v1`)" # 匹配條件


      entryPoints:
        - websecure        # 使用哪個大門（通常是 443）
        - web              # 或是一般web http


      service: my-service  # 轉發給哪個服務
      middlewares:
        - auth-check       # 經過哪些中間件 , 可以自行串街
        - aaa
        - bbb
      tls:
        certResolver: myresolver # 如果要自動申請 SSL

  # 2. 中間件：在請求到達後端前的「加工」或「攔截」
  middlewares:

    # 第一關：流量限制
    global-rate-limit:
      rateLimit:
        average: 5      # 每秒平均 5 個請求
        burst: 10       # 允許瞬間 10 個請求的爆發
        sourceCriterion:
          ipStrategy:
            depth: 2    # 如果 depth: 2：Traefik 會去讀取 Header 中的 X-Forwarded-For 列表，並從右往左數第 2 個。這樣就能抓到 [真實用戶 IP]，精準打擊壞蛋。

    # 第二關：轉發認證
    api-auth-check:
      forwardAuth:
        address: "http://auth-service:3000/verify" # 你的 Auth 服務
        trustForwardHeader: true
        authResponseHeaders:
          - "X-User-ID" # 驗證成功後，從 Auth API 帶回 User ID 給後端

    auth-check:
      basicAuth:
        users:
          - "admin:$apr1$7j9... (htpasswd 格式)"
    test-header:
      headers:
        customRequestHeaders:
          X-Script-Name: "test"
    redirect-ssl:
      redirectScheme:
        scheme: https
        permanent: true

  # 3. 服務：定義後端真實伺服器的 IP 與 端口
  services:
    my-service:
      loadBalancer:
        servers:
          - url: "http://192.168.1.10:8080"
          - url: "http://192.168.1.11:8080" # 自動做負載均衡
```



### rateLimit 

[官方文件](https://doc.traefik.io/traefik/reference/routing-configuration/http/middlewares/ratelimit/)

rateLimit 基本使用限制

period  - 調整時間計算區間, 不寫的話預設1s
average - 毎period 中 可以接受多少請求
burst   - period 中 ,瞬間流量上限,超過部份額度遞延到下一個period,直到 burst 額度額滿, 返回429