---
tags:
  - docker
  - traefik
  - authelia
---
# Giới thiệu về Authelia
Authelia là máy chủ dùng để xác thực và ủy quyền (authentication and authorization) mã nguồn mở, cung cấp xác thực đa yếu tố và đăng nhập một lần (SSO) cho các ứng dụng của bạn thông qua giao diện web. Nó thường được dùng đi kèm với một hệ thống Reverse Proxy ví dụ như NPM (Nginx Proxy Manager) hay Traefik Proxy mà mình cũng đã giới thiệu đến với các bạn trước đó.

- [Trang chủ](https://www.authelia.com/)
- [Trang cấu hình](https://www.authelia.com/configuration/prologue/introduction/)

## Tại sao lại là Authelia
Hiện có nhiều ứng dụng hoạt động tương tự như Authelia là Authentik, Keycloak ..., mặc dù hiện tại Authelia vẫn đang trong quá trình phát triển bước đầu chỉ đủ dùng nhưng mình chọn sử dụng vì tính gọn nhẹ không đòi hỏi cấu hình cao của nó.

## Ứng dụng Authelia và Reverse Proxy
- Authentication (xác thực): 1FA (1 lớp) hoặc 2FA (2 lớp).
- Authorization (ủy quyền): Quản lý truy cập domain theo chính sách dựa trên rule-based, SSO, OpenID Connect 1.0.

``` yaml
access_control:
  rules:
    - domain: dev.example.com
      resources:
        - '^/groups/dev/.*$'
      subject: 'group:dev'
      policy: two_factor
      methods:
        - GET
        - POST
      networks:
        - 192.168.1.0/24
```

## Lệnh khởi tạo một số chuỗi ngẫu nhiên

Tạo client_id
``` bash
tr -cd '[:alnum:]' < /dev/urandom | fold -w "64" | head -n 1
```

Tạo client_secret theo pbkdf2
``` bash
docker run authelia/authelia:latest authelia crypto hash generate pbkdf2 --random --random.length 32 --random.charset alphanumeric
```

Tạo chuỗi Alphanumberic ngẫu nhiên, dùng cho password
``` bash
docker run authelia/authelia:latest authelia crypto hash generate argon2 --random --random.length 64 --random.charset alphanumeric
```

## Deploy Authelia lên Docker
### Tạo thư mực config, thêm file cấu hình.
``` bash
mkdir config && cd config
touch users-database.yml
touch configuration.yml
```

### File cấu hình chính của Authelia
``` yaml title="configuration.yml"
theme: dark
identity_validation:
  reset_password:
    jwt_secret: your_jwt_secret
default_2fa_method: "totp"
server:
  address: 'tcp://0.0.0.0:9091/'
  asset_path: '/config/assets/'
  endpoints:
    enable_pprof: false
    enable_expvars: false
  disable_healthcheck: false
  tls:
    key: ""
    certificate: ""
    client_certificates: []
  headers:
    csp_template: ""
log:
  level: info
  format: json
  file_path: "/var/log/crowdsec/authelia.log"
  keep_stdout: true
telemetry:
  metrics:
    enabled: false
    address: 'tcp://0.0.0.0:9959/metrics'
totp:
  disable: false
  issuer: authelia.com
  algorithm: sha1
  digits: 6
  period: 30
  skew: 1
  secret_size: 32
webauthn:
  disable: false
  timeout: 60s
  display_name: Authelia
  attestation_conveyance_preference: indirect
  user_verification: preferred
ntp:
  address: "time.cloudflare.com:123"
  version: 4
  max_desync: 3s
  disable_startup_check: false
  disable_failure: false
authentication_backend:
  file:
    path: '/config/users_database.yml'
    watch: false
    search:
      email: false
      case_insensitive: false
    password:
      algorithm: 'argon2'
      argon2:
        variant: 'argon2id'
        iterations: 3
        memory: 65536
        parallelism: 4
        key_length: 32
        salt_length: 16
password_policy:
  standard:
    enabled: false
    min_length: 8
    max_length: 0
    require_uppercase: true
    require_lowercase: true
    require_number: true
    require_special: true
  zxcvbn:
    enabled: false
    min_score: 3
access_control:
  default_policy: deny
  networks:
  - name: internal
    networks: 10.0.0.0/8
  rules:
  - domain: "app1.yourdomain.com"
    policy: bypass
  - domain:
    - "web1.yourdomain.com"
    - "web2.yourdomain.com"
    subject:
    - "group:admins"
    policy: two_factor
  - domain:
    - "*.yourdomain.com"
    subject:
    - "group:admins"
    policy: one_factor
session:
  secret: 'insecure_session_secret'
  name: 'authelia_session'
  same_site: 'lax'
  inactivity: '5m'
  expiration: '1h'
  remember_me: '1M'
regulation:
  max_retries: 3
  find_time: 2m
  ban_time: 5m
storage:
  encryption_key: your_encryption_key
  local:
    path: /config/db.sqlite3
notifier:
  disable_startup_check: false
  smtp:
    username: your mail account
    password: your mail password
    address: 'smtp://smtp.maildomain.com:587'
    sender: your mail account
    identifier: YOUR APP
    subject: "[YOUR APP AUTHENTICATION] {title}"
    startup_check_address: yourmail
    disable_require_tls: false
    disable_html_emails: false
    tls:
      skip_verify: false
      minimum_version: TLS1.2
...
```

### File users-database.yml
``` yaml title="users-database.yml"
users:
  john:
    disabled: false
    displayname: 'John Doe'
    password: '$argon2id$v=19$m=65536,t=3,p=2$BpLnfgDsc2WD8F2q$o/vzA4myCqZZ36bUGsDY//8mKUYNZZaR0t4MFFSs+iM'
    email: 'john.doe@authelia.com'
    groups:
      - 'admins'
      - 'dev'
  harry:
    disabled: false
    displayname: 'Harry Potter'
    password: '$argon2id$v=19$m=65536,t=3,p=2$BpLnfgDsc2WD8F2q$o/vzA4myCqZZ36bUGsDY//8mKUYNZZaR0t4MFFSs+iM'
    email: 'harry.potter@authelia.com'
    groups: []
```

### File docker-compose.yml để deploy Authelia lên Docker.
``` yaml title="docker-compose.yml"
---
version: "3.9"
services:
  authelia:
    image: authelia/authelia:latest
    container_name: authelia
    volumes:
      - ./config:/config
      #- /var/log/crowdsec:/var/log/crowdsec # log file khi chạy với crowdsec
    networks:
      - proxy
    security_opt:
      - no-new-privileges:true
    labels:
      - 'traefik.enable=true'
      - 'traefik.http.routers.authelia.rule=Host(`auth.yourdomain.com`)'
      - 'traefik.http.routers.authelia.entrypoints=websecure'
      - 'traefik.http.routers.authelia.tls=true'
      - "traefik.http.routers.authelia.tls.certresolver=prod"
      - 'traefik.http.services.authelia.loadbalancer.server.port=9091'
    expose:
      - 9091
    restart: unless-stopped
    environment:
      - TZ=Asia/Ho_Chi_Minh
    healthcheck:
      disable: true
  redis:
    image: redis:alpine
    container_name: redis
    volumes:
      - ./redis/data:/data
    networks:
      - proxy
    expose:
      - 6379
    restart: unless-stopped
    environment:
      - TZ=Asia/Ho_Chi_Minh
networks:
  proxy:
    external: true #network của Traefik Proxy
```

### Khai báo middleware trong Traefik dynamic file
``` yaml title="config.yml"
middlewares:
  authelia:
      forwardAuth:
        address: 'http://authelia:9091/api/authz/forward-auth'
        trustForwardHeader: true
        authResponseHeaders:
          - 'Remote-User'
          - 'Remote-Groups'
          - 'Remote-Email'
          - 'Remote-Name'
    authelia-basic:
      forwardAuth:
        address: 'http://authelia:9091/api/verify?auth=basic'
        trustForwardHeader: true
        authResponseHeaders:
          - 'Remote-User'
          - 'Remote-Groups'
          - 'Remote-Email'
          - 'Remote-Name'
```
### Gọi middleware Authelia bằng "authelia@file" trong http.routers.
``` yaml title="config.yml"
http:
  routers:
    myweb:
      entryPoints:
        - "websecure"
      rule: "Host (`web.yourdomain.com`)"
      service: myweb
      middlewares: authelia@file
```
hoặc trong labels của docker-compose.yml
``` yaml title="docker-compose.yml"
labels:
      - 'traefik.enable=true'
      - 'traefik.http.routers.myweb.rule=Host(`web.yourdomain.com`)'
      - 'traefik.http.routers.myweb.middlewares=authelia@file'
```