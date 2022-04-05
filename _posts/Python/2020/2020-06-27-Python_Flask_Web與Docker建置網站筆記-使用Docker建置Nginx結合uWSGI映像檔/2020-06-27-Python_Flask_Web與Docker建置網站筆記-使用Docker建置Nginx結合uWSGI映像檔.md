---
layout: Article/content
title:  "Python_Flask_Web與Docker建置網站筆記-使用Docker建置Nginx結合uWSGI映像檔"
date:   2020-06-27 19:01:00 +0800
categories: [python,flask,docker,nginx,uwsgi]
tag: python
img_category_num: python_03
excerpt_separator: <!--more-->
---

此筆記為筆者學習建立一個Nginx整合uWSGI Server服務 Docker映像檔，紀錄建立一個Dockerfile檔案到建置完成過程。

<!--more-->

GitHub: [https://github.com/s123600g/FlaskDemoNotes/tree/master/Docker/Nginx_uWSGI_Flask](https://github.com/s123600g/FlaskDemoNotes/tree/master/Docker/Nginx_uWSGI_Flask)


> Docker Contianer → 在筆記稱為**容器**。 <br/>
> Docker Image →在筆記稱為**映像檔**。

---

根據`Docker\Nginx_uWSGI_Flask\` 底下會有檔案內容如下:
1. Dockerfile
2. nginx.conf
3. requirements.txt
4. start_up.sh
5. web.conf

## 映像檔建置操作

**Step1. 開啟終端機並將起始位置切換至Dockerfile位置底下**
```shell
cd  path to\dockerfile
```

**Setp2. 執行建立映像檔**
```shell
docker build -t "自訂映像檔名稱:自訂Tag" .
```
實際範例，建置一個命名為*nginxflaskserver*映像檔
```shell
docker build -t nginx_flask_server .
```

---

## 映像檔匯出操作

```shell
docker save --output 檔案名稱.tar image識別標籤
```

實際範例，匯出一個映像實體檔(nginx_flask_server.tar)
```shell
docker save --output nginx_flask_server.tar nginx_flask_server
```

## 映像檔匯入操作

```shell
docker load --input 檔案名稱.tar
```

實際範例，匯入一個映像實體檔(nginx_flask_server.tar)
```shell
docker load --input nginx_flask_server.tar
```

---

## 映像檔刪除操作

```shell
docker rmi image識別標籤
```

實際範例，刪除一個映像媒體(nginx_flask_server)
```shell
docker rmi nginx_flask_server
```

---

## 列出目前所有映像檔資訊

```shell
docker images
```

---

## Dockerfile內容

<script src="https://gist.github.com/s123600g/4ff15b02a611b2adff1c7b4e155789f1.js"></script>

**(1) 指定本機與容器之間連接Port**
```
ENV web_port=85
```

預設是設定85為容器開放Port。 <br/>

*請注意此設置代表之後以此映像檔建立之容器，開放有效溝通端口為85 Port，實際上還需要在docker run進行開通設定才能完全讓本機跟容器對接起來。*

<p></p>

**(2) 預設容器內部網站目錄指向為 /web/web_data**
```
chdir=/web/web_data/
```

<p></p>

**(3) 複製必要檔案至容器內部**
```
COPY '檔案名稱' '容器內部位置'
```

映像檔在建置過程中，會複製當前所在位置底下內有存在指定檔案至內部對應位置。 <br/>

**nginx.conf**
```
COPY "nginx.conf" "/web"
```

更改Nginx內部主設定配置檔案，覆蓋掉原本官方預設配置檔案內容。 <br/>
先複製要覆蓋的配置檔在內部`/web`底下，再透過下面程序環節更新覆蓋掉官方預設配置檔案
```
RUN mv /etc/nginx/nginx.conf /etc/nginx/nginx_old.conf \
    && cp /web/nginx.conf /etc/nginx/
```

**web.conf**
```
COPY "web.conf" "/etc/nginx/sites-enabled"
```

Nginx對應uWSGI Socket配置檔放置內部`/etc/nginx/sites-enabled`底下，此為Nginx反向代理器與Flask網站uWSGi Server對接配置。 <br/>


**requirements.txt**
```
COPY "requirements.txt" "/web"
```

Python套件需求名單文字檔，在映像檔建置過程中，順便安裝運行Python環境所需要套件，會依據此文字檔內紀錄套件名稱進行下載安裝，內容如下
```
alembic==1.3.0
aniso8601==8.0.0
astroid==2.3.1
autopep8==1.4.4
certifi==2019.9.11
Click==7.0
colorama==0.4.1
DateTime==4.3
Flask==1.1.1
Flask-Admin==1.5.4
Flask-CKEditor==0.4.3
Flask-Login==0.4.1
Flask-Migrate==2.5.2
Flask-RESTful==0.3.7
Flask-Script==2.0.6
Flask-Session==0.3.1
Flask-SQLAlchemy==2.4.1
Flask-WTF==0.14.2
isort==4.3.21
itsdangerous==1.1.0
Jinja2==2.10.3
lazy-object-proxy==1.4.2
Mako==1.1.0
MarkupSafe==1.1.1
mccabe==0.6.1
pycodestyle==2.5.0
pylint==2.4.2
python-dateutil==2.8.0
python-editor==1.0.4
pytz==2019.3
pymssql==2.1.4
six==1.12.0
SQLAlchemy==1.3.10
typed-ast==1.4.0
Werkzeug==0.16.0
wincertstore==0.2
wrapt==1.11.2
WTForms==2.2.1
zope.interface==4.6.0
openpyxl==3.0.3
uWSGI==2.0.18
mysqlclient==1.4.6
```

**start_up.sh**
```
COPY "start_up.sh" "/var/local"
```
此為Nginx與uWSGI自啟動腳本，在映像檔建置最後步驟中，設置每當以此映像檔建置之容器，每一次啟動時，優先執行此自啟動腳本設置網站服務運作。
透過最後一道程序，來設置每次容器啟動時首先執行此腳本
```
CMD [ "/bin/bash", "-c" , "/var/local/start_up.sh ;"]
```

關於此自啟動腳本內容如下
```bash
#!/bin/bash
nginx -g 'daemon off;' | uwsgi --ini /web/web_data/uwsgi.ini
```

關於腳本內部添加nginx全域設定-關掉守護線程
```
nginx -g 'daemon off;'
```

將 Nginx 設置為執行啟動時不依附上層核心程序進行，避免因為上層核心程序終結而導致連帶中止結束，因為容器啟動執行時，Nginx服務是以程序驅動執行腳本而不是一個獨立程序，導致腳本執行完畢時，因上層執行驅動腳本核心程序結束，導致連帶一起關閉腳本所啟用服務程序連帶中止，也就是一開始pid=1為執行腳本核心程序，一但結束此核心程序那Nginx也連帶一起中止，docker會判定此容器工作已經結束執行，自動關閉容器服務。


---

## Nginx

Nginx主體配置檔為`nginx.conf`

<script src="https://gist.github.com/s123600g/7f9435ec887f9000a5c42d11e9dabf35.js"></script>

Nginx web socket配置檔為web.conf

<script src="https://gist.github.com/s123600g/90b9b3c7b331f4f6c4be74ce25a582fa.js"></script>

Nginx參數說明：
1. listen → 指定Nginx要監聽的Port
2. server_name → 主機Host位址
3. access_log → Nginx 連線紀錄日誌放置位置
4. error_log → Nginx 連線錯誤紀錄日誌放置位置
5. location → 設置監聽端口Port
6. uwsgi_pass： 配置指定的uwsgi連線管道(Web Socket)
7. include：將指定的uwsgi配置全部包含讀取進來

---


