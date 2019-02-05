編譯器支持使用環境變量配置如何監視文件和目錄的變化。

## 使用`TSC_WATCHFILE`環境變量來配置文件監視

選項                                           | 描述
-----------------------------------------------|----------------------------------------------------------------------
`PriorityPollingInterval`                      | 使用`fs.watchFile`但針對源碼文件，配置文件和消失的文件使用不同的輪詢間隔
`DynamicPriorityPolling`                       | 使用動態隊列，對經常被修改的文件使用較短的輪詢間隔，對未修改的文件使用較長的輪詢間隔
`UseFsEvents`                                  | 使用 `fs.watch`，它使用文件系統事件（但在不同的系統上可能不一定準確）來查詢文件的修改/創建/刪除。注意少數的系統如Linux，對監視者的數量有限制，如果使用`fs.watch`創建監視失敗那麼將通過`fs.watchFile`來創建監視
`UseFsEventsWithFallbackDynamicPolling`        | 此選項與`UseFsEvents`類似，只不過當使用`fs.watch`創建監視失敗後，回退到使用動態輪詢隊列進行監視（如`DynamicPriorityPolling`介紹的那樣）
`UseFsEventsOnParentDirectory`                 | 此選項通過`fs.watch`（使用系統文件事件）監視文件的父目錄，因此CPU佔用率低但也會降低精度
默認 （無指定值）                               | 如果環境變量`TSC_NONPOLLING_WATCHER`設置為`true`，監視文件的父目錄（如同`UseFsEventsOnParentDirectory`）。否則，使用`fs.watchFile`監視文件，超時時間為`250ms`。

## 使用`TSC_WATCHDIRECTORY`環境變量來配置目錄監視

在那些Nodejs原生就不支持遞歸監視目錄的平台上，我們會根據`TSC_WATCHDIRECTORY`的不同選項遞歸地創建對子目錄的監視。 注意在那些原生就支持遞歸監視目錄的平台上（如Windows），這個環境變量會被忽略。

選項                                           | 描述
-----------------------------------------------|----------------------------------------------------------------------
`RecursiveDirectoryUsingFsWatchFile`           | 使用`fs.watchFile`監視目錄和子目錄，它是一個輪詢監視（消耗CPU週期）
`RecursiveDirectoryUsingDynamicPriorityPolling`| 使用動態輪詢隊列來獲取目錄與其子目錄的改變
默認 （無指定值）                               | 使用`fs.watch`來監視目錄及其子目錄

## 背景

在編譯器中`--watch`的實現依賴於Nodejs提供的`fs.watch`和`fs.watchFile`，兩者各有優缺點。

`fs.watch`使用文件系統事件通知文件及目錄的變化。
但是它依賴於操作系統，且事件通知並不完全可靠，在很多操作系統上的行為難以預料。
還可能會有創建監視個數的限制，如Linux系統，在包含大量文件的程序中監視器個數很快被耗盡。
但也正是因為它使用文件系統事件，不需要佔用過多的CPU週期。
典型地，編譯器使用`fs.watch`來監視目錄（比如配置文件裡聲明的源碼目錄，無法進行模塊解析的目錄）。
這樣就可以處理改動通知不準確的問題。
但遞歸地監視僅在Windows和OSX系統上支持。
這就意味著在其它系統上要使用替代方案。

`fs.watchFile`使用輪詢，因此涉及到CPU週期。
但是這是最可靠的獲取文件/目錄狀態的機制。
典型地，編譯器使用`fs.watchFile`監視源文件，配置文件和消失的文件（失去文件引用），這意味著對CPU的使用依賴於程序裡文件的數量。
