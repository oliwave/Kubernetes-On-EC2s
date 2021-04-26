# Kubernetes-On-EC2s 

## 問題情境

在建置公有雲資源的時候通常會使用 `IaC` 的工具 (`Terraform`, `CloudFromation`) 來自動化部署和管理基礎設施，可如果像是應用程式或是套件依賴的安裝儘管在這些工具上也可以達成，但就顯得不利索。在這個範例中使用 `CloudFormation + Ansbile` 來搭配完成整個基礎設施和 Kubernetes Cluster (Control Plane with Single Master Node)的自動化建置。

## 前置作業 Prerequisites
在工作機上安裝以下軟體：
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/index.html) 
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) 和配置 [credentials](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html)
- [Boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html) (AWS SDK)


## 專案結構 Project Structure

`site.yaml` 作為 ansible 主要的執行程式

`cloudformation/k8s_tmplate.yaml` 定義在 `AWS` 想要建置的基礎設施 (Ideal State)
  - 啟動 `EC2` 和 `EC2Fleet`
  - 設定 `Security Group` (kubernetes [required ports](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports))

`roles` 劃分不同種類的工作內容
  - `roles/k8s-common` 針對所有伺服器都需要執行的設定(安裝 docker、kubeadm 等等)
  - `roles/master` 針對 master 伺服器 需要執行的設定(安裝 kubectl 以及設定 CNI 等等)
  - `roles/worker` 針對 worker 伺服器 需要執行的設定 (加入 K8s cluster)

## 系統架構圖 System Architecture
![](https://i.imgur.com/MN8RrOy.png)

1. Ansible 藉由 Boto3 (AWS SDK) 呼叫 `CloudFormation` 去建置 `cloudformation/k8s_tmplate.yaml` 所定義的資源

2. Ansible 再由 Boto3 (AWS SDK) 去取得所有 `EC2 instance` 的 Domain Name

3. 透過 `Ansible Dynamic Inventory` 去註冊已取得的 Domain Name

4. Ansible 安裝 Kubernetes 在這些 `EC2 instance` 上

## 使用說明 User Manual

1. 進入 `Kubernetes-On-EC2s` 專案
   - > $ cd <your_workspace>/Kubernetes-On-EC2s

2. 修改 `groups_var/all.yaml` 的 `ansible_ssh_private_key_file` 的 value 為自己的 ec2 key

3. (Optional) 在 `host_vars` 設定 AWS 的客製化參數

4. 執行 ansible 的 playbook
   - > ansible-playbook site.yaml

5. 執行結果
![](https://i.imgur.com/ZsTcdrm.png)

## 功能 Features
- [X] Single master node
- [X] Multiple worker nodes
- [ ] High Availability Control Plane (multi-master cluster)
- [ ] Nodes across multi-az

