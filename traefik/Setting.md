
treafik 原本可以針對同機器docker 上進行


.
├── docker-compose.yml
├── traefik.yml          # 靜態配置 (Static Config)
└── dynamic.yml     # 動態配置 (Dynamic Config)


使用 dynamic.yml 的缺點
管理需要使用檔案


plan B
providers:
  http:
    # 這裡填寫你 Python API 的位址
    endpoint: "http://host.docker.internal:8000/traefik-config"
    pollingInterval: "5s" # 每 5 秒自動更新一次



    
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