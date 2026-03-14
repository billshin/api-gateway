# Whoami 簡介

**Whoami** 是一個超輕量的 HTTP 測試服務，常用於測試 **API Gateway / Reverse Proxy / Load Balancer**，它本身不提供業務功能，只回傳 HTTP 請求資訊。

---

## 主要用途

1. **測試 Routing**
   - 檢查請求是否正確被 Traefik、NGINX 等 gateway 導向指定服務
   - 例如不同 Host、不同 Path 的請求

2. **檢查 Header**
   - 顯示原始請求 Header，例如 `Host`、`X-Forwarded-For`、`X-Real-IP`
   - 可確認 gateway 是否正確加上或修改 Header

3. **測試 Load Balancing**
   - 當多個 whoami container 同時運行，刷新頁面可看到請求分配到哪個 container
   - 容器 Hostname 用來辨識不同實例

4. **檢查 Container / Network**
   - 顯示 container 的 Hostname 與 IP
   - 幫助確認 Docker network 或 Kubernetes service 是否正常連通

---

## 使用方式

### Docker 簡單測試

```bash
docker run --rm -p 81:80 traefik/whoami
```