---
layout: Article/content
title:  "使用Docker運行Nginx與uWSGI容器服務"
date:   2020-06-27 20:05:00 +0800
categories: [Docker,Nginx,uWSGI]
tag: Docker
excerpt_separator: <!--more-->
---

此筆記內容記錄著依據Nginx整合uWSGI Server服務映像檔，建立一個運行Nginx與uWSGI容器服務。
<!--more-->

有關於**Nginx整合uWSGI Server服務映像檔**可參考-->[使用Docker建置Nginx結合uWSGI映像檔](/CVdNFGTyTqKoMZRwauV2eg)。

相關參考資源:
1. [https://docs.docker.com/engine/reference/commandline/run/](https://docs.docker.com/engine/reference/commandline/run/)
2. [https://medium.com/@VisonLi/docker-%E5%85%A5%E9%96%80-%E7%AD%86%E8%A8%98-part-2-91e4dfa2b365](https://medium.com/@VisonLi/docker-%E5%85%A5%E9%96%80-%E7%AD%86%E8%A8%98-part-2-91e4dfa2b365)
3. [https://medium.com/@honglong/%E7%94%A8-docker-%E8%B7%91-mysql-d09c95c91da3](https://medium.com/@honglong/%E7%94%A8-docker-%E8%B7%91-mysql-d09c95c91da3)

# 建立容器服務
語法格式:
```docker
docker run  --name 自定義容器名稱 --restart always -p 容器對外port:容器內部port -v 本機掛載目錄:容器內部目錄 -d 使用image名稱
```
關於詳細參數項目資訊可參考 [docker run options](https://docs.docker.com/engine/reference/commandline/run/#options)

* **--name**
給予此容器一個自訂識別名稱，可在**docker ps -a**資訊內看到。

* **--restart always**
Restart policies (--restart) https://docs.docker.com/engine/reference/commandline/run/#restart-policies---restart
設定為當Docker運行時，自動啟動此容器服務，不管容器狀態是如何都會被Docker嘗試重新啟動。

* **-p**
設置主機host port對應容器內部port，當有連線需要跟容器對接時，透過設置host port導向到容器內部port。
假設要開放host 8080 port對應容器內部80 port，可以參考下面方式
    ```
    -p 8080:80
    ```
    或
    ```
    -p 127.0.0.1:8080:80
    ```
    
* **-v**
設置在本機上網站目錄掛載進容器內部網站目錄，在內部/web/web_data底下檔案會跟本機上實體網站目錄同步一樣

* **-d**
設置容器要以指定映像檔來建置運作環境。

# 實際建立容器服務範例
運行一個容器服務以Nginx整合uWSGI Server服務映像檔作為建立依據
```docker
docker run  --name flask_helloworld --restart always -p 85:80 -v E:\Project\flask_helloworld:/web/web_data -d nginx_flask_server
```
* 運行一個容器服務識別標籤為 **flask_helloworld**
* 設置總是隨著Docker主程序啟動時，自動啟動容器服
* Host **85** Port對接容器內部 **80** Port
* 掛載一個本機實體位置 **E:\Project\flask_helloworld** 至容器內部 **/web/web_data**
* 使用映像檔 **nginx_flask_server** 為容器內容建置基礎

關於掛載範例Flask程式檔案
[https://github.com/s123600g/FlaskDemoNotes/tree/master/my_flask_web](https://github.com/s123600g/FlaskDemoNotes/tree/master/my_flask_web)

下載完畢放置在掛載本機實體位置內，並手動建立一個名為uwsgi空目錄即可。

執行完可在瀏覽器視窗輸入下列位址: [http://127.0.0.1:85/helloworld](http://127.0.0.1:85/helloworld)

可透過下面指令查看容器資訊
```docker
docker ps -a
```
有關於docker ps指令可參考 [docker ps](https://docs.docker.com/engine/reference/commandline/ps/)

如果要進入容器內部環境可執行以下指令
``` docker
docker exec -it 容器識別名稱 bash
```
用以上範例容器名稱 **flask_helloworld**
```docker
docker exec -it flask_helloworld bash
```
有關於docker exec指令可參考 [https://docs.docker.com/engine/reference/commandline/exec/](https://docs.docker.com/engine/reference/commandline/exec/)

