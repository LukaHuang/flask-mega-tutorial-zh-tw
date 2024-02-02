## Chapter 1 : Hello, World

> Posted by on [Miguel Grinberg](https://blog.miguelgrinberg.com/author/Miguel%20Grinberg)

歡迎！你即將開始學習如何使用 Python 和 Flask 框架建立網頁應用程式。在這第一章中，你將學習如何設置一個 Flask 專案。在本章結束時，你將能夠讓一個簡單的 Flask 網頁應用程式在你的電腦上運行！

### 目錄

- [Chapter 1: Hello, World!](/python/flask-mega-tutorial/chapter_1_hello_world)
- [Chapter 2: Templates](/python/flask-mega-tutorial/chapter_2_templates)
- [Chapter 3: Web Forms](/python/flask-mega-tutorial/chapter_3_web_forms)
- [Chapter 4: Database](/python/flask-mega-tutorial/chapter_4_database)
- [Chapter 5: User Logins](/python/flask-mega-tutorial/chapter_5_user_logins)
- [Chapter 6: Profile Page and Avatars](/python/flask-mega-tutorial/chapter_6_profile_page_and_avatars)
- [Chapter 7: Error Handling](/python/flask-mega-tutorial/chapter_7_error_handling)
- [Chapter 8: Followers](/python/flask-mega-tutorial/chapter_8_followers)
- [Chapter 9: Pagination](/python/flask-mega-tutorial/chapter_9_pagination)
- [Chapter 10: Email Support](/python/flask-mega-tutorial/chapter_10_email_support)
- [Chapter 11: Facelift](/python/flask-mega-tutorial/chapter_11_facelift)
- [Chapter 12: Dates and Times](/python/flask-mega-tutorial/chapter_12_dates_and_times)
- [Chapter 13: I18n and L10n](/python/flask-mega-tutorial/chapter_13_i18n_and_l10n)
- [Chapter 14: Ajax](/python/flask-mega-tutorial/chapter_14_ajax)
- [Chapter 15: A Better Application Structure](/python/flask-mega-tutorial/chapter_15_a_better_application_structure)
- [Chapter 16: Full-Text Search](/python/flask-mega-tutorial/chapter_16_full_text_search)
- [Chapter 17: Deployment on Linux](/python/flask-mega-tutorial/chapter_17_deployment_on_linux)
- [Chapter 19: Deployment on Docker Containers](/python/flask-mega-tutorial/chapter_19_deployment_on_docker_containers)
- [Chapter 20: Some JavaScript Magic](/python/flask-mega-tutorial/chapter_20_some_javascript_magic)
- [Chapter 21: User Notifications](/python/flask-mega-tutorial/chapter_21_user_notifications)
- [Chapter 22: Background Jobs](/python/flask-mega-tutorial/chapter_22_background_jobs)
- [Chapter 23: Application Programming Interfaces （APIs）](/python/flask-mega-tutorial/chapter_23_application_programming_interfaces_apis)

> 你正在閱讀 Flask Mega-Tutorial 的 2024 年版本。完整的課程也可以在 [Amazon](https://amzn.to/3ahVnPN) 以電子書和平裝書的形式訂購。感謝你的支持！
> 如果你正在尋找 2018 年版本的課程，你可以在[這裡](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world-2018)找到它。

本書中展示的所有範例程式碼都托管在 GitHub 倉庫上。從 GitHub 下載程式碼可以節省你很多打字的時間，但我強烈建議你自己打出程式碼，至少對於前幾章來說。一旦你對 Flask 和範例應用程式更熟悉，如果打字變得太繁瑣，你可以直接從 GitHub 存取程式碼。

在每章的開始，我會給你三個 GitHub 連結，這些在你研讀章節時會很有用。瀏覽連結將打開 GitHub 倉庫中 Microblog 的位置，在這裡加入了你正在閱讀的章節中的變更，而不包括未來章節中引入的任何變更。壓縮檔連結是一個下載連結，包括整個應用程式直到本章節的變更。Diff 連結將打開一個圖形視圖，顯示你即將閱讀的章節中所做的所有變更。

這一章的 GitHub 連結有：瀏覽、壓縮檔、Diff。

### 安裝 Python
如果你的電腦還沒有安裝 Python，現在就去安裝吧。如果你的作業系統沒有提供 Python 套件，你可以從 Python 官方網站下載安裝程式。如果你正在使用 Microsoft Windows 並且有 WSL 或 Cygwin，請注意你將不會使用 Windows 本機版本的 Python，而是需要從 Ubuntu（如果你使用 WSL）或 Cygwin 獲得一個 UNIX 友好版本的 Python。

為了確保你的 Python 安裝是可用的，你可以打開一個終端機視窗並輸入 python3，或者如果那不行，就輸入 python。以下是你應該看到的內容：

```bash
$ python3
Python 3.12.0 (main, Oct  5 2023, 10:46:39) [GCC 11.4.0] on linux
輸入 "help", "copyright", "credits" 或 "license" 來獲取更多資訊。
>>> _
```

Python 直譯器現在正在互動提示符下等待，你可以在此輸入 Python 語句。在未來的章節中，你將學習這個互動提示符有哪些用途。但現在，你已經確認了你的系統上安裝了 Python。要退出互動提示符，你可以輸入 exit() 並按 Enter。在 Linux 和 Mac OS X 版本的 Python 上，你也可以通過按 Ctrl-D 來退出直譯器。在 Windows 上，退出快捷方式是 Ctrl-Z，然後按 Enter。

### 安裝 Flask
下一步是安裝 Flask，但在我講解之前，我想告訴你關於安裝 Python 套件的最佳實踐。

在 Python 中，像 Flask 這樣的套件可在公共倉庫中獲得，任何人都可以從中下載並安裝它們。官方的 Python 套件倉庫稱為 PyPI，代表 Python Package Index（有些人也將這個倉庫稱為 “乳酪店”）。從 PyPI 安裝套件非常簡單，因為 Python 附帶了一個叫做 pip 的工具來完成這項工作。

要在你的機器上安裝套件，你可以如下使用 pip：

```bash
$ pip install <套件名稱>
```

有趣的是，這種安裝套件的方法在大多數情況下都不會奏效。如果你的 Python 直譯器是為你電腦的所有使用者全局安裝的，那麼你的普通使用者帳戶可能沒有權限對其進行修改，所以上面命令的唯一方法是從管理員帳戶運行。但即使沒有這種複雜情況，想想當你以這種方式安裝套件時會發生什麼。pip 工具將從 PyPI 下載套件，然後將其加入到你的 Python 安裝中。從那時起，你系統上的每個 Python 指令稿都將能夠存取這個套件。想像一下這樣的情況：當你開始時，你已經使用 Flask 第 2 版完成了一個網頁應用程式，那是你開始時 Flask 的最新版本，但現在它已經

被第 3 版取代。現在你想開始第二個應用程式，你希望使用第 3 版，但如果你升級了你安裝的第 1 版，你可能會破壞你的舊應用程式。你看到問題了嗎？如果能夠有 Flask 第 2 版安裝並可供你的舊應用程式使用，同時也為你的新應用程式安裝 Flask 第 3 版，那將是理想的。

為了解決為不同應用程式維護不同版本的套件的問題，Python 使用了虛擬環境的概念。一個虛擬環境是 Python 直譯器的一個完整副本。當你在虛擬環境中安裝套件時，系統範圍的 Python 直譯器不會受到影響，只有副本會受到影響。因此，為每個應用程式使用不同的虛擬環境來完全自由地安裝任何版本的套件是解決方案。虛擬環境的另一個好處是它們是由建立它們的使用者擁有的，因此不需要管理員帳戶。

讓我們開始建立一個專案將生活的目錄。我將這個目錄稱為 microblog，因為這是應用程式的名稱：

```bash
$ mkdir microblog
$ cd microblog
```

最近版本的 Python 都內建了對虛擬環境的支援，所以你需要做的就是這樣來建立一個：

```bash
$ python3 -m venv venv
```

使用這個命令，我請求 Python 運行 venv 套件，該套件建立了一個名為 venv 的虛擬環境。命令中的第一個 venv 是 -m 選項的參數，這是 Python 虛擬環境套件的名稱，第二個是我要用於這個特定環境的虛擬環境名稱。如果你覺得這有點混淆，你可以將第二個 venv 替換為你想要指派給你的虛擬環境的不同名稱。一般來說，我在專案目錄中使用名為 venv 的名稱來建立我的虛擬環境，所以每當我進入一個專案時，我都會找到它對應的虛擬環境。

請注意，在某些作業系統中，你可能需要在上面的命令中使用 python 代替 python3。一些安裝使用 python 表示 2.x 版本的 Python，並使用 python3 表示 3.x 版本，而其他安裝則將 python 映射到 3.x 版本，根本沒有 python3 命令。

命令完成後，你將擁有一個名為 venv 的目錄，其中存儲了虛擬環境文件。

現在你必須告訴系統你想使用這個虛擬環境，你通過激活它來做到這一點。要激活你全新的虛擬環境，使用以下命令：

```bash
$ source venv/bin/activate
(venv) $ _
```

如果你使用的是 Microsoft Windows 命令提示字元視窗，激活命令略有不同：

```bash
$ venv\Scripts\activate
(venv) $ _
```

如果你在 Windows 上使用 PowerShell 而不是命令提示字元，那麼還有另一個激活命令你應該使用：

```bash
$ venv\Scripts\Activate.ps1
(venv) $ _
```

當你激活一個虛擬環境時，你的終端機會話配置會被修改，以便當你輸入 python 時，存儲在其中的 Python 直譯器是被調用的那一個。此外，終端機提示符也會被修改以包括已激活虛擬環境的名稱。對你的終端機會話所做的更改都是臨時的，並且是私有的，所以當你關閉終端機視窗時它們不會持續存在。如果你同時打開多個終端機視窗工作，那麼在每一個上激活不同的虛擬環境是完全可以的。

現在你已經建立並激活了一個虛擬環境，你終於可以在其中安裝 Flask 了：

```bash
(venv) $ pip install flask
```

如果你想確認你的虛擬環境現在已經安裝了 Flask，你可以啟動 Python 解釋器並導入 Flask：

```python
>>> import flask
>>> _
```
如果這個語句沒有產生任何錯誤，你可以慶祝一下，因為 Flask 已經安裝好並準備好被使用了。

請注意，上面的安裝指令並未指定你想安裝的 Flask 版本。當沒有指定版本時，預設會安裝套件庫中可用的最新版本。這份教學是為 Flask 的第 3 版設計的，但也應該適用於第 2 版。上面的指令會安裝最新的 3.x 版本，這對大多數使用者來說應該是合適的。如果你因為某些原因想要按照這份教學在 Flask 的 2.x 版本上操作，你可以使用以下指令來安裝最新的 1.x 版本：

```shell
(venv) $ pip install "flask<3" "werkzeug<3"
```
#### "Hello, World" Flask 應用程式

如果你前往 Flask 的快速入門頁面，你會看到一個非常簡單的範例應用程式，只有五行程式碼。我不打算重複那個簡單的範例，而是要展示一個稍微複雜一點的，這將為你寫更大型應用程式提供一個良好的基礎結構。

這個應用程式將存在於一個套件中。在 Python 中，包含 __init__.py 檔案的子目錄被認為是一個套件，並且可以被導入。當你導入一個套件時，__init__.py 會執行並定義套件對外界暴露的符號。

我們來建立一個叫做 app 的套件，用來承載這個應用程式。確保你在 microblog 目錄中，然後執行以下指令：

```shell
(venv) $ mkdir app
```
app 套件的 __init__.py 將包含以下程式碼：

app/__init__.py: Flask 應用程式實例

```python
from flask import Flask

app = Flask(__name__)

from app import routes
```
上面的腳本建立了一個應用程式物件，作為從 flask 套件導入的 Flask 類別的實例。傳遞給 Flask 類別的 __name__ 變數是 Python 預先定義的變數，設置為使用它的模組的名稱。Flask 使用這裡傳遞的模組位置作為起點，當它需要載入相關資源時，例如範本檔案，我將在第 2 章中介紹。出於所有實際目的，傳遞 __name__ 幾乎總是會以正確的方式配置 Flask。然後應用程式導入了尚不存在的 routes 模組。

一個起初可能會讓人困惑的方面是，有兩個叫做 app 的實體。app 套件是由 app 目錄和 __init__.py 腳本定義的，並在 from app import routes 語句中被引用。app 變數是在 __init__.py 腳本中定義為 Flask 類別的實例，這使它成為 app 套件的成員。

另一個特殊之處是在腳本底部而不是頂部導入 routes 模組，這通常是習慣。底部導入是一個眾所周知的解決方法，用來避免循環導入，這是 Flask 應用程式的常見問題。你將看到 routes 模組需要導入這個腳本中定義的 app 變數，所以將互相導入的其中一個放在底部可以避免因這兩個檔案之間的相互引用而導致的錯誤。

那麼 routes 模組中有什麼內容呢？路由處理應用程式支援的不同 URL。在 Flask 中，應用程式路由的處理器寫成 Python 函式，稱為視圖函式。視圖函式映射到一個或多個路由 URL，以便 Flask 知道當使用者端請求給定 URL 時該執行什麼邏輯。

這是該應用程式的第一個視圖函式，你需要在一個名為 app/routes.py 的新模組中寫入：

app/routes.py: 主頁路由

```python
from app import app

@app.route('/')
@app.route('/index')
def index():
    return "Hello, World!"
```
這個視圖函式實際上相當簡短，它只是返回一個問候作為字串。上面函式的兩個奇怪的 @app.route 行是裝飾器，Python 語言的一個獨特功能。裝飾器修改其後面的函式。裝飾器的一個常見模式是將它們用來註冊函式作為某些事件的回調。在這個案例中，@app.route 裝飾器在給定的 URL 和函式之間建立了關聯。在這個範例中有兩個裝飾器，將 URL / 和 /index 與此函式關聯。這意味著當網路瀏覽器請求這兩個 URL 中的任何一個時，Flask 將調用此函式，並將其返回值作為回應傳回瀏覽器。如果這還沒有完全理解，當你運行這個應用程式時很快就會明白。

要完成應用程式，你需要在頂層有一個 Python 腳本，定義 Flask 應用程式實例。我們將這個腳本稱為 microblog.py，並將其定義為一個導入應用程式實例的單行：

microblog.py: 主要應用程式模組

```python
from app import app
```
還記得兩個 app 實體嗎？在這裡你可以在同一句話中看到它們。Flask 應用程式實例被稱為 app，是 app 套件的成員。from app import app 語句導入了 app 套件的成員 app 變數。如果你覺得這很困惑，你可以將套件或變數之一重新命名為其他東西。

為了確保你正確地做了一切，以下你可以看到到目前為止項目結構的圖表：

microblog/
  venv/
  app/
    __init__.py
    routes.py
  microblog.py
不管你信不信，這個應用程式的第一版現在已經完成了！但是在運行它之前，需要通過設置 FLASK_APP 環境變數來告訴 Flask 如何導入它：

```shell
(venv) $ export FLASK_APP=microblog.py
```
如果你在使用 Microsoft Windows 命令提示字元，請在上述指令中使用 set 而不是 export。

你準備好被震撼了嗎？你可以通過輸入以下指令來運行你的第一個網路應用程式：

```shell
(venv) $ flask run
 * Serving Flask app 'microblog.py' (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```
這裡發生了什麼？flask run 指令將在 FLASK_APP 環境變數引用的模組中查找 Flask 應用程式實例，在這種情況下是 microblog.py。該指令設置了一個網路伺服器，配置為將請求轉發給這個應用程式。

伺服器初始化後將等待使用者端連接。flask run 的輸出表明伺服器正在 IP 地址 127.0.0.1 上運行，這總是你自己電腦的地址。這個地址如此常見，以至於也有一個更簡單的名稱你可能以前見過：localhost。網路伺服器監聽特定埠號上的連接。部署在生產網路伺服器上的應用程式通常在埠 443 監聽，或者如果它們不實施加密，有時是 80，但訪問這些埠需要管理員權限。由於這個應用程式在開發環境中運行，Flask 使用埠 5000。現在打開你的網路瀏覽器，並在地址欄中輸入以下 URL：

```
http://localhost:5000/
```
或者你可以使用這個其他 URL：

```
http://localhost:5000/index
```

你看到應用程式路由對應的執行了嗎？第一個 URL 對應到 /，而第二個對應到 /index。這兩個路由都關聯到應用程式中唯一的視圖函式，所以它們產生相同的輸出，這就是函式回傳的字串。如果你輸入其他的 URL，你會得到一個錯誤，因為只有這兩個 URL 被應用程式識別。

![](./img/2024-02-03-00-30-27.png)

當你玩完伺服器後，你可以按 Ctrl-C 停止它。

恭喜你，你已經完成成為網頁開發者的第一大步！

#### Flask 應用程式執行問題

你在執行 Flask 應用程式時遇到麻煩了嗎？在大多數電腦中，5000 端口是可用的，但你的電腦可能已經在運行一個使用這個端口的應用程式，在這種情況下，`flask run` 指令將因為「地址已被使用」或類似的錯誤而失敗。如果你使用 Macintosh 電腦，macOS 的某些版本會在這個端口上運行一個名為「Airplay Receiver」的服務。如果你無法找出如何移除使用端口 5000 的軟體，你可以嘗試在不同端口上執行 Flask。例如，以下是如何在端口 5001 上啟動伺服器：

```bash
(venv) $ flask run --port 5001
```

#### 環境變數的註冊

在我結束這章之前，我要再向你展示一件事。由於環境變數在終端會話之間不會被記住，當你開啟一個新的終端視窗來處理你的 Flask 應用程式時，你可能會發現每次都要設定 `FLASK_APP` 環境變數很麻煩。但幸運的是，Flask 允許你註冊你希望在執行 `flask` 指令時自動使用的環境變數。要使用這個選項，你必須安裝 python-dotenv 套件：

```bash
(venv) $ pip install python-dotenv
```

現在你只需在項目頂層目錄中名為 .flaskenv 的檔案中寫下環境變數名稱和值：

.flaskenv: flask 指令的環境變數

```bash
FLASK_APP=microblog.py
```

`flask` 指令會尋找 .flaskenv 檔案並導入其中定義的所有變數，就如同它們是在環境中定義的一樣。
