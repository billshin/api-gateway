
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