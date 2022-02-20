---
layout: Article/content
title:  "Docker開啟Remote API與遠程管理設置筆記"
date:   2021-01-25 23:01:00 +0800
categories: [docker]
tag: docker
excerpt_separator: <!--more-->
---

<!--more-->

[Configure where the Docker daemon listens for connections](https://docs.docker.com/engine/install/linux-postinstall/#configure-where-the-docker-daemon-listens-for-connections)

在官方說明文件內有提到兩種方式
* 使用**systemd unit file** <-- 此筆記主要使用此方式
* 使用**daemon.json**

需要注意兩種方式一起設置使用的話，會造成`docker.service`啟動時衝突導致服務無法啟動。

此筆記操作環境是在`Ubuntu 20.04 Server`環境下，在安裝系統設定中並未勾選預先裝`Docker`，而是事後系統安裝完畢啟動後，在執行手動安裝`Docker`。

關於安裝`Docker`可參考
https://docs.docker.com/engine/install/ubuntu/

## Step 1. 編輯`docker.service  unit file`
```shell
sudo systemctl edit docker.service
```

設置以下內容
```
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:4243
```
* 官方預設TCP是配置127.0.0.1，筆者自己設置為`0.0.0.0`允許外部IP連入
* 官方預設是配置*2375* port，筆者自己使用的是*4243* port，可按照自己的需求設定。

因是在`nano`編輯器底下設定，設置完成後請按`Ctrl+x`-->`y`-->`Enter`儲存內容。

## Step 2. 重新啟動`systemctl`刷新配置檔案
```shell
sudo systemctl daemon-reload
```

## Step 3. 重新啟動`Docker`服務
```shell
sudo systemctl restart docker.service
```

確認服務啟動狀態
```shell
sudo systemctl status docker.service
```

當重新啟動完成後，可以用下列指令確認目前監聽狀態資訊
```shell
sudo netstat -lntp | grep dockerd
```

## Step 4. 關於防火牆操作 - `ufw`

* 確認Ubuntu Server 防火牆目前是否有啟動
```shell
sudo ufw status
```
如果要在觀察更仔細資訊內容，可以加`verbose`參數
```shell
sudo ufw verbose
```

* 啟動防火牆指令
```shell
sudo ufw enable
```

如果需要其他參數說明幫助可用以下指令，來進一步查看可用的參數
```shell
sudo ufw -h
```

* 設置防火牆規則為Default，並且拒絕掉所有的連入的請求只允許連出
```shell
sudo ufw default deny
```
在`Default`的規則中內容可以看到如下
```
Default: deny (incoming), allow (outgoing), deny (routed)
```

### 將Docker Remote API Port 在防火牆設置允許指定來源連入規則
筆者習慣針對來源連線進行控管，在這裡只允許指定的IP來進行遠程連線，在此部分設定為只允許IP`10.0.0.10`(*筆者電腦在內部網路的 **IP***)透過`4243`Port來通過防火牆，進一步取得跟Docker Remote API溝通。

* 指令格式
```shell
sudo ufw allow from '你的來源連線IP' to any port '開放允許使用的Port'
```

* 實際執行內容
```shell
sudo ufw allow from 10.0.0.10 to any port 4243
```

在設定完成此規則後，除了指定允許的IP之外，其他IP透過此Port連入至系統時，會被防火牆給擋掉不給予連入。

