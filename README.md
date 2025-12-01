# Nginx-Security-conf
🔹 Nginx 安全架構圖
NGINX 主配置 (nginx.conf)
├── http{} 全域區塊
│ ├── include /etc/nginx/conf.d/security.conf
│ │ ├─ server_tokens off # 隱藏版本
│ │ ├─ Timeout / Buffer 設定
│ │ ├─ SSL/TLS 強化
│ │ ├─ limit_conn_zone / limit_req_zone # 流量區域定義
│ │ └─ map $http_user_agent $bad_bot # 定義惡意 UA
│ ├── include /etc/nginx/conf.d/.conf
│ │ └─ 其他全域 conf（可選）
│ └── include /etc/nginx/conf.d/auto-server-include.conf
│ └─ server {
│ include /etc/nginx/snippets/server-security.conf
│ # 這個 server block 會套用到所有站
│ }
├── server{} （各個站台 / sites-enabled/.conf）
│ ├── listen 80 / 443
│ ├── server_name ...
│ ├── root /var/www/...
│ ├── index ...
│ ├── include /etc/nginx/snippets/server-security.conf
│ │ ├─ limit_conn / limit_req # 針對每個 server 的流量限制
│ │ ├─ if ($bad_bot) { return 403; } # 阻擋惡意 bot
│ │ └─ location ~ /.(git|svn|env)$ { deny all; } # 隱藏敏感檔案
│ └── location / { ... } # 原本站點的路徑設定
└── events {} # 保留 Nginx 原始設定



---

## 規則套用原理

1. **http{}**
   - 放全域可用設定：timeout、buffer、SSL/TLS、全域流量區域、map 定義
   - **不放 if / location**，避免語法錯誤

2. **server{} snippet（server-security.conf）**
   - 放每個 server 專用的限制：
     - `limit_conn` / `limit_req`
     - `if ($bad_bot)`
     - `location` 隱藏敏感檔案
   - **只能出現在 server{} 或 location{}**

3. **auto include server{}**
   - `/etc/nginx/conf.d/auto-server-include.conf` 創建一個 server{}
   - include snippet，確保 snippet 自動套用到所有站
   - **不需要手動改每個 sites-enabled/*.conf**

4. **各站點 server{}**
   - 原本的站點配置正常運作
   - snippet 自動生效，不會覆蓋現有設定

---

## 特性

- 所有站台自動套用安全規則
- 支援多站 / 多虛擬主機
- 防止 BadBot、掃描器、隱藏敏感檔案
- 支援 `limit_conn` / `limit_req` 流量限制
- 不會出現 `nginx -t` 語法錯誤
- 可與原有 server{} location 設定共存
