---
layout: Article/content
title: "使用OpenXml產生Excel筆記"
date: 2022-10-10 18:57:00 +0800
categories: [dotnet_core]
tag: dotnet_core
img_category_num: dotnet_core_02
excerpt_separator: <!--more-->
---

<!--more-->

使用微軟官方提供[Open XML SDK](https://github.com/OfficeDev/Open-XML-SDK)來產生Excel檔案，此篇筆記主要紀錄簡易產生操作。

---

## 簡易範例

專案範例：[jyu.demo.OpenXMLCreateExcel](https://github.com/s123600g/jyu.demo.OpenXMLCreateExcel)

NuGet套件需要安裝 [DocumentFormat.OpenXml](https://www.nuget.org/packages/DocumentFormat.OpenXml/)

簡易範例資料設置在`src/GenExcelFile/Program.cs`內`GetExcelContentSample()`。

範例資料皆是以List集合為結構，每位置代表每列內容。

---

## 簡易理解概念

一個Excel產生操作步驟如下

### Step 1. 建置一個內容實體與新活頁簿(WorkBook、Sheets)

透過`SpreadsheetDocument`來建立一個實體，在程式宣告會如下

```csharp
MemoryStream memoryStream = new MemoryStream();

using (
    SpreadsheetDocument spreadsheetDocument = SpreadsheetDocument.Create(
                    memoryStream,
                    SpreadsheetDocumentType.Workbook
    )
)
{
    # Do Something .....   
}
```

所有的操作都會應在此範圍內，當還未確認內容完成時，通常先放在記憶體內，透過`MemoryStream`來達成記憶體暫存。

當有了實體不代表就有活頁簿，就像你開一個全新Excel時，不會看到一個空白活頁簿，需要使用者手動建置新空白活頁簿，所以，透過`WorkbookPart`來建置一個空白活頁簿，在程式宣告會如下

```csharp
WorkbookPart workbookPart = spreadsheetDocument.AddWorkbookPart();
workbookPart.Workbook = new Workbook();
```

有了活頁簿之後，我們還需要一個放置工作表單集合實體，這是用來存放每個新增的工作表單儲存區塊，所以，透過`Sheets`來建置集合實體，在程式宣告會如下

```csharp
Sheets sheets = spreadsheetDocument.WorkbookPart.Workbook.AppendChild<Sheets>(
       new Sheets()
);
```

### Step 2. 工作表單建置與實作內容(WorksheetPart、SheetData)

當需要建置一個獨立工作表單實體時，透過`WorksheetPart`來建置獨立實體，在程式宣告會如下

```csharp
WorksheetPart worksheetPart = workbookPart.AddNewPart<WorksheetPart>();
```

當工作表單建好實體後，初始化實體內容才能進行資料操作，透過`SheetData`來初始化實體內容，在程式宣告會如下

```csharp
SheetData sheetData = new SheetData();
```

工作表單內容結構其實就是列表資料概念，所以，操作上會是先建置一列，針對建置好的一列補上欄位內容，簡單來看就是一個List集合概念。

透過`Row`來初始化列，在程式宣告會如下
```csharp
Row sheetRow = new Row();
```

每列都是一個集合，所以，可以一直依序新增每個欄位資料，在程式操作如下

```csharp
sheetRow.Append(cells);
```

- `cells` 可以是一個List集合，也就是可以先把欄位資料先集合起來，一次送進來，也可以單獨一個資料送進來。


透過`Cell`來初始化欄位，在程式宣告會如下
```csharp
Cell tempCell = new Cell();
```

設置欄位內容，在程式操作會如下
```csharp
tempCell.CellValue = new CellValue(cellValue);
```

設置欄位資料類型，在程式操作會如下
```csharp
tempCell.DataType = CellValues.String;
```
- `CellValues` 這是一個Enum，有其他類型的屬性，並非只有`String`。

當一列內容建置好後，我們需要將它更新進工作表單內容(`SheetData`)內，在程式操作會如下
```csharp
sheetData.AppendChild(sheetRow);
```

此時還尚未完成一個工作表單建置，當確認完成工作表單內容後，還需更新進工作表單實體(`WorksheetPart`)，在程式操作會如下
```csharp
worksheetPart.Worksheet = new Worksheet(sheetData);
```

最後，再將此工作表單實體(`WorksheetPart`)註冊進工作表單儲存區塊(`Sheets`)，這樣才是一個工作表單建立操作步驟，在程式操作會如下
```csharp
sheets.Append(new Sheet()
{
     Id = spreadsheetDocument.WorkbookPart.GetIdOfPart(worksheetPart), 
     SheetId = 1,
     Name = "Sheet1"
});
```

從以上操作內容敘述，簡單來看操作流程如下
1. 建立`Worksheet`。
2. 建立`SheetData`。
3. 建立`Row`與`Cell`產生每列內容。
4. 建置好每列內容更新進`SheetData`。
5. 將`SheetData`更新進`Worksheet`。
6. 將`Worksheet`新增進`Sheets`完成一個工作表單建置。

當有多個工作表單要建置時，依序重複上面六個操作。

可以在範例`lib/ExcelGenerator/ExcelGenerator.cs`內參考整體流程。

<script src="https://gist.github.com/s123600g/6a03a5e303eab4b62800d7f307c3120d.js"></script>

---

#### 相關參考

1. [ASP.NET Core 教學 - Open XML SDK 匯出 Excel](https://blog.johnwu.cc/article/asp-net-core-export-to-excel.html)
2. [Create a spreadsheet document by providing a file name (Open XML SDK)](https://learn.microsoft.com/en-us/office/open-xml/how-to-create-a-spreadsheet-document-by-providing-a-file-name#sample-code)
3. [OpenXML Multiple Sheets](https://stackoverflow.com/a/22230453)

