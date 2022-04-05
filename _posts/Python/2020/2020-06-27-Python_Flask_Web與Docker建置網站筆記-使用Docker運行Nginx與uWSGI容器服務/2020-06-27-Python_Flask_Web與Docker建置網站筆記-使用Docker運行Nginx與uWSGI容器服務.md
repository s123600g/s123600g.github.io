---
layout: Article/content
title:  "Python_Flask_Web與Docker建置網站筆記-使用Docker運行Nginx與uWSGI容器服務"
date:   2020-06-27 21:01:00 +0800
categories: [python,flask,docker,nginx,uwsgi]
tag: python
img_category_num: python_04
excerpt_separator: <!--more-->
---

此筆記內容記錄著依據Nginx整合uWSGI Server服務映像檔，建立一個運行Nginx與uWSGI容器服務。

<!--more-->

GitHub: [https://github.com/s123600g/FlaskDemoNotes/tree/master/flask_base_framework](https://github.com/s123600g/FlaskDemoNotes/tree/master/flask_base_framework)


配合筆記： [Python_Flask_Web與Docker建置網站筆記-使用Docker建置Nginx結合uWSGI映像檔](https://s123600g.github.io/python/flask/docker/nginx/uwsgi/2020/06/27/Python_Flask_Web%E8%88%87Docker%E5%BB%BA%E7%BD%AE%E7%B6%B2%E7%AB%99%E7%AD%86%E8%A8%98-%E4%BD%BF%E7%94%A8Docker%E5%BB%BA%E7%BD%AENginx%E7%B5%90%E5%90%88uWSGI%E6%98%A0%E5%83%8F%E6%AA%94.html)

---

## 建立一個容器服務

語法格式:
```
docker run  --name 自定義容器名稱 --restart always -p 容器對外port:容器內部port -v 本機掛載目錄:容器內部目錄 -d 使用image名稱
```

關於詳細參數項目資訊可參考：
[https://docs.docker.com/engine/reference/commandline/run/#options](https://docs.docker.com/engine/reference/commandline/run/#options)


* `-–name`
給予此容器一個自訂識別名稱，可在docker ps -a資訊內看到。

* `-–restart always`
設定為當Docker運行時，自動啟動此容器服務，不管容器狀態是如何都會被Docker嘗試重新啟動。

* `-p`
設置主機host port對應容器內部port，當有連線需要跟容器對接時，透過設置host port導向到容器內部port。
假設要開放host 8080 port對應容器內部80 port，可以參考下面方式
```
-p 8080:80
```
或
```
-p 127.0.0.1:8080:80
```

* `-v`
設置在本機上網站目錄掛載進容器內部網站目錄，在內部/web/web_data底下檔案會跟本機上實體網站目錄同步一樣。

* `-d`
設置容器要以指定映像檔來建置運作環境。

---

## 實際建立一個容器服務範例

運行一個容器服務以Nginx整合uWSGI Server服務映像檔作為建立依據
```shell
docker run  --name flask_helloworld --restart always -p 85:80 -v E:\Project\flask_helloworld:/web/web_data -d nginx_flask_server
```

1. 運行一個容器服務識別標籤為 flask_helloworld
2. 設置總是隨著Docker主程序啟動時，自動啟動容器服
3. Host 85 Port對接容器內部 80 Port
4. 掛載一個本機實體位置 E:\Project\flask_helloworld 至容器內部 /web/web_data
5. 使用映像檔 nginx_flask_server 為容器內容建置基礎

關於掛載範例Flask程式檔案
[https://github.com/s123600g/FlaskDemoNotes/tree/master/my_flask_web](https://github.com/s123600g/FlaskDemoNotes/tree/master/my_flask_web)

下載完畢放置在掛載本機實體位置內，並手動建立一個名為uwsgi空目錄即可。<br/>

執行完可在瀏覽器視窗輸入位址: `http://127.0.0.1:85/helloworld`

可透過下面指令查看容器資訊
```
docker ps -a
```

有關於docker ps指令可參考： [https://docs.docker.com/engine/reference/commandline/ps/](https://docs.docker.com/engine/reference/commandline/ps/)


如果要進入容器內部環境可執行以下指令
```shell
docker exec -it 容器識別名稱 bash
```

用以上範例容器名稱 flask_helloworld
```shell
docker exec -it flask_helloworld bash
```

有關於docker exec指令可參考： [https://docs.docker.com/engine/reference/commandline/exec/](https://docs.docker.com/engine/reference/commandline/exec/)

---

## 其他容器服務

**portainer** <br/>
方便控管容器GUI介面工具

```shell
docker run -d -p 9000:9000 -p 8000:8000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v E:\app\Docker\portainer_data:/data portainer/portainer
```

* [https://www.portainer.io/](https://www.portainer.io/)
* [https://www.portainer.io/installation/](https://www.portainer.io/installation/)
* [https://alanhou.org/portainer-gui-docker/](https://alanhou.org/portainer-gui-docker/)
* [https://xupeiyao.github.io/2019/05/02/ubuntu_install_docker_and_portainer/](https://xupeiyao.github.io/2019/05/02/ubuntu_install_docker_and_portainer/)


**MySQL**

```shell
docker run --name jyu-mysql -e MYSQL_ROOT_PASSWORD=password -d mysql:latest
```

* [docker hub mysql](https://hub.docker.com/_/mysql)

後面範例實作會用到此容器服務MYSQL，因為是範例實作所以MySQL Root密碼會用簡單password來設置，實際狀況下使用時請謹慎設置Root密碼。

---


