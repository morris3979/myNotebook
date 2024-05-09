# Create a Kubernetes Cluster (Kubeadm)

![image](https://hackmd.io/_uploads/r1fYjW5M0.png =30%x) ![image](https://hackmd.io/_uploads/rkFJnZqzR.png =36%x)


## Step1 - 準備好主節點及工作節點主機，如下表所示

|           |   Name        |   IP              |
|   :----:  |   :----:      |   :----:          |
|   Master  |   k8s-master  |   XX.XX.XXX.OOO   |
|   Worker  |   k8s-node1   |   XX.XX.XXX.OXO   |

## Step2 - 安裝並配置所需工具

- ### 1. 安裝 [containerd](https://k8s.huweihuang.com/project/runtime/containerd/install-containerd)

        # 更新 apt
        sudo apt-get update

        # 安装 containerd
        sudo apt-get update
        sudo apt-get install -y containerd.io

        # 查看運行狀態
        systemctl enable containerd
        systemctl status containerd

  - 修改 containerd 配置

    1. 生成預設配置

            containerd config default > /etc/containerd/config.toml

    2. 修改 CgroupDriver 為 systemd

        ```sudo nano /etc/containerd/config.toml```

            [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
                ...
                [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
                    SystemdCgroup = true  # 預設是 false

- ### 2. 安裝並配置 [kubernetes](https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

    ###### 註: 以下指令適用於 Kubernetes v1.30

  - 2.1 更新 apt 套件索引並安裝使用 Kubernetes apt 倉庫所需的套件：

            sudo apt-get update
            sudo apt-get install -y apt-transport-https ca-certificates curl gpg

  - 2.2 下載用於 Kubernetes 軟體包倉庫的公共簽署金鑰。

            # 如果 `/etc/apt/keyrings` 目錄不存在，則應在 curl 命令之前創建它
            # sudo mkdir -p -m 755 /etc/apt/keyrings
            curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

  - 2.3 新增 Kubernetes apt 倉庫

            echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

  - 2.4 更新apt套件索引，安裝kubelet、kubeadm 和kubectl，並鎖定其版本

            sudo apt-get update
            sudo apt-get install -y kubelet kubeadm kubectl
            sudo apt-mark hold kubelet kubeadm kubectl

  - 2.5 關閉 swap 功能

    ###### 註: k8s 不使用 swap，所以我們需要將他關閉

            sudo swapoff -a #暫時關閉
            sudo vim /etc/fstab # 永久關閉 (需重啟電腦才能更新配置)

## Step3 - 開始建立 Kubernetes 集群

- ### 1. 建立 Control Plane (建立 node 請跳到 2.)

  - 1.1 在主節點主機，確定完成 Step2 全部步驟後，執行下述指令

        sudo kubeadm init --pod-network-cidr=10.244.0.0/16

  - 1.2 複製一份 root 的憑證資訊到 home 目錄下

        mkdir -p $HOME/.kube
        sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
        sudo chown $(id -u):$(id -g) $HOME/.kube/config

  - 1.3 確認 K8S Cluster 是否建立完畢 (確認下面步驟完成，代表 Cluster 建置完畢)

    - 1. ```kubectl get nodes```
    - 2. ```kubectl get cs```

- ### 2. 建立 Worker Node

  - 2.1 先切換到工作節點主機，確定完成 Step2 全部步驟後，執行下述指令

        sudo kubeadm join <server-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<token-ca-cert>

  - 2.2 到 master 中檢查 node1 是否有加入成功

        kubectl get nodes

- ### 3. 安裝 Kubernetes Plugin

    ###### 註: 下述步驟指令只需在 master 執行

  - 3.1 Network Plugin - [flannel](https://github.com/flannel-io/flannel)

        kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml

        # 確認每個 node 狀態皆為 Ready，即完成集群配置
        kubectl get node -w

  - 3.2 GPU Plugin -  [Nvidia device plugin](https://github.com/NVIDIA/k8s-device-plugin) (optional)

        kubectl create -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v0.15.0/deployments/static/nvidia-device-plugin.yml
