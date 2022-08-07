---
layout: Article/content
title:  "使用EdgeTpu應用在語音模型預測之簡單實例(六)-進行EdgeTpu模型預測"
date:   2019-07-21 00:00:00 +0800
categories: [google,EdgeTpu]
tag: google
img_category_num: edgetpu06
excerpt_separator: <!--more-->
---

<!--more-->

GitHub：[https://github.com/s123600g/asr_edgetpu_demo](https://github.com/s123600g/asr_edgetpu_demo)

經過[(五) 使用 edgetpu_compiler 轉換 EdgeTpu 可識別 tflite模型](https://s123600g.github.io/google/edgetpu/2019/07/20/%E4%BD%BF%E7%94%A8EdgeTpu%E6%87%89%E7%94%A8%E5%9C%A8%E8%AA%9E%E9%9F%B3%E6%A8%A1%E5%9E%8B%E9%A0%90%E6%B8%AC%E4%B9%8B%E7%B0%A1%E5%96%AE%E5%AF%A6%E4%BE%8B(%E4%BA%94)-%E4%BD%BF%E7%94%A8edgetpu_compiler%E8%BD%89%E6%8F%9BEdgeTpu%E5%8F%AF%E8%AD%98%E5%88%A5tflite%E6%A8%A1%E5%9E%8B.html)，此階段使用EdgeTpu模型進行簡單預測。

此階段所使用程式為`classify_ASR.py`與`classification_engine.py`，執行時需要搭配**Google Coral USB Accelerator**。

`classify_ASR.py`為主要執行程式，調用`classification_engine.py`程式取得最後EdgeTpu模型預測結果，在將此結果進行分類標籤識別。

`classification_engine.py`為`classify_ASR.py`調用分支程式，主要處理EdgeTpu模型在分類預測處理方面，此為參考官方範例中`classify_image.py`內所調用 edgetpu.classification.engine-ClassificationEngine()進行修改實作。

關於`classify_image.py`原碼可參考： <br/>
[https://coral.googlesource.com/edgetpu/+/refs/heads/release-chef/edgetpu/demo/classify_image.py](https://coral.googlesource.com/edgetpu/+/refs/heads/release-chef/edgetpu/demo/classify_image.py)

關於edgetpu.basic.basic_engine說明可參考： <br/>
[https://coral.withgoogle.com/docs/reference/edgetpu.basic.basic_engine/](https://coral.withgoogle.com/docs/reference/edgetpu.basic.basic_engine/)

---

## 程式執行

使用`classify_ASR.py`。

<script src="https://gist.github.com/s123600g/5cea6f1ad87ac129f1e00831d0aa75cc.js"></script>

需要先設置確認EdgeTpu模型來源

```python
tf_model = "ASR_Model_edgetpu.tflite" # 模型名稱

tf_model_dir = "tflite_model" # 模型放置目錄名稱
```

確認預測資料來源，這裡只使用一筆語音特徵來源

這裡使用語音特徵為前面語音資料集處理階段，已經擷取轉換好語音特徵之文字檔。

存在位置在專案目錄內 `tflite_model/validdata/down/0a2b400e_nohash_0.txt`


不需要給予指定語音特徵文字檔名稱，如上面`0a2b400e_nohash_0.txt`完整檔案名稱，只要給予檔案存在位置到目錄名稱即可，會自動掃描抓取目錄內檔案。

```python
valid_data_path = os.path.join(    
    os.getcwd(), # 自動判斷專案位置    
    "tflite_model", # 設置專案目錄內放置預測資料來源目錄之最外層    
    "validdata", # 設置專案目錄內放置預測資料來源目錄之內層
)
```

需要注意的是資料型態，在讀取預測語音特徵來源時，資料型態是float32。

因為轉換後EdgeTpu模型，在針對輸入的資料來源之型態上，要求為uint8型態給予模型，所以在讀取完語音特徵內容後，要將其資料型態轉換為uint8形式。

```python
''' 轉換語音特徵向量為numpy array，並且將資料型態轉為 uint8格式 '''
valid_data = np.array(valid_data).astype('uint8')
```

模型執行分類預測處理，調用`classification_engine.py`內類別ClassificationEngine()

先將 ClassificationEngine() 類別進行物件實作，在類別 __init__() 有指定要求給予模型位置參數

```shell
def __init__(self, model_path, device_path=None)
```

在類別物件初始化實作時，只需要將模型位置給予`model_path`參數即可

```python
engine = ClassificationEngine(model)
```

在執行`engine.ClassifyWithASR_Feature(valid_data, top_k=3)`進行模型預測處理，此方法最後會回傳模型預測之結果。

`valid_data`參數是給予上方轉換資料型態為**uint8**之語音特徵矩陣，top_k=3 預設固定為3，是指在模型推斷預測出來結果矩陣中，先找出3個可信度最大結果出在下去做篩選，最後，回傳篩選後結果。

```python
get_score_list = engine.ClassifyWithASR_Feature(valid_data, top_k=3)
```

最後，從`engine.ClassifyWithASR_Feature()`方法回傳之結果中，取出結果矩陣內第一個位置數值，此為分類編號，在下去做分類標籤資料庫查詢編號所對應分類標籤，此為最後我們看到畫面之預測分類標籤名稱。


使用分類標籤資料庫為專案目錄內`DB/class.db3`

在`Config.py`內關於SQLite3配置
```python
SQLite_DB_DirectoryName = "DB" # 放置SQLite3 DB檔案目錄名稱
SQLite_name = "class.db3"  # SQLite3 DB檔案
db_TableName = 'audioclass'  # SQLite3 資料表名稱
column_ClassNum = 'ClassNum' # SQLite3 資料表欄位名稱
column_Classname = 'ClassName' # SQLite3 資料表欄位名稱
```

<br/>

程式執行結果

<script src="https://gist.github.com/s123600g/551ab08764b70132afb637f3ea76de16.js"></script>


---
