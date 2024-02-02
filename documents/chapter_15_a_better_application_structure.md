## Chapter 15: A Better Application Structure

> Posted by on [Miguel Grinberg](https://blog.miguelgrinberg.com/author/Miguel%20Grinberg)

這是 Flask 大型教學系列的第十五部分，在這裡我將使用適合更大型應用程式的風格來重構應用程式。

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

Microblog 已經是一個相當大的應用程式，所以我認為這是討論 Flask 應用程式如何在不變得混亂或太難管理的情況下成長的好機會。Flask 是一個旨在讓你以任何你想要的方式組織你的專案的框架，作為這種哲學的一部分，它使得當應用程式變得更大，或者你的需求或經驗水平變化時，改變或適應應用程式結構成為可能。

在這一章中，我將討論一些適用於大型應用程式的模式，並為了展示它們，我將對我的 Microblog 專案的結構進行一些改變，目的是使程式碼更容易維護和更好地組織。但當然，保持著真正的 Flask 精神，我鼓勵你將這些改變僅作為一個建議，當你試圖決定如何組織你自己的專案時。

這一章的 GitHub 連結是：[Browse](https://github.com/miguelgrinberg/microblog/tree/v0.15) [Zip](https://github.com/miguelgrinberg/microblog/archive/v0.15.zip) [Diff](https://github.com/miguelgrinberg/microblog/compare/v0.14...v0.15)


### 當前的限制
應用程式目前狀態存在兩個基本問題。如果你看看應用程式的結構，你會注意到有幾個不同的子系統可以被識別，但支持它們的程式碼都混雜在一起，沒有任何明確的界限。讓我們回顧一下這些子系統是什麼：

1. 使用者認證子系統，包括一些在 app/routes.py 中的視圖函式、一些在 app/forms.py 中的表單、一些在 app/templates 中的範本和在 app/email.py 中的電子郵件支持。
2. 錯誤子系統，定義了在 app/errors.py 中的錯誤處理器和在 app/templates 中的範本。
3. 核心應用程式功能，包括顯示和撰寫部落格帖子、使用者個人資料和追蹤，以及部落格帖子的即時翻譯，這些功能散布在大部分的應用程式模組和範本中。

考慮到我已經識別的這三個子系統以及它們的結構，你可能會注意到一個模式。到目前為止，我所遵循的組織邏輯是基於有不同應用程式功能的專用模組。有一個模組是為視圖函式，另一個是為網頁表單，一個

是錯誤，一個是電子郵件，一個目錄是 HTML 範本，等等。雖然這是一個對於小型專案有意義的結構，但一旦一個專案開始成長，它傾向於使這些模組變得非常大和混亂。

一個清楚看到問題的方式是考慮如何通過從這個專案中盡可能多地重用來開始第二個專案。例如，使用者認證部分應該在其他應用程式中運作良好，但如果你想要如實使用那些程式碼，你將不得不進入幾個模組，並將相關部分複製 / 貼上到新專案中的新檔案中。看看這有多不方便？如果這個專案將所有與認證相關的檔案與應用程式的其餘部分分開，不是更好嗎？Flask 的藍圖功能有助於實現更實用的組織，使程式碼重用變得更容易。

還有第二個問題不那麼明顯。Flask 應用程式實例是作為全局變數在 app/__init__.py 中建立的，然後由許多應用程式模組導入。雖然這本身並不是問題，但將應用程式作為全局變數可能會使某些情況變得複雜，尤其是與測試相關的情況。想像你想要在不同配置下測試這個應用程式。因為應用程式被定義為全局變數，真的沒有辦法實例化兩個使用不同配置變數的應用程式。另一個不理想的情況是所有測試都使用相同的應用程式，所以一個測試可能會對應用程式進行更改，從而影響稍後運行的另一個測試。理想情況下，你希望所有測試都在原始的應用程式實例上運行。

### 藍圖
在 Flask 中，藍圖是代表應用程式子集的邏輯結構。一個藍圖可以包括路由、視圖函式、表單、範本和靜態檔案等元素。如果你在一個單獨的 Python 套件中撰寫你的藍圖，那麼你就有了一個封裝與應用程式特定功能相關元素的元件。

藍圖的內容最初處於休眠狀態。為了啟動這些元素，需要將藍圖註冊到應用程式。在註冊期間，加入到藍圖的所有元素都被傳遞到應用程式。因此，你可以將藍圖視為暫時儲存應用程式功能的工具，有助於組織你的程式碼。

### 錯誤處理藍圖
我建立的第一個藍圖是封裝錯誤處理器支持的藍圖。這個藍圖的結構如下：

```
app/
    errors/                             <-- 藍圖套件
        __init__.py                     <-- 藍圖建立
        handlers.py                     <-- 錯誤處理器
    templates/
        errors/                         <-- 錯誤範本
            404.html
            500.html
    __init__.py                         <-- 藍圖註冊
```

本質上，我所做的是將 app/errors.py 模組移至 app/errors/handlers.py，

並將兩個錯誤範本移至 app/templates/errors，以便將它們與其他範本分離。我還必須更改兩個錯誤處理器中的 `render_template()` 調用，以使用新的錯誤範本子目錄。之後，我在 app/errors/__init__.py 模組中加入了藍圖建立，並在應用程式實例建立後，將藍圖註冊到 app/__init__.py。

我應該指出，Flask 藍圖可以配置為擁有單獨的範本或靜態檔案目錄。我決定將範本移到應用程式的範本目錄的子目錄中，這樣所有範本都在單一層級結構中，但如果你更喜歡將屬於藍圖的範本儲存在藍圖套件內，這也是支援的。例如，如果你在 Blueprint() 建構子中加入了 `template_folder='templates'` 參數，那麼你可以將藍圖的範本儲存在 app/errors/templates 中。

藍圖的建立與應用程式的建立相當相似。這在藍圖套件的 ___init__.py 模組中完成：

```python
app/errors/__init__.py: 錯誤藍圖。

from flask import Blueprint

bp = Blueprint('errors', __name__)

from app.errors import handlers
```

Blueprint 類別需要藍圖的名稱、基礎模組的名稱（通常設置為 `__name__`，就像 Flask 應用程式實例一樣），以及一些可選參數，在這種情況下我不需要。在建立藍圖物件後，我導入了 handlers.py 模組，以便將其中的錯誤處理器註冊到藍圖。這個導入放在底部是為了避免循環依賴。

在 handlers.py 模組中，我使用藍圖的 `@bp.app_errorhandler` 裝飾器來附加錯誤處理器，而不是使用應用程式的 `@app.errorhandler` 裝飾器。雖然這兩個裝飾器達到相同的最終結果，但目標是嘗試使藍圖獨立於應用程式，使其更具可移植性。我還需要修改兩個錯誤範本的路徑，以適應新的錯誤子目錄。

完成錯誤處理器重構的最後一步是將藍圖註冊到應用程式：

```python
app/__init__.py: 將錯誤藍圖註冊到應用程式。

app = Flask(__name__)

## ...

from app.errors import bp as errors_bp
app.register_blueprint(errors_bp)

## ...

from app import routes, models  ## <-- 從這個導入中移除 errors！
```

要註冊藍圖，使用 Flask 應用程式實例的 `register_blueprint()` 方法。當藍圖被註冊後，任何視圖函式、範本、靜態檔案、錯誤處理器等都會連接到應用程式。我將藍圖的導入放在 `app.register_blueprint()` 上方，以避免循環依賴問題。

** 身份驗證藍圖 **
將應用程式的身份驗證功能重構為藍圖的過程與錯誤處理器相當類似。以下是重構後藍圖的結構圖：

```
app/
    auth/                               <-- 藍圖套件
        __init__.py                     <-- 藍圖建立
        email.py                        <-- 身份驗證郵件
        forms.py                        <-- 身份驗證表單
        routes.py                       <-- 身份驗證路由
    templates/
        auth/                           <-- 藍圖範本
            login.html
            register.html
            reset_password_request.html
            reset_password.html
    __init__.py                         <-- 藍圖註冊
```

要建立這個藍圖，我必須將所有與身份驗證相關的功能移至藍圖中我建立的新模組。這包括一些視圖函式、網頁表單和支援函式，例如通過電子郵件發送密碼重設令牌的函式。我還將範本移至子目錄中，與應用程式的其餘部分分離，就像我對錯誤頁面所做的那樣。

在藍圖中定義路由時，使用 `@bp.route` 裝飾器，而不是 `@app.route`。還需要更改 `url_for()` 構建 URL 的語法。對於直接附加到應用程式的常規視圖函式，`url_for()` 的第一個參數是視圖函式名稱。當在藍圖中定義路由時，此參數必須包括藍圖名稱和視圖函式名稱，用句點分隔。例如，我必須將所有 `url_for('login')` 的出現替換為 `url_for('auth.login')`，其餘視圖函式也是如此。

要將 auth 藍圖與應用程式註冊，我使用了稍微不同的格式：

```python
app/__init__.py: 將身份驗證藍圖註冊到應用程式。

## ...
from app.auth import bp as auth_bp
app.register_blueprint(auth_bp, url_prefix='/auth')
## ...
```

這種情況下的 `register_blueprint()` 調用有一個額外的參數，`url_prefix`。這是完全可選的，但 Flask 給你選擇在 URL 前綴下附加藍圖的選項，因此藍圖中定義的任何路由都會在其 URL 中獲得此前綴。在許多情況下，

這作為一種「命名空間」很有用，它將藍圖中的所有路由與應用程式或其他藍圖中的其他路由分開。對於身份驗證，我認為讓所有路由以 `/auth` 開頭很好，所以我加入了前綴。所以現在登錄 URL 將是 http://localhost:5000/auth/login。因為我使用 `url_for()` 生成 URL，所以所有 URL 都將自動包含前綴。

### 主應用程式藍圖
第三個藍圖包含核心應用程式邏輯。重構這個藍圖的過程與前兩個藍圖相同。我給這個藍圖命名為 main，所以所有引用視圖函式的 `url_for()` 調用都必須加上 main. 前綴。鑑於這是應用程式的核心功能，我決定將範本保留在原來的位置。這不是問題，因為我已將其他兩個藍圖的範本移至子目錄中。

### 應用程式工廠模式
正如我在本章引言中提到的，將應用程式作為全域變數引入了一些複雜性，主要是對某些測試場景的限制。在我引入藍圖之前，應用程式必須是一個全域變數，因為所有視圖函式和錯誤處理器都需要用來自 app 的裝飾器進行裝飾，例如 `@app.route`。但現在所有路由和錯誤處理器都移至藍圖，將應用程式保持為全域的理由就少了很多。

因此，我將要做的是加入一個名為 `create_app()` 的函式，用於構建 Flask 應用程式實例，並消除全域變數。這次轉換並非微不足道，我必須解決一些複雜問題，但讓我們先看看應用程式工廠函式：

app/__init__.py: Application factory function.
```python
## ...
db = SQLAlchemy()
migrate = Migrate()
login = LoginManager()
login.login_view = 'auth.login'
login.login_message = _l('Please log in to access this page.')
mail = Mail()
moment = Moment()
babel = Babel()

def create_app(config_class=Config):
    app = Flask(__name__)
    app.config.from_object(config_class)

    db.init_app(app)
    migrate.init_app(app, db)
    login.init_app(app)
    mail.init_app(app)
    moment.init_app(app)
    babel.init_app(app)

    ## ... no changes to blueprint registration

    if not app.debug and not app.testing:
        ## ... no changes to logging setup

    return app
```

你已經看到，大多數 Flask 擴充套件是通過建立擴充套件的實例並將應用程式作為參數傳遞來初始化的。當應用程式不作為全域變數存在時，有一種替代模式，即擴充套件通過兩階段初始化。首先在全域範圍內建立擴充套件實例，如同以前一樣，但不傳遞任何參數。這建立了一個未附加到應用程式的擴充套件實例。在工廠函式中建立應用程式實例時，必須對擴充套件實例調用 `init_app()` 方法，以便將其綁定到現在已知的應用程式。

在初始化期間執行的其他任務保持不變，但移至工廠函式而不是在全域範圍內。這包括藍圖的註冊和日誌配置。請注意，我在決定是否啟用電子郵件和檔案日誌的條件中加入了一個 not app.testing 子句，以便在單元測試期間跳過所有這些日誌。當執行單元測試時，app.testing 標誌將為 True，因為在配置中設置了 TESTING 變數為 True。

那麼誰來調用應用程式工廠函式？使用這個函式的顯而易見的地方是頂層的 microblog.py 腳本，這是應用程式現在存在於全域範圍的唯一模組。另一個地方是在 tests.py 中，我將在下一節中更詳細地討論單元測試。

正如我上面提到的，隨著藍圖的引入，大多數對 app 的引用都消失了，但在程式碼中仍然有一些我必須處理的。例如，app/models.py、app/translate.py 和 app/main/routes.py 模組都有對 app.config 的引用。幸運的是，Flask 開發者試圖使視圖函式易於訪問應用程式實例，而不必像我到現在為止一直在做的那樣導入它。Flask 提供的 current_app 變數是一個特殊的「上下文」變數，Flask 在分發請求之前用應用程式初始化它。你之前已經看到了另一個上下文變數，即我用來儲存當前語境的 g 變數。這兩個，以及 Flask-Login 的 current_user 和一些你還沒見過的其他變數，是有些「魔法」的變數，它們像全域變數一樣工作，但只能在處理請求期間和處理它的線程中訪問。

用 Flask 的 current_app 變數替換 app 消除了將應用程式實例作為全域變數導入的需求。我能夠毫無困難地通過簡單的搜索和替換將所有對 app.config 的引用改為 current_app.config。

app/email.py 模組提出了一個稍微大一點的挑戰，所以我不得不使用一個小技巧：

```python
app/email.py: 將應用程式實例傳遞給另一個線程。

from flask import current_app

def send_async_email(app, msg):
    with app.app_context():
        mail.send(msg)

def send_email(subject, sender, recipients, text_body, html_body):
    msg = Message(subject, sender=sender, recipients=recipients)
    msg.body = text_body
    msg.html = html_body
    Thread(target=send_async_email,
           args=(current_app._get_current_object(), msg)).start()
```

在 send_email() 函式中，應用程式實例作為參數傳遞給一個背景線程，然後該線程將在不阻塞主應用程式的情況下發送電子郵件。在作為背景線程運行的 send_async_email() 函式中直接使用 current_app 是不行的，因為 current_app 是一個與處理客戶端請求的線程綁定的上下文感知變數。在不同的線程中，current_app 將沒有分配值。將 current_app 直接作為參數傳遞給線程對象也是不行的，因為 current_app 實際上是一個動態映射到應用程式實例的代理物件。因此，傳遞代理物件將與在線程中直接使用 current_app 相同。我需要做的是訪問存儲在代理物件內部的真實應用程式實例，並將其作為 app 參數傳遞。current_app._get_current_object() 表達式從代理物件內部提取實際的應用程式實例，所以這就是我傳遞給線程的參數。

另一個棘手的模組是 app/cli.py，它實現了一些用於管理語言翻譯的捷徑命令，並使用了 `@app.cli.group()` 裝飾器。在這種情況下，用 current_app 替換 app 是不行的，因為這些命令是在啟動時註冊的，而不是在處理請求期間，這是唯一可以使用 current_app 的時候。為了在這個模組中移除對 app 的引用，我建立了另一個藍圖：

```python
app/cli.py: 自定義應用程式命令的藍圖。

import os
from flask import Blueprint
import click

bp = Blueprint('cli', __name__, cli_group=None)


@bp.cli.group()
def translate():
    """翻譯和本地化命令。"""
    pass

@translate.command()
@click.argument('lang')
def init(lang):
    """初始化新語言。"""
    ## ...

@translate.command()
def update():
    """更新所有語言。"""
    ## ...

@translate.command()
def compile():
    """編譯所有語言。"""
    ## ...
```

Flask 預設將附加到藍圖的命令放在以藍圖名稱為名的分組下。這會導致這些命令可以透過 `flask cli translate ...` 的方式使用。為了避免額外的 cli 分組，藍圖中設定了 `cli_group=None`。

然後，我在應用程式工廠函式中註冊這個 cli 藍圖：

```python
app/__init__.py: 應用程式工廠函式。

## ...

def create_app(config_class=Config):
    ## ...
    from app.cli import bp as cli_bp
    app.register_blueprint(cli_bp)
    ## ...

    return app
```

### 單元測試的改進
正如我在本章開頭所暗示的，我到目前為止所做的工作目標是改善單元測試的工作流程。當你運行單元測試時，你希望確保應用程式以不干擾你的開發資源（例如你的資料庫）的方式進行配置。

目前版本的 tests.py 依靠在應用程式實例上應用配置之後修改配置的技巧，這是一種危險的做法，因為並非所有類型的更改在那麼晚的時候都有效。我想要的是在配置加入到應用程式之前有機會指定我的測試配置。

現在的 `create_app()` 函式接受一個配置類作為參數。預設情況下使用 config.py 中定義的 Config 類，但現在我可以通過向工廠函式傳遞一個新類來建立使用不同配置的應用程式實例。以下是一個適合用於我的單元測試的配置類範例：

```python
tests.py: 測試配置。

from config import Config

class TestConfig(Config):
    TESTING = True
    SQLALCHEMY_DATABASE_URI = 'sqlite://'
```

這裡我所做的是建立應用程式的 Config 類的子類，並覆寫 SQLAlchemy 配置以使用內存中的 SQLite 資料庫。我還加入了設置為 True 的 TESTING 屬性，這在應用程式需要確定它是否在單元測試下運行時很有用。

如果你還記得，我的單元測試依賴於 `setUp()` 和 `tearDown()` 方法，這些方法由單元測試框架自動調用，以建立和銷毀每個測試運行所需的環境。現在我可以使用這兩個方法來為每個測試建立和銷毀一個全新的應用程式：

```python
tests.py: 為每個測試建立一個應用程式。

class UserModelCase(unittest.TestCase):
    def setUp(self):
        self.app = create_app(TestConfig)
        self.app_context = self.app.app_context()
        self.app_context.push()
        db.create_all()

    def tearDown(self):
        db.session.remove()
        db.drop_all()
        self.app_context.pop()
```

新的應用程式將存儲在 `self.app

` 中，並且像以前一樣，從中建立了一個應用程式上下文，但這是什麼呢？

記得 `current_app` 變數嗎？當沒有全域應用程式可以導入時，它以某種方式作為應用程式的代理。這個變數知道應用程式實例，因為它查找在當前線程中推送的應用程式上下文。如果它找到一個，就從中獲得應用程式實例。如果沒有上下文，則無法知道哪個應用程式是活躍的，所以 `current_app` 會引發異常。以下是在 Python 控制台中這是如何工作的範例。這需要通過運行 python 啟動的控制台，因為 flask shell 命令為方便起見自動激活了一個應用程式上下文。

```python
>>> from flask import current_app
>>> current_app.config['SQLALCHEMY_DATABASE_URI']
Traceback (most recent call last):
    ...
RuntimeError: Working outside of application context.

>>> from app import create_app
>>> app = create_app()
>>> app.app_context().push()
>>> current_app.config['SQLALCHEMY_DATABASE_URI']
'sqlite:////home/miguel/microblog/app.db'
```

這就是秘密！在調用你的視圖函式之前，Flask 推送了一個應用程式上下文，從而使 `current_app` 和 `g` 活躍起來。當請求完成時，上下文被移除，以及這些變數。為了使 `setUp()` 方法中的 `db.create_all()` 調用有效，必須為應用程式實例推送一個應用程式上下文，這樣 `db.create_all()` 就可以使用 `current_app.config` 來知道資料庫的位置。然後在 `tearDown()` 方法中我彈出上下文，將一切重置為乾淨狀態。

你還應該知道，應用程式上下文是 Flask 使用的兩種上下文之一。還有一個是請求上下文，它更具體，因為它適用於一個請求。當一個請求上下文在處理請求之前被激活時，Flask 的 `request` 和 `session` 變數變得可用，以及 Flask-Login 的 `current_user`。

### 環境變數
正如你在建立這個應用程式時所見，有許多配置選項依賴於在啟動伺服器之前在你的環境中設置變數。這包括你的密鑰、電子郵件伺服器資訊、資料庫 URL 和 Microsoft Translator API 金鑰。你可能會同意這是不便的，因為每次你打開一個新的終端會話時，這些變數都需要再次設置。

依賴大量環境變數的應用程式通常將這些變數儲存在根應用程式目錄的 `.env` 檔案中。當應用程式啟動時，它會導入此檔案中的變數，這樣就無需手動設置所有這些變數。

有一個支援 .env 檔案的 Python 套件叫作 python-dotenv，它已經安裝了，因為我在 .flaskenv 檔案中使用了它。雖然 .env 和 .flaskenv 檔案相似，但 Flask 預期 Flask 自己的配置變數放在 .flaskenv 中，而應用程式配置變數（包括一些可能屬於敏感性質的變數）則放在 .env 中。.flaskenv 檔案可以加入到你的原始碼控制中，因為它不包含任何秘密或密碼。.env 檔案則不應加入原始碼控制，以確保你的秘密受到保護。

flask 命令會自動將定義在 .flaskenv 和 .env 檔案中的任何變數導入環境。對於 .flaskenv 檔案來說，這足夠了，因為它的內容僅在通過 flask 命令運行應用程式時需要。然而，.env 檔案還將在這個應用程式的生產部署中使用，這不會使用 flask 命令。出於這個原因，明確導入 .env 檔案的內容是個好主意。

由於 config.py 模組是我讀取所有環境變數的地方，我將在建立 Config 類之前導入 .env 檔案，以便在類構建時變數已經設置好：

```python
config.py: 導入帶有環境變數的 .env 檔案。

import os
from dotenv import load_dotenv

basedir = os.path.abspath(os.path.dirname(__file__))
load_dotenv(os.path.join(basedir, '.env'))

class Config(object):
    ## ...
```

所以現在你可以建立一個 .env 檔案，其中包含應用程式所需的所有環境變數。重要的是，不要將你的 .env 檔案加入原始碼控制。你不會希望包含密碼和其他敏感資訊的檔案被包含在你的原始碼庫中。

.env 檔案可用於所有配置時變數，但不能用於 Flask 的 FLASK_APP 和 FLASK_DEBUG 環境變數，因為這些在應用程式啟動過程的非常早期就需要了，早於應用程式實例及其配置物件的存在。

以下範例顯示了一個定義秘密鑰匙、配置郵件在本地運行的郵件伺服器上的 8025 端口且無需認證、設置 Microsoft Translator API 金鑰，並讓資料庫配置使用預設值的 `.env` 檔案：

```bash
SECRET_KEY=a-really-long-and-unique-key-that-nobody-knows
MAIL_SERVER=localhost
MAIL_PORT=8025
MS_TRANSLATOR_KEY=<your-translator-key-here>
```

### 需求檔案
此時，我已經在 Python 虛擬環境中安裝了相當多的套件。如果你需要在另一台機器上重新生成你的環境，你將會有記住安裝了哪些套件的困難，因此普遍接受的做法是在項目的根文件夾中寫一個 requirements.txt 檔案，列出所有依賴項及其版本。製作這個列表實際上很容易：

```bash
(venv) $ pip freeze > requirements.txt
```

pip freeze 命令會以 requirements.txt 檔案所需的正確格式將安裝在你虛擬環境中的所有套件傾印出來。現在，如果你需要在另一台機器上建立相同的虛擬環境，而不是一個一個安裝套件，你可以運行：

```bash
(venv) $ pip install -r requirements.txt
```

