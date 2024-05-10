# Create a minio object storage

## Step1 安裝並配置所需工具

- ### 安裝 [Docker](https://docs.docker.com/engine/install/ubuntu/#installation-methods)

  - 1. 設定 Docker 的 apt 儲存庫

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

  - 2. 安裝 Docker 軟體包

            sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

  - 3. 透過執行 hello-world 映像來驗證 Docker 引擎安裝是否成功

            sudo docker run hello-world

  - 4. 加入 docker 群組 && 暫時先取得 docker 群組 (optional)

            sudo usermod -aG docker $USER && newgrp docker

## Step2 啟動 minio object storage

- 運行 minio object storage

        docker run --restart always -p 9000:9000 -p 9001:9001 --name minio --user root -e "MINIO_ROOT_USER=<username>" -e "MINIO_ROOT_PASSWORD=<password>" -v <local-path>:/storage minio/minio server /storage --console-address ":9001"

- 查看 minio 是否正在運行

        docker ps
