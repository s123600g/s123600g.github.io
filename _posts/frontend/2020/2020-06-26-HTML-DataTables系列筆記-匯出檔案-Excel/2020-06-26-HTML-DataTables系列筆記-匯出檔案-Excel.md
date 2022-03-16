---
layout: Article/content
title:  "HTML-DataTables系列筆記-匯出檔案-Excel"
date:   2020-06-26 20:56:00 +0800
categories: [frontend,datatables]
tag: frontend
img_category_num: frontend_02
excerpt_separator: <!--more-->
---

<!--more-->

在下載Datatables 套件包時，在`Extensions`內部**Buttons**項目記得要勾選
 - [x] HTML5 export

這樣才會把必要模組功能一起包進套件包裡。

關鍵參數設置
Datatables 主體渲染樣式設置
```javascript
dom: 'Bfrtip'
```

最重要的是`B`這個值，要開啟Datatables Button 功能模組。

```javascript
buttons: [ ]
```
這裡面可以定義Button功能模組內有哪些，例如在官方說明文件可以看到有這些
```
'copy', 'csv', 'excel', 'pdf', 'print'
```
* copy --> 複製整個Datatables內容。
* csv --> 將整個Datatables內容匯出成csv檔案下載。
* excel --> 將整個Datatables內容匯出成excel檔案下載。
* pdf --> 將整個Datatables內容匯出成pdf檔案下載。
* print --> 將整個Datatables內容進行列印。


不使用官方預設配置，改為以下設置下載Excel為例
```javascript
buttons: [
    {
        extend: 'excelHtml5',
        title: "MyDatatables_Excel",
        text: "匯出Excel",
        customize: function (xlsx) {
        }
    }
],
```

* extend --> 此按鈕繼承自哪一個模組，在這裡要繼承`'excelHtml5'`，此為模組名稱。
* title --> 檔案名稱與標題都跟此參數有關，如果沒設置的話會跑預設Datatables自己定義名稱。
* text --> 按鈕顯示名稱。
* customize --> 如果要針對內部資料內容做設置，在此進行資料處理。

預設匯出下載的內容整個第一列(A)會是一個跨欄置中的標題列，內部標題依據`title`設定的值。

---

#### 相關參考

* [DOM](https://datatables.net/reference/option/dom)
* [Buttons](https://datatables.net/reference/button/)
* [File name](https://datatables.net/extensions/buttons/examples/html5/filename.html)
* [File export](https://datatables.net/extensions/buttons/examples/initialisation/export.html)
* [excel](https://datatables.net/reference/button/excel)


