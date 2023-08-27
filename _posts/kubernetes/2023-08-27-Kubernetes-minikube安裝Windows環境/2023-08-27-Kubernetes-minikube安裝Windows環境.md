---
layout: Article/content
title:  "Kubernetes - minikube安裝Windows環境"
date:   2023-08-27 15:20:00 +0800
categories: [kubernetes,minikube]
tag: kubernetes
img_category_num: kubernetes_minikube
excerpt_separator: <!--more-->
---

<!--more-->

Install Tools：[https://kubernetes.io/docs/tasks/tools/](https://kubernetes.io/docs/tasks/tools/)

minikube start：[https://minikube.sigs.k8s.io/docs/start/](https://minikube.sigs.k8s.io/docs/start/)

系統需求可參考：[https://minikube.sigs.k8s.io/docs/start/#what-youll-need](https://minikube.sigs.k8s.io/docs/start/#what-youll-need)

minikube GitHub： [https://github.com/kubernetes/minikube](https://github.com/kubernetes/minikube)

Install Docker Desktop on Windows：[https://docs.docker.com/desktop/windows/install/](https://docs.docker.com/desktop/windows/install/)

安裝使用環境
* Windows 11 Pro 10.0.22000 Build 22000
* Docker

安裝順序
1. kubectl
2. minikube

## Windows 安裝 kubectl

[https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/)

下載 **kubectl.exe** 並執行安裝，不使用執行`curl`操作安裝方式。

<img src="kubernetes_minikube_1.png" class="img-fluid rounded mx-auto" >

測試安裝是否正確
* 顯示安裝版本資訊
```shell
kubectl version --client
```

<img src="kubernetes_minikube_2.png" class="img-fluid rounded mx-auto" >

* 顯示安裝版本資訊(詳細)
```shell
kubectl version --client --output=yaml  
```

<img src="kubernetes_minikube_3.png" class="img-fluid rounded mx-auto" >

## Windows 安裝 minikube

GitHub Release，下載 **minikube-installer.exe** 並執行安裝
[https://github.com/kubernetes/minikube/releases](https://github.com/kubernetes/minikube/releases)

<img src="kubernetes_minikube_4.png" class="img-fluid rounded mx-auto" >

<img src="kubernetes_minikube_5.png" class="img-fluid rounded mx-auto" >

### 執行啟動

開啟**PowerShell**進行以下操作
```shell
minikube start
```

<img src="kubernetes_minikube_6.png" class="img-fluid rounded mx-auto" >

在此階段會設定必要配置與下載執行啟動必要相關套件。

<img src="kubernetes_minikube_7.png" class="img-fluid rounded mx-auto" >

啟動完成後，打開**Docker Desktop**可以看到**minikube**容器運行中。

<img src="kubernetes_minikube_8.png" class="img-fluid rounded mx-auto" >

### 運行 minikube dashboard

```shell
minikube dashboard
```

<img src="kubernetes_minikube_10.png" class="img-fluid rounded mx-auto" >

預設會自動開啟瀏覽器瀏覽頁面

<img src="kubernetes_minikube_11.png" class="img-fluid rounded mx-auto" >

如果不要預設自動開啟瀏覽器，可加上`--url`參數
```shell
minikube dashboard --url
```

使用kubectl指令可查看當前cluster資訊
```shell
kubectl cluster-info
```

<img src="kubernetes_minikube_12.png" class="img-fluid rounded mx-auto" >

---