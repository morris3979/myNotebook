# Create a Private Docker Registry

## Step1 安裝並配置所需工具

- ### 1. 安裝 [Docker](https://docs.docker.com/engine/install/ubuntu/#installation-methods)

  - 1.1 設定 Docker 的 apt 儲存庫

        # Add Docker's official GPG key:
        sudo apt-get update
        sudo apt-get install ca-certificates curl
        sudo install -m 0755 -d /etc/apt/keyrings
        sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
        sudo chmod a+r /etc/apt/keyrings/docker.asc

        # Add the repository to Apt sources:
        echo \
        "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
        $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
        sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
        sudo apt-get update

  - 1.2 安裝 Docker 軟體包

        sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

  - 1.3 透過執行 hello-world 映像來驗證 Docker 引擎安裝是否成功

        sudo docker run hello-world

  - 1.4 加入 docker 群組 && 暫時先取得 docker 群組 (optional)

        sudo usermod -aG docker $USER && newgrp docker

- ### 2. 產生 SSL 憑證

        # 建立憑證目錄
        sudo mkdir /certs/
        sudo openssl req -newkey rsa:4096 -nodes -sha256 -keyout /certs/domain.key -x509 -days 365 -out /certs/domain.crt

- ### 3. 使用 htpasswd 建立身份驗證

        # 安裝 apache2-utils
        sudo apt-get install apache2-utils

        # 建立憑證存放資料夾
        sudo mkdir -p /var/registry/auth
        cd /var/registry/auth

        # 建立使用者及密碼
        sudo htpasswd -Bbn <user> <password> | sudo tee ./htpasswd > /dev/null
        ls -l htpasswd
        cat htpasswd

- ### 4. 建立 Docker registry 設定檔

  ###### 註: 確認完成上述產生 SSL 憑證及使用 htpasswd 建立身份驗證後，才可進行此步驟

        # 建立設定檔目錄
        mkdir /etc/docker/registry/
        sudo nano /etc/docker/registry/my-reg-config.yml

        # 將如下所示的配置複製並貼上到文件中
        version: 0.1
        log:
        fields:
            service: registry
        storage:
        delete:
            enabled: true
        cache:
            blobdescriptor: inmemory
        filesystem:
            rootdirectory: /var/lib/registry
        http:
        addr: :443
        tls:
        #    certificate: /etc/mkcert/localhost.pem
        #    key: /etc/mkcert/localhost-key.pem
            certificate: /certs/domain.crt
            key: /certs/domain.key
        headers:
            X-Content-Type-Options: [nosniff]
            Access-Control-Allow-Origin: ['*']
            Access-Control-Allow-Methods: ['*']
            Access-Control-Allow-Headers: ['Authorization', 'Accept']
            Access-Control-Max-Age: [1728000]
            Access-Control-Allow-Credentials: [true]
            Access-Control-Expose-Headers: ['Docker-Content-Digest']
        auth:
        htpasswd:
            realm: "Registry Realm"
            path: /auth/auth.htpasswd

## Step2 啟動 Docker registry

    docker run --restart always -d -p 443:443 --name myregistry \
    -v /data/myregistry:/var/lib/registry \
    -v /etc/docker/registry/my-reg-config.yml:/etc/docker/registry/config.yml \
    -v /var/registry/auth/htpasswd:/auth/auth.htpasswd \
    -v /certs:/certs registry:latest

## Step3 配置 Docker Daemon

- 將 Docker daemon 設定為信任 SSL 證書

        # 如果該文件尚不存在，則建立該文件
        sudo nano /etc/docker/daemon.json

        # 將如下所示的配置複製並貼上到文件中
        {
            "runtimes": {
                "nvidia": {
                    "args": [],
                    "path": "nvidia-container-runtime"
                }
            },
            "insecure-registries": ["<your-domain-or-ip>"]
        }

- 更改 Docker daemon 配置後，重新啟動 Docker 服務使變更生效

        sudo systemctl restart docker

## Step4 Pull / Push image

- 從 Docker Hub 拉取 Nginx 映像

        sudo docker pull nginx

- 使用私有 registry 位址標記拉取的 Nginx 映像

        sudo docker tag nginx:latest <your-domain-or-ip>/<image-name>:<tag>

- 將標記的映像推送到私人 docker registry

        sudo docker push <your-domain-or-ip>/<image-name>:<tag>
