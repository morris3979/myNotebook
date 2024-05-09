# Create a Kubernetes Cluster (Minikube)

## Step1 - 安裝並配置所需工具

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

  - 1.5 配置 Docker Daemon (optional, 需用到 GPU 資源才需要配置)

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
                }
            }

    - 更改 Docker daemon 配置後，重新啟動 Docker 服務使變更生效

            sudo systemctl restart docker

- ### 2. 安裝 [Minikube](https://minikube.sigs.k8s.io/docs/start/)

  - 2.1 安裝

        curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
        sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64

  - 2.2 啟動 Cluster

    ###### 註: [參數設定](https://minikube.sigs.k8s.io/docs/commands/start/)

        minikube start --insecure-registry=10.86.188.68 --cpus 4 --memory 10240 --driver docker --gpus all

  - 2.3 與 cluster 交互

        # 查看 minikube 所有集群配置列表
        minikube profile list

        # 獲取當前所在集群
        kubectl config current-context

        # 切換集群
        kubectl config use-context <cluster-name>

        # 查看 cluster、nodes、pods 狀態
        kubectl get cs
        kubectl get nodes -A
        kubectl get pods -A
