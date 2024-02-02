## Chapter 23: Application Programming Interfaces (APIs)

> Posted by on [Miguel Grinberg](https://blog.miguelgrinberg.com/author/Miguel%20Grinberg)

這篇文章是 Flask Mega-Tutorial 系列的第二十三部分，也是最後一部分，我將告訴你如何透過擴展微部落格加入一個應用程式介面 (API)，客戶端可以用來以更直接的方式比傳統的網頁瀏覽器工作流程來與應用程式進行互動。

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

到目前為止，我為這個應用程式建立的所有功能都是為了一種特定類型的客戶端：網頁瀏覽器。但是其他類型的客戶端呢？例如，如果我想要為 Android 或 iOS 應用程式建立一個簡單的應用程式，只有一個填滿整個螢幕的網頁視圖組件，載入微部落格網站，但這比在設備的網頁瀏覽器中打開應用程式提供的好處要少。一個更好的解決方案（儘管更勞累）是建立一個原生應用程式，但這個應用程式如何與只返回 HTML 頁面的伺服器互動呢？

這是應用程式介面 (API) 可以幫助解決的問題領域。API 是一系列的 HTTP 路由，設計為應用程式的低層次入口點。API 允許客戶端直接與應用程式的資源進行工作，而不是定義路由和視圖函式返回 HTML 供網頁瀏覽器使用，將如何向使用者展示資訊的決定完全留給客戶端。例如，微部落格中的一個 API 可以提供使用者和部落格文章資訊給客戶端，並且也允許使用者編輯現有的部落格文章，但只是在資料層面，而不將此邏輯與 HTML 混合。

如果你研究目前在應用程式中定義的所有路由，你會注意到有一些可能符合我上述使用的 API 定義。你找到它們了嗎？我在談論一些返回 JSON 的路由，例如在第 14 章定義的 / translate 路由。這是一個路由，它接收一個文本、源語言和目標語言，所有這些都在 POST 請求中以 JSON 格式給出。對這個請求的回應是該文本的翻譯，也是以 JSON 格式。伺服器只返回請求的資訊，將向使用者展示這些資訊的責任留給客戶端。

雖然應用程式中的 JSON 路由具有 API 的 "感覺"，但它們被設計來支援在瀏覽器中運行的網頁應用程式。考慮到如果智能手機應用程式想要使用這些路由，它們將無法做到，因為它們要求一個已登入的使用者，而登入只能通過 HTML 表單進行。在這一章中，我將展示如何建立不依賴於網頁瀏覽器的 API，並且不

對連接到它們的客戶端類型做出任何假設。

本章的 GitHub 連結為：[Browse](https://github.com/miguelgrinberg/microblog/tree/v0.23)、[Zip](https://github.com/miguelgrinberg/microblog/archive/v0.23.zip)、[Diff](https://github.com/miguelgrinberg/microblog/compare/v0.22...v0.23)

### REST 作為 API 設計的基礎

有些人可能強烈不同意我上面的說法，即 / translate 和其他 JSON 路由是 API 路由。其他人可能同意，但會加上一個免責聲明，認為它們是設計不良的 API。那麼，什麼是設計良好的 API 的特點，為什麼 JSON 路由不在該類別中呢？

你可能聽說過 REST API 這個術語。REST，代表表現性狀態轉移，是 Roy Fielding 博士在他的博士論文中提出的一種架構。在他的工作中，Fielding 博士以相當抽象和通用的方式提出了 REST 的六個定義特徵。

除了 Fielding 博士的論文外，沒有 REST 的官方規範，這留下了很多細節供讀者解釋。關於一個給定的 API 是否符合 REST 的話題，經常是 REST「純粹主義者」之間的激烈辯論的源頭，他們相信一個 REST API 必須觀察所有六個特徵，並以非常特定的方式這樣做，與 REST「實用主義者」形成對比，後者將 Fielding 博士在他的論文中提出的想法作為指導或建議。Fielding 博士本人站在純粹主義者一邊，並在博客文章和網上評論中提供了對他的願景的一些額外見解。

目前實施的絕大多數 API 都遵循「實用」的 REST 實現。這包括來自「大玩家」的大多數 API，如 Facebook、GitHub、Twitter 等。很少有公開 API 被一致認為是純 REST 的，因為大多數 API 缺少純粹主義者認為必須擁有的某些實現細節。儘管 Fielding 博士和其他 REST 純粹主義者對什麼是或不是 REST API 有嚴格的觀點，但在軟體行業中，將 REST 以實用的意義引用是常見的。

為了給你一個 REST 論文內容的想法，以下部分描述了 Fielding 博士列舉的六個原則。


#### Client-Server 客戶端 - 伺服器

客戶端 - 伺服器原則相當直接，它簡單地指出在 REST API 中，客戶端和伺服器的角色應該被清楚地區分開來。在實踐中，這意味著客戶端和伺服器處於分開的程式中，透過一種傳輸進行通訊，這種傳輸在大多數情況下是透過 TCP 網路的 HTTP 協定。

#### Layered System 分層系統

分層系統原則說明，當客戶端需要與伺服器通訊時，它可能最終連接到一個中介，而不是實際的伺服器。這個想法是，對於客戶端來說，如果不是直接連接到伺服器，它發送請求的方式應該完全沒有差別，事實上，它甚至可能不知道自己是否連接到目標伺服器。同樣，這個原則指出，伺服器可能從一個中介而不是直接從客戶端接收客戶端請求，因此它絕不能假設連接的另一端是客戶端。

這是 REST 的一個重要特性，因為能夠加入中介節點允許應用程式架構師設計大型和複雜的網絡，這些網絡能夠透過使用負載平衡器、快取、代理伺服器等滿足大量的請求。

#### Cache 快取

這個原則通過明確指出允許伺服器或中介快取經常接收到的請求響應來擴展分層系統，以改善系統性能。你可能熟悉的一種快取實現是所有網頁瀏覽器中的快取。網頁瀏覽器的快取層經常被用來避免反覆請求相同的文件，如圖像等。

對於 API 的目的，目標伺服器需要透過使用快取控制來指示一個響應是否可以被中介在返回客戶端時快取。注意，因為安全原因，部署到生產環境的 API 必須使用加密，除非這個節點終止 SSL 連接或執行解密和重新加密，通常不在中介節點進行快取。

### Code On Demand

這是一個可選要求，指出伺服器可以在對客戶端的響應中提供可執行程式碼。因為這個原則要求伺服器和客戶端之間就客戶端能夠執行哪種可執行程式碼達成協議，這在 API 中很少使用。你可能會認為伺服器可以返回 JavaScript 程式碼供網頁瀏覽器客戶端執行，但 REST 並不是專門針對網頁瀏覽器客戶端的。例如，執行 JavaScript 可能會引入一個複雜性，如果客戶端是 iOS 或 Android 設備的話。

### Stateless 無狀態

無狀態原則是 REST 純粹主義者和實用主義者之間大多數辯論的中心之一。它指出 REST API 不應該保存任何客戶端狀態以供每次給定客戶端發送請求時回調。這意味著，網頁開發中常見的用於 “記住” 使用者隨著他們通過應用程式的頁面導航的機制不能被使用。在一個無狀態的 API 中，每個請求都需要包含伺服器需要的資訊來識別和認證客戶端並執行請求。這也意味著伺服器不能在資料庫或其他形式的存儲中存儲與客戶端連接相關的任何資料。

如果你想知道為什麼 REST 要求無狀態伺服器，主要原因是無狀態伺服器非常容易擴展，你需要做的就是在負載平衡器後面運行伺服器的多個實例。如果伺服器存儲客戶端狀態，事情就會變得更複雜，因為你必須弄清楚多個伺服器如何訪問和更新該狀態，或者另外確保給定客戶端總是由同一伺服器處理，這通常被稱為粘性會話。

如果你再次考慮章節引言中討論的 `/translate` 路由，你會意識到它不能被認為是 RESTful，因為與該路由相關的視圖函式依賴於 Flask-Login 的 `@login_required` 裝飾器，該裝飾器反過來將使用者的登入狀態存儲在 Flask 使用者會話中。

### Uniform Interface 統一介面

最後，最重要，最有爭議，也是最模糊記錄的 REST 原則是統一介面。Fielding 博士列舉了 REST 統一介面的四個區別方面：唯一資源標識符、資源表示、自描述消息和超媒體。

唯一資源標識符通過為每個資源分配一個唯一的 URL 來實現。例如，與給定使用者相關聯的 URL 可以是 `/api/users/<user-id>`，其中 `<user-id>` 是在資料庫表的主鍵中分配給使用者的標識符。這在大多數 API 中實施得相當好。

使用資源表示意味著當伺服器和客戶端交換關於資源的資訊時，它們必須使用一個雙方都同意的格式。對於大多數現代 API 來說，使用 JSON 格式來構建資源表示。一個 API 可以選擇支援多種資源表示格式，在這種情況下，HTTP 協定中的內容協商選項是客戶端和伺服器可以同意雙方都喜歡的格式的機制。

自描述消息意味著客戶端和伺服器之間交換的請求和響應必須包含另一方需要的所有資訊。作為一個典型的例子，HTTP 請求方法被用來指示客戶端希望伺服器執行什麼操作。GET 請求表明客戶端想要檢索有關資源的資訊，POST 請求表明客戶端想要建立一個新資源，PUT 或 PATCH 請求定義對現有資源的修改，而 DELETE 表明請求移除一個資源。目標資源作為請求 URL 指出，額外資訊在 HTTP 頭部、URL 的查詢字符串部分或請求體中提供。

超媒體要求是這套中最有爭議的，也是很少有 API 實施的，那些實施它的 API 很少以滿足 REST 純粹主義者的方式這樣做。由於應用程式中的資源都是相互關聯的，這個要求要求在資源表示中包含這些關係，以便客戶端可以通過遍歷關係發現新資源，幾乎就像你通過點擊連結從一個頁面跳轉到另一個頁面在網頁應用程式中發現新頁面一樣。這個想法是客戶端可以在沒有關於其中資源的任何先前知識的情況下進入一個 API，並僅僅通過跟隨超媒體連結來了解它們。實施這個要求的一個方面是，與 HTML 和 XML 不同，常用於 API 中的資源表示的 JSON 格式沒有定義一種標準方式來包含連結，所以你被迫使用一個自定義結構，或者嘗試解決這個差距的一個建議的 JSON 擴展，如 JSON-API、HAL、JSON-LD 或類似的。


### 實現 API 藍圖
為了讓你體驗開發 API 所涉及的內容，我將向微部落格加入一個 API。這不會是一個完整的 API，我將實現所有與使用者相關的功能，將其他資源如部落格文章的實現留給讀者作為練習。

為了保持組織性，並遵循我在第 15 章中描述的結構，我將建立一個新的藍圖，其中將包含所有 API 路由。所以讓我們開始建立這個藍圖將要存放的目錄：

```bash
(venv) $ mkdir app/api
```

藍圖的 `__init__.py` 檔案建立了藍圖對象，類似於應用中的其他藍圖：

```python
## app/api/__init__.py: API 藍圖構造器。

from flask import Blueprint

bp = Blueprint('api', __name__)

from app.api import users, errors, tokens
```

你可能記得有時需要將導入移至底部以避免循環依賴錯誤。這就是為什麼在建立藍圖後導入 `app/api/users.py`、`app/api/errors.py` 和 `app/api/tokens.py` 模塊（我還未寫）的原因。

API 的核心將存儲在 `app/api/users.py` 模塊中。下表總結了我將要實現的路由：

| HTTP 方法 | 資源 URL                | 說明            |
|----------|----------------------|----------------|
| GET      | `/api/users/<id>`    | 返回一個使用者。    |
| GET      | `/api/users`         | 返回所有使用者的集合。 |
| GET      | `/api/users/<id>/followers` | 返回這個使用者的追隨者。 |
| GET      | `/api/users/<id>/following` | 返回這個使用者正在追隨的使用者。 |
| POST     | `/api/users`         | 註冊一個新使用者帳號。 |
| PUT      | `/api/users/<id>`    | 修改一個使用者。    |

目前，我將建立一個帶有所有這些路由占位符的骨架模塊：

```python
## app/api/users.py: 使用者 API 資源占位符。

from app.api import bp

@bp.route('/users/<int:id>', methods=['GET'])
def get_user(id):
    pass

@bp.route('/users', methods=['GET'])
def get_users():
    pass

@bp.route('/users/<int:id>/followers', methods=['GET'])
def get_followers(id):
    pass

@bp.route('/users/<int:id>/following', methods=['GET'])
def get_following(id):
    pass

@bp.route('/users', methods=['POST'])
def create_user():
    pass

@bp.route('/users/<int:id>', methods=['PUT'])
def update_user(id):
    pass
```

`app/api/errors.py` 模塊將定義一些處理錯誤響應的輔助函式。但現在，我也將使用一個稍後會填充的占位符：

```python
## app/api/errors.py: 錯誤處理占位符。

def bad_request():
    pass
```

`app/api/tokens.py` 是將要定義身份驗證子系統的模塊。這將為非網頁瀏覽器客戶端提供一種登錄的替代方式。目前，我也將為這個模塊寫一個占位符：

```python


## app/api/tokens.py: 令牌處理占位符。

def get_token():
    pass

def revoke_token():
    pass
```

新的 API 藍圖需要在應用工廠函式中註冊：

```python
## app/__init__.py: 在應用中註冊 API 藍圖。

## ...

def create_app(config_class=Config):
    app = Flask(__name__)

    ## ...

    from app.api import bp as api_bp
    app.register_blueprint(api_bp, url_prefix='/api')

    ## ...
```

### 將使用者表示為 JSON 對象
實現 API 時要考慮的第一個方面是決定其資源的表示方式。我將實現一個與使用者工作的 API，所以我需要決定我的使用者資源的表示。經過一番頭腦風暴，我想出了以下 JSON 表示：

```json
{
    "id": 123,
    "username": "susan",
    "password": "my-password",
    "email": "susan@example.com",
    "last_seen": "2021-06-20T15:04:27+00:00",
    "about_me": "Hello, my name is Susan!",
    "post_count": 7,
    "follower_count": 35,
    "following_count": 21,
    "_links": {
        "self": "/api/users/123",
        "followers": "/api/users/123/followers",
        "following": "/api/users/123/following",
        "avatar": "https://www.gravatar.com/avatar/..."
    }
}
```

許多字段直接來自使用者資料庫模型。`password` 字段特殊之處在於它僅在註冊新使用者時使用。正如你從第 5 章記得的那樣，使用者密碼不存儲在資料庫中，只存儲一個雜湊，所以密碼永遠不會被返回。`email` 字段也被特殊對待，因為我不想暴露使用者的電子郵件地址。`email` 字段僅在使用者請求他們自己的條目時返回，但在他們檢索其他使用者的條目時不返回。`post_count`、`follower_count` 和 `following_count` 字段是 “虛擬” 字段，它們不作為資料庫中的字段存在，但為了方便提供給客戶端。這是一個很好的例子，展示了資源表示不需要與伺服器中實際資源的定義相匹配。

注意 `_links` 部分，它實現了超媒體要求。定義的連結包括指向當前資源的連結、跟隨此使用者的使用者列表、使用者跟隨的使用者列表，以及使用者的頭像圖片的連結。將來，如果我決定向這個 API 加入文章，這裡也應該包括指向使用者文章列表的連結。

關於 JSON 格式的一件好事是它總是轉換為 Python 字典或列表的表示。Python 標準庫的 json 包負責將 Python 數據結構與 JSON 進行來回轉換。所以為了生成這些表示，我將向 User 模型加入一個名為 `to_dict()` 的方法，它返回一個 Python 字典：

```python
## app/models.py: 使用者模型到表示。

from flask import url_for
## ...

class User(UserMixin, db.Model):
    ## ...

    def posts_count(self):
        query = sa.select(sa.func.count()).select

_from(
            self.posts.select().subquery())
        return db.session.scalar(query)

    def to_dict(self, include_email=False):
        data = {
            'id': self.id,
            'username': self.username,
            'last_seen': self.last_seen.replace(
                tzinfo=timezone.utc).isoformat(),
            'about_me': self.about_me,
            'post_count': self.posts_count(),
            'follower_count': self.followers_count(),
            'following_count': self.following_count(),
            '_links': {
                'self': url_for('api.get_user', id=self.id),
                'followers': url_for('api.get_followers', id=self.id),
                'following': url_for('api.get_following', id=self.id),
                'avatar': self.avatar(128)
            }
        }
        if include_email:
            data['email'] = self.email
        return data
```

這個方法應該大致是不言自明的。我定下的使用者表示字典簡單地生成並返回。為了計算文章、追隨者和跟隨的數量，我使用輔助方法，為文章數量加入了一個方法，這是我之前從未需要使用過的。正如我上面提到的，`email` 字段需要特殊處理，因為我只想在使用者請求他們自己的數據時包含電子郵件。所以我使用 `include_email` 標誌來確定該字段是否包含在表示中。

注意 `last_seen` 字段是如何生成的。對於日期和時間字段，我將使用 ISO 8601 格式，Python 的 datetime 可以通過 `isoformat()` 方法生成。但因為 SQLAlchemy 使用的是 UTC 的天真 datetime 對象，但它們的狀態中沒有記錄時區，我需要首先設置時區，以確保它包含在 ISO 8601 字符串中。

最後，看看我如何實現超媒體連結。對於指向其他應用路由的三個連結，我使用 `url_for()` 生成 URL（目前指向我在 `app/api/users.py` 中定義的占位符視圖函數）。頭像連結是特殊的，因為它是一個應用外部的 Gravatar URL。對於這個連結，我使用了同樣的 `avatar()` 方法，我用它來在網頁中渲染頭像。

`to_dict()` 方法將使用者對象轉換為 Python 表示，然後將其轉換為 JSON。我還需要看看相反的方向，其中客戶端在請求中傳遞使用者表示，伺服器需要解析它並將其轉換為 User 對象。這是實現從 Python 字典到模型轉換的 `from_dict()` 方法：

```python
## app/models.py: 表示到使用者模型。

class User(UserMixin, db.Model):
    ## ...

    def from_dict(self, data, new_user=False):
        for field in ['username', 'email', 'about_me']:
            if field in data:
                setattr(self, field, data[field])
        if new_user and 'password' in data:
            self.set_password(data['password'])
```

在這種情況下，我決定使用一個循環來導入客戶端可以設置的任何字段，這些字段是 `username`、`email` 和 `about_me`。對於每個字段，我檢查在 `data` 參數中是否提供了一個值，如果有，我使用 Python 的 `setattr()` 在對象的相應屬性中設置新值。

`password` 字段被視為一個特例，因為它不是對象中

的一個字段。`new_user` 參數確定這是否是新使用者註冊，這意味著包含了一個密碼。要在使用者模型中設置密碼，我調用 `set_password()` 方法，它建立密碼雜湊。

### 表示使用者集合
除了處理單一資源表示之外，這個 API 還需要一種表示集合的方式。當客戶端請求使用者列表或追隨者列表時，將使用這種格式。這是使用者集合的表示：

```json
{
    "items": [
        {... user resource ...},
        {... user resource ...},
        ...
    ],
    "_meta": {
        "page": 1,
        "per_page": 10,
        "total_pages": 20,
        "total_items": 195
    },
    "_links": {
        "self": "http://localhost:5000/api/users?page=1",
        "next": "http://localhost:5000/api/users?page=2",
        "prev": null
    }
}
```

在這個表示中，`items` 是使用者資源的列表，每個資源都如前一節所述定義。`_meta` 部分包括客戶端可能在向使用者呈現分頁控制時發現有用的集合元數據。`_links` 部分定義了相關連結，包括指向集合本身的連結，以及前一頁和下一頁連結，也幫助客戶端分頁列表。

生成使用者集合的表示因分頁邏輯而棘手，但這個邏輯將對我未來可能想要加入到這個 API 中的其他資源通用，所以我將以一種我可以應用於其他模型的通用方式實現這個表示。回到第 16 章，我在全文檢索索引中處於類似情況，這是另一個我想要通用實現的功能，以便它可以應用於任何模型。我使用的解決方案是實現一個 `SearchableMixin` 類，任何需要全文索引的模型都可以繼承。我將使用相同的想法，所以這裡是一個我命名為 `PaginatedAPIMixin` 的新混入類：

```python
## app/models.py: 分頁表示混入類。

class PaginatedAPIMixin(object):
    @staticmethod
    def to_collection_dict(query, page, per_page, endpoint, **kwargs):
        resources = db.paginate(query, page=page, per_page=per_page,
                                error_out=False)
        data = {
            'items': [item.to_dict() for item in resources.items],
            '_meta': {
                'page': page,
                'per_page': per_page,
                'total_pages': resources.pages,
                'total_items': resources.total
            },
            '_links': {
                'self': url_for(endpoint, page=page, per_page=per_page,
                                **kwargs),
                'next': url_for(endpoint, page=page + 1, per_page=per_page,
                                **kwargs) if resources.has_next else None,
                'prev': url_for(endpoint, page=page - 1, per_page=per_page,
                                **kwargs) if resources.has_prev else None
            }
        }
        return data
```

`to_collection_dict()` 方法產生一個包含使用者集合表示的字典，包括 `items`、`_meta` 和 `_links` 部分。你可能需要仔細審查該方法以理解其運作方式。前三個參數是一個 SQLAlchemy 查詢、頁碼和頁面大小。這些參數決定了將返回哪些項目。實現使用 Flask-SQLAlchemy 的 `db.paginate()` 方法獲取一頁的項目，就像我在網頁應用的索引、探索和個人檔案頁面中對文章所做的那樣。

生成連結的複雜部分包

括自我參考和指向下一頁和前一頁的連結。我想讓這個函數通用，所以我不能，例如，使用 `url_for('api.get_users', id=id, page=page)` 生成自我連結。`url_for()` 的參數將依賴於特定的資源集合，所以我將依賴於呼叫者傳入 `endpoint` 參數作為 `url_for()` 調用中需要使用的視圖函數。由於許多應用中的路由需要參數，我還需要捕獲 `kwargs` 中的任何額外路由參數，並將這些參數傳遞給 `url_for()`。`page` 和 `per_page` 查詢字符串參數明確給出，因為這些控制了所有 API 路由的分頁。

這個混入類需要作為父類加入到 User 模型中：

```python
## app/models.py: 將 PaginatedAPIMixin 加入到 User 模型。

class User(PaginatedAPIMixin, UserMixin, db.Model):
    ## ...
```

在使用者集合的情況下，我不需要反向方向，因為我不會有任何路由需要客戶端發送使用者列表。如果項目要求客戶端發送使用者集合，我將需要實現一個 `from_collection_dict()` 方法。

### 錯誤處理
我在第 7 章定義的錯誤頁面僅適用於使用網頁瀏覽器與應用交互的使用者。當 API 需要返回一個錯誤時，它需要是一種 “機器友好型” 的錯誤，客戶端應用可以輕鬆解釋的東西。所以就像我為我的 API 資源定義了 JSON 表示一樣，現在我將決定 API 錯誤消息的表示。這是我將使用的基本結構：

```json
{
    "error": "short error description",
    "message": "error message (optional)"
}
```

除了錯誤的有效負載外，我將使用 HTTP 協定的狀態碼來指示錯誤的一般類別。為了幫助我生成這些錯誤響應，我將在 `app/api/errors.py` 中編寫 `error_response()` 函數：

```python
## app/api/errors.py: 錯誤響應。

from werkzeug.http import HTTP_STATUS_CODES

def error_response(status_code, message=None):
    payload = {'error': HTTP_STATUS_CODES.get(status_code, 'Unknown error')}
    if message:
        payload['message'] = message
    return payload, status_code
```

這個函數使用 Werkzeug 的方便的 `HTTP_STATUS_CODES` 字典（Flask 的一個核心依賴），為每個 HTTP 狀態碼提供了一個簡短描述性名稱。我使用這些名稱作為我的錯誤表示中的 `error` 字段，這樣我只需要擔心數字狀態碼和可選的長描述。這個表示被返回給 Flask，Flask 將其轉換為 JSON 並發送給客戶端。加入了一個帶有錯誤狀態碼的第二返回值，以覆蓋 Flask 發送的默認狀態碼 200（HTTP 狀態碼 “OK” 的狀態碼

）。

API 將返回的最常見錯誤將是程式碼 400，這是 “錯誤請求” 的錯誤。當客戶端發送包含無效數據的請求時，使用這個錯誤。為了更容易生成這個錯誤，我將加入一個專門的函數，它只需要長描述消息作為參數。這是我之前加入的 `bad_request()` 占位符：

```python
## app/api/errors.py: 錯誤請求響應。

## ...

def bad_request(message):
    return error_response(400, message)
```

API 藍圖可能會生成各種錯誤，Flask 默認會將其渲染為 HTML 錯誤頁面。為了確保來自 API 藍圖的所有錯誤都返回 JSON 格式響應，我可以安裝一個捕獲所有 API 錯誤的錯誤處理器：

```python
## app/api/errors.py: 捕獲所有 API 錯誤。

from werkzeug.exceptions import HTTPException
from app.api import bp

## ...

@bp.errorhandler(HTTPException)
def handle_exception(e):
    return error_response(e.code)
```

API 藍圖的 `errorhandler()` 裝飾器現在將被調用來處理所有基於 `HTTPException` 類的錯誤，Flask 用於所有 HTTP 錯誤。

### 使用者資源端點
我現在完成了處理使用者 JSON 表示所需的支持，所以我準備開始編碼 API 端點。

#### 檢索一個使用者
讓我們從檢索給定 id 的單一使用者請求開始：

```python
## app/api/users.py: 返回一個使用者。

from app.models import User

@bp.route('/users/<int:id>', methods=['GET'])
def get_user(id):
    return db.get_or_404(User, id).to_dict()
```

視圖函數接收請求的使用者 id 作為 URL 中的動態參數。Flask-SQLAlchemy 的 `db.get_or_404()` 輔助函數如果存在給定 id 的模型則返回模型，但當 id 不存在時，它中止請求並向客戶端返回 404 錯誤。這很方便，因為它去除了檢查查詢結果的需要，簡化了視圖函數中的邏輯。

最後，我加入到 User 的 `to_dict()` 方法被用於生成選定使用者的資源表示的字典，當返回給客戶端時 Flask 將自動將其轉換為 JSON。

如果你想看看這個第一個 API 路由是如何工作的，啟動伺服器然後在瀏覽器的地址欄中輸入以下 URL：

```
http://localhost:5000/api/users/1
```

這應該會以 JSON 格式顯示第一個使用者。也嘗試使用一個大的 id 值，看看 SQLAlchemy 查詢對象的 `get_or_404()` 方法如何觸發 404 錯誤（我稍後將展示如何擴展錯誤處理，以便這些錯誤也以 JSON 格式返回）。

為了以更合適的方式測試這個新路由，我將安裝 HTTPie，這是一個用 Python 編寫的命令行 HTTP 客戶端，使發送 API 請求變得容易：

```bash
(ven

v) $ pip install httpie
```

我現在可以從終端請求有關 id 為 1 的使用者（可能是你自己）的資訊，使用以下命令：

```bash
(venv) $ http GET http://localhost:5000/api/users/1
```

#### 檢索使用者集合
要返回所有使用者的集合，我現在可以依賴 `PaginatedAPIMixin` 的 `to_collection_dict()` 方法：

```python
## app/api/users.py: 返回所有使用者的集合。

import sqlalchemy as sa
from flask import request

## ...

@bp.route('/users', methods=['GET'])
def get_users():
    page = request.args.get('page', 1, type=int)
    per_page = min(request.args.get('per_page', 10, type=int), 100)
    return User.to_collection_dict(sa.select(User), page, per_page,
                                   'api.get_users')
```

對於這個實現，我首先從請求的查詢字符串中提取 `page` 和 `per_page`，如果它們未定義則分別使用 1 和 10 的默認值。`per_page` 參數有額外的邏輯，將其限制在 100 以內。給客戶端控制請求真正大的頁面並不是一個好主意，因為這可能會對伺服器造成性能問題。`page` 和 `per_page` 參數隨後傳遞給 `to_collection_dict()` 方法，連同返回所有使用者的查詢。最後一個參數是 `api.get_users`，這是我需要用於表示中使用的三個連結的端點名稱。

要用 HTTPie 測試這個端點，使用以下命令：

```bash
(venv) $ http GET http://localhost:5000/api/users
```

接下來的兩個端點是返回追隨者和被追隨的使用者。這些與上面的非常相似：

```python
## app/api/users.py: 返回追隨者和被追隨的使用者。

@bp.route('/users/<int:id>/followers', methods=['GET'])
def get_followers(id):
    user = db.get_or_404(User, id)
    page = request.args.get('page', 1, type=int)
    per_page = min(request.args.get('per_page', 10, type=int), 100)
    return User.to_collection_dict(user.followers.select(), page, per_page,
                                   'api.get_followers', id=id)


@bp.route('/users/<int:id>/following', methods=['GET'])
def get_following(id):
    user = db.get_or_404(User, id)
    page = request.args.get('page', 1, type=int)
    per_page = min(request.args.get('per_page', 10, type=int), 100)
    return User.to_collection_dict(user.following.select(), page, per_page,
                                   'api.get_following', id=id)
```

由於這兩個路由特定於一個使用者，它們在 URL 中有 `id` 動態參數。`id` 被用於從資料庫獲取使用者，然後向 `to_collection_dict()` 方法提供 `user.followers` 和 `user.following` 關係查詢。希望現在你可以看到，花一點額外的時間並以通用方式設計這個方法真的是值得的。`to_collection_dict()` 的最後兩個參數是端點名稱，和 `id`，方法將其作為 `kwargs` 中的額外關鍵字參數接受，然後在生成表示的連結部分時將其傳遞給 `url_for()`。

類似於之前的例子，你可以使用 HTTPie 按照以下方式測試這兩個路

由：

```bash
(venv) $ http GET http://localhost:5000/api/users/1/followers
(venv) $ http GET http://localhost:5000/api/users/1/following
```

我應該指出，多虧了超媒體，你不需要記住這些 URL，因為它們包含在使用者表示的 `_links` 部分中。

### 註冊新使用者
`POST` 請求到 `/users` 路由將被用來註冊新使用者帳號。你可以在下面看到這個路由的實現：

```python
## app/api/users.py: 註冊一個新使用者。

from flask import url_for
from app import db
from app.api.errors import bad_request

@bp.route('/users', methods=['POST'])
def create_user():
    data = request.get_json()
    if 'username' not in data or 'email' not in data or 'password' not in data:
        return bad_request('必須包括使用者名稱、電子郵件和密碼字段')
    if db.session.scalar(sa.select(User).where(
            User.username == data['username'])):
        return bad_request('請使用不同的使用者名稱')
    if db.session.scalar(sa.select(User).where(
            User.email == data['email'])):
        return bad_request('請使用不同的電子郵件地址')
    user = User()
    user.from_dict(data, new_user=True)
    db.session.add(user)
    db.session.commit()
    return user.to_dict(), 201, {'Location': url_for('api.get_user',
                                                     id=user.id)}
```

這個請求將接受客戶端提供的、以 JSON 格式的使用者表示，包含在請求體中。Flask 提供了 `request.get_json()` 方法從請求中提取 JSON 體並將其作為 Python 結構返回。如果客戶端發送的內容不是 JSON 格式，或者 JSON 內容格式錯誤，這個方法可能導致請求以 415 狀態碼（不支持的媒體類型）失敗，這兩種情況都將由 `app/api/errors.py` 中的 `handle_http_exception()` 處理器處理。

在我可以使用數據之前，我需要確保我獲得了所有訊息，所以我從檢查三個必需字段是否包含開始。這些是使用者名稱、電子郵件和密碼。如果任何這些缺失，那麼我使用 `app/api/errors.py` 模塊中的 `bad_request()` 輔助函數向客戶端返回一個錯誤。除了這個檢查外，我需要確保使用者名稱和電子郵件字段沒有被另一個使用者使用，所以我嘗試從資料庫通過提供的使用者名稱和電子郵件加載一個使用者，如果任何這些返回了一個有效的使用者，我也會向客戶端返回一個錯誤。

一旦我通過了數據驗證，我就可以輕鬆建立一個使用者對象並將其加入到資料庫中。建立使用者時，我依賴於 User 模型中的 `from_dict()` 方法。`new_user` 參數設置為 True，以便它也接受通常不是使用者表示的一部分的密碼字段。

對於這個請求的回應將是新使用者的表示，所以 `to_dict()` 生成了該有效載荷。建立資源的 `POST` 請求的狀態碼應該是 201，這是當新實體已被建立時使用的程式碼。此外，HTTP 協定要求 201 響應包含一個 `Location` 頭部，設置為新資源的 URL，我可以使用 `url_for()` 生成。

下面你可以看到如何

通過 HTTPie 從命令行註冊一個新使用者：

```bash
(venv) $ http POST http://localhost:5000/api/users username=alice password=dog \
    email=alice@example.com "about_me = 你好，我是 Alice。"
```

### 編輯使用者
我在我的 API 中將要使用的最後一個端點是用於修改現有使用者的：

```python
## app/api/users.py: 修改一個使用者。

@bp.route('/users/<int:id>', methods=['PUT'])
def update_user(id):
    user = db.get_or_404(User, id)
    data = request.get_json()
    if 'username' in data and data['username'] != user.username and \
        db.session.scalar(sa.select(User).where(
            User.username == data['username'])):
        return bad_request('請使用不同的使用者名稱')
    if 'email' in data and data['email'] != user.email and \
        db.session.scalar(sa.select(User).where(
            User.email == data['email'])):
        return bad_request('請使用不同的電子郵件地址')
    user.from_dict(data, new_user=False)
    db.session.commit()
    return user.to_dict()
```

對於這個請求，我收到一個使用者 id 作為 URL 的動態部分，所以我可以加載指定的使用者並在找不到時返回 404 錯誤。請注意，目前還沒有身份驗證，所以現在 API 將允許使用者對任何其他使用者的帳戶進行更改。這顯然是一個安全問題，但我將在下一節中解決。

像新使用者的情況一樣，我需要驗證客戶端提供的使用者名稱和電子郵件字段在我可以使用它們之前不會與其他使用者衝突，但在這種情況下，驗證更加棘手。首先，這些字段在這個請求中是可選的，所以我需要檢查一個字段是否存在。第二個複雜的是，客戶端可能提供了相同的值，所以在我檢查使用者名稱或電子郵件是否被占用之前，我需要確保它們與當前的不同。如果任何這些驗證檢查失敗，那麼我就像以前一樣向客戶端返回 400 錯誤。

一旦數據通過了驗證，我可以使用 User 模型的 `from_dict()` 方法導入客戶端提供的所有數據，然後將更改提交到資料庫。這個請求的回應將使用者更新的表示返回給使用者，默認狀態碼為 200。

這是一個使用 HTTPie 編輯 `about_me` 字段的範例請求：

```bash
(venv) $ http PUT http://localhost:5000/api/users/2 "about_me = 嗨，我是 Miguel"
```

### API 身份驗證
我在前一節中加入的 API 端點目前對任何客戶端都是開放的。顯然，它們需要僅對註冊使用者可用，為此我需要加入身份驗證和授權，或簡稱為 "AuthN" 和 "AuthZ"。這個想法是客戶端發送的請求提供某種身份識別，以便伺服器知道客戶端代表哪個使用者，並可以驗證該使用者是否允許執行

所請求的操作。

保護這些 API 端點最明顯的方式是使用 Flask-Login 的 `@login_required` 裝飾器，但這種方法對於 API 端點有一些問題。當裝飾器檢測到非認證使用者時，它將使用者重定向到 HTML 登錄頁面。在 API 中沒有 HTML 或登錄頁面的概念，如果客戶端發送的請求包含無效或缺少憑證，伺服器必須拒絕請求並返回 401 狀態碼。伺服器不能假設 API 客戶端是網頁瀏覽器，或者它可以處理重定向，或者它可以渲染和處理 HTML 登錄表單。當 API 客戶端收到 401 狀態碼時，它知道需要向使用者請求憑證，但如何做到這一點真的不是伺服器的事情。

#### 使用者模型中的令牌
對於 API 身份驗證的需求，我將使用令牌認證方案。當客戶端想要開始與 API 交互時，它需要請求一個臨時令牌，使用使用者名稱和密碼進行認證。然後客戶端可以發送 API 請求，傳遞令牌作為認證，只要令牌有效即可。一旦令牌過期，需要請求一個新的令牌。為了支持使用者令牌，我將擴展 User 模型：

```python
## app/models.py: 支援使用者令牌。

from datetime import timedelta
import secrets

class User(PaginatedAPIMixin, UserMixin, db.Model):
    ## ...
    token: so.Mapped[Optional[str]] = so.mapped_column(
        sa.String(32), index=True, unique=True)
    token_expiration: so.Mapped[Optional[datetime]]

    ## ...

    def get_token(self, expires_in=3600):
        now = datetime.now(timezone.utc)
        if elseelf.token and self.token_expiration.replace(
                tzinfo=timezone.utc) > now + timedelta(seconds=60):
            return self.token
        self.token = secrets.token_hex(16)
        self.token_expiration = now + timedelta(seconds=expires_in)
        db.session.add(self)
        return self.token

    def revoke_token(self):
        self.token_expiration = datetime.now(timezone.utc) - timedelta(
            seconds=1)

    @staticmethod
    def check_token(token):
        user = db.session.scalar(sa.select(User).where(User.token == token))
        if user is None or user.token_expiration.replace(
                tzinfo=timezone.utc) <datetime.now(timezone.utc):
            return None
        return user
```

有了這個變化，我為使用者模型加入了一個 `token` 屬性，並且因為我需要按它在資料庫中進行搜索，所以我使其唯一且索引。我還加入了 `token_expiration` 字段，它具有令牌到期的日期和時間。這樣令牌就不會長時間保持有效，這可能成為安全風險。

我建立了三個方法來處理這些令牌。`get_token()` 方法為使用者返回一個令牌。令牌使用 Python 標準庫中的 `secrets.token_hex()` 函數生成。`token` 字段長度為 32 個字符，所以我必須傳遞 16 給 `token_hex()`，以便生成的令牌有 16 個字節，當以十六進制渲染

時將使用 32 個字符。在建立新令牌之前，這個方法檢查當前分配的令牌在過期前是否至少還有一分鐘，如果是這樣，則返回現有令牌。

在處理令牌時，擁有一個策略立即撤銷令牌，而不僅僅依賴於過期日期，這是一個經常被忽視的安全最佳實踐。`revoke_token()` 方法使當前分配給使用者的令牌無效，僅僅通過將過期日期設置為當前時間之前的一秒鐘。

`check_token()` 方法是一個靜態方法，它接受一個令牌作為輸入，並返回這個令牌屬於的使用者作為響應。如果令牌無效或過期，方法返回 None。

因為我對資料庫進行了更改，我需要生成一個新的資料庫遷移，然後用它升級資料庫：

```bash
(venv) $ flask db migrate -m "user tokens"
(venv) $ flask db upgrade
```

#### 令牌請求
當你寫一個 API 時，你必須考慮你的客戶端並不總是連接到網頁應用的網頁瀏覽器。API 的真正力量來自於當智能手機應用或甚至基於瀏覽器的單頁應用可以訪問後端服務時。當這些專業客戶端需要訪問 API 服務時，它們從請求一個令牌開始，這是傳統網頁應用中登錄表單的對應物。

為了簡化客戶端和伺服器在使用令牌認證時的互動，我將使用一個叫做 Flask-HTTPAuth 的 Flask 擴展。Flask-HTTPAuth 通過 pip 安裝：

```bash
(venv) $ pip install flask-httpauth
```

Flask-HTTPAuth 支持幾種不同的認證機制，都是對 API 友好的。首先，我將使用 HTTP 基本認證，在這種情況下，客戶端在標準 `Authorization` HTTP 頭中發送使用者憑證。為了與 Flask-HTTPAuth 集成，應用需要提供兩個函數：一個定義檢查使用者提供的使用者名稱和密碼的邏輯，另一個在身份驗證失敗的情況下返回錯誤響應。這些函數通過裝飾器註冊到 Flask-HTTPAuth 中，然後在身份驗證流程中根據需要由擴展自動調用。你可以在下面看到實現：

```python
## app/api/auth.py: 基本認證支持。

import sqlalchemy as sa
from flask_httpauth import HTTPBasicAuth
from app import db
from app.models import User
from app.api.errors import error_response

basic_auth = HTTPBasicAuth()

@basic_auth.verify_password
def verify_password(username, password):
    user = db.session.scalar(sa.select(User).where(User.username == username))
    if user and user.check_password(password):
        return user

@basic_auth.error_handler
def basic_auth_error(status):
    return error_response

(status)
```

Flask-HTTPAuth 中的 `HTTPBasicAuth` 類實現了基本認證流程。兩個必需的函數分別通過 `verify_password` 和 `error_handler` 裝飾器配置。

驗證函數接收客戶端提供的使用者名稱和密碼，如果憑證有效則返回經過認證的使用者，否則返回 None。為了檢查密碼，我依賴 User 類的 `check_password()` 方法，這也被 Flask-Login 在網頁應用的認證中使用。經過認證的使用者將作為 `basic_auth.current_user()` 可用，以便在 API 視圖函數中使用。

錯誤處理函數返回標準的錯誤響應，這是由 `app/api/errors.py` 中的 `error_response()` 函數生成的。`status` 參數是 HTTP 狀態碼，在無效身份驗證的情況下將是 401。401 錯誤在 HTTP 標準中定義為 “未授權” 錯誤。HTTP 客戶端知道當它們收到這個錯誤時，它們發送的請求需要重新發送並帶有有效的憑證。

現在我已經實現了基本認證支持，所以我可以加入令牌檢索路由，客戶端將在需要令牌時調用：

```python
## app/api/tokens.py: 生成使用者令牌。

from app import db
from app.api import bp
from app.api.auth import basic_auth

@bp.route('/tokens', methods=['POST'])
@basic_auth.login_required
def get_token():
    token = basic_auth.current_user().get_token()
    db.session.commit()
    return {'token': token}
```

這個視圖函數被 `HTTPBasicAuth` 實例的 `@basic_auth.login_required` 裝飾器裝飾，它將指示 Flask-HTTPAuth 驗證身份驗證（通過我上面定義的驗證函數）並只在提供的憑證有效時允許函數運行。這個視圖函數的實現依賴於使用者模型的 `get_token()` 方法來產生令牌。在生成令牌後發出資料庫提交以確保令牌及其過期時間被寫回資料庫。

如果你嘗試向令牌 API 路由發送 `POST` 請求，會發生以下情況：

```bash
(venv) $ http POST http://localhost:5000/api/tokens
```

HTTP 響應包括 401 狀態碼，和我在 `basic_auth_error()` 函數中定義的錯誤有效載荷。這是同一請求，這次包括基本認證憑證：

```bash
(venv) $ http --auth <username>:<password> POST http://localhost:5000/api/tokens
```

現在狀態碼是 200，這是成功請求的程式碼，有效載荷包括使用者的新生成的令牌。請注意，當你發送這個請求時，你需要用自己的憑證替換 `<username>:<password>`，同你使用登錄表單的憑證一樣。使用者名稱和密碼需要用冒號作為分隔

符提供。

#### 使用令牌保護 API 路由
客戶端現在可以請求一個令牌來使用 API 端點，所以剩下的就是向這些端點加入令牌驗證。這是 Flask-HTTPAuth 也可以為我處理的事情。我需要建立基於 `HTTPTokenAuth` 類的第二個認證實例，並提供一個令牌驗證回調：

```python
## app/api/auth.py: 令牌認證支持。

## ...
from flask_httpauth import HTTPTokenAuth

## ...
token_auth = HTTPTokenAuth()

## ...

@token_auth.verify_token
def verify_token(token):
    return User.check_token(token) if token else None

@token_auth.error_handler
def token_auth_error(status):
    return error_response(status)
```

使用令牌認證時，Flask-HTTPAuth 使用 `verify_token` 裝飾的函數，但除此之外，令牌認證的工作方式與基本認證相同。我的令牌驗證函數使用 `User.check_token()` 來定位擁有提供的令牌的使用者並返回它。如前所述，返回 `None` 將導致客戶端被拒絕並出現身份驗證錯誤。

要用令牌保護 API 路由，需要加入 `@token_auth.login_required` 裝飾器：

```python
## app/api/users.py: 用令牌認證保護使用者路由。

from flask import abort
from app.api.auth import token_auth

@bp.route('/users/<int:id>', methods=['GET'])
@token_auth.login_required
def get_user(id):
    ## ...

@bp.route('/users', methods=['GET'])
@token_auth.login_required
def get_users():
    ## ...

@bp.route('/users/<int:id>/followers', methods=['GET'])
@token_auth.login_required
def get_followers(id):
    ## ...

@bp.route('/users/<int:id>/following', methods=['GET'])
@token_auth.login_required
def get_following(id):
    ## ...

@bp.route('/users', methods=['POST'])
def create_user():
    ## ...

@bp.route('/users/<int:id>', methods=['PUT'])
@token_auth.login_required
def update_user(id):
    if token_auth.current_user().id != id:
        abort(403)
    ## ...
```

請注意，除了 `create_user()` 之外，所有 API 視圖函數都加入了裝飾器，這顯然不能接受認證，因為將要請求令牌的使用者需要首先被建立。還要注意，修改使用者的 `PUT` 請求有一個額外的檢查，防止使用者嘗試修改另一個使用者的帳戶。如果我發現請求的使用者 id 與經過認證的使用者的 id 不匹配，那麼我將返回一個 403 錯誤響應，這表明客戶端沒有權限執行請求的操作。

如果你像之前所示發送任何這些端點的請求，你將得到 401 錯誤響應。為了獲得訪問權限，你需要加入 `Authorization` 頭部，帶有你從 `/api/tokens` 請求中收到的令牌。Flask-HTTPAuth 期望令牌作為 “bearer” 令牌發送，可以用 HTTPie 如下發送：

```bash
(venv) $ http -A bearer --auth <token> GET http://localhost:5000/api/users/1
```

### 撤銷令牌
我將要實現的最後一個與令牌相關的功能是令牌撤銷，你可以在下面看到：

```python
## app/api/tokens.py: 撤銷令牌。

from app.api.auth import token_auth

@bp.route('/tokens', methods=['DELETE'])
@token_auth.login_required
def revoke_token():
    token_auth.current_user().revoke_token()
    db.session.commit()
    return '', 204
```

客戶端可以發送 `DELETE` 請求到 `/tokens` URL 來使令牌無效。這個路由的認證是基於令牌的，事實上，在 `Authorization` 頭部發送的令牌就是被撤銷的那個。撤銷本身使用 User 類中的輔助方法，該方法重置令牌的過期日期。資料庫會話被提交，以便將這一變更寫入資料庫。這個請求的回應沒有主體，所以我可以返回一個空字符串。在返回語句中的第二個值將回應的狀態碼設置為 204，這是對於沒有回應主體的成功請求要使用的程式碼。

這是從 HTTPie 發送的一個範例令牌撤銷請求：

```bash
(venv) $ http -A bearer --auth <token> DELETE http://localhost:5000/api/tokens
```

### 對 API 友好的錯誤訊息
你還記得本章早些時候我要求你從瀏覽器發送一個無效使用者 URL 的 API 請求時發生了什麼嗎？伺服器返回了 404 錯誤，但這個錯誤被格式化為標準的 404 HTML 錯誤頁面。API 可能需要返回的許多錯誤可以在 API 藍圖中用 JSON 版本覆蓋，但有些由 Flask 處理的錯誤仍然通過全局為應用註冊的錯誤處理器處理，這些仍然返回 HTML。

HTTP 協定支援一種機制，稱為內容協商，客戶端和伺服器可以根據客戶端偏好協商回應的最佳格式。客戶端需要在請求中發送一個 `Accept` 頭部，指示格式偏好。然後伺服器查看列表，並使用客戶端提供的列表中它支援的最佳格式進行回應。

我想做的是修改全局應用錯誤處理器，以便它們使用內容協商根據客戶端偏好以 HTML 或 JSON 回應。這可以使用 Flask 的 `request.accept_mimetypes` 對象完成：

```python
## app/errors/handlers.py: 錯誤回應的內容協商。

from flask import render_template, request
from app import db
from app.errors import bp
from app.api.errors import error_response as api_error_response

def wants_json_response():
    return request.accept_mimetypes['application/json'] >= \
        request.accept_mimetypes['text/html']

@bp.app_errorhandler(404)
def not_found_error(error):
    if wants_json_response():
        return api_error_response(404)
    return render_template('errors/404.html'), 404

@bp.app_errorhandler(500)
def internal_error(error):
    db.session.rollback()
    if wants_json_response():
        return api_error_response(500)
    return render_template('errors/500.html'), 500


```

`wants_json_response()` 輔助函數比較了客戶端在其偏好格式列表中選擇的 JSON 或 HTML 的偏好。如果 JSON 的評級高於 HTML，那麼我返回一個 JSON 回應。否則我將返回基於模板的原始 HTML 回應。對於 JSON 回應，我將從 API 藍圖導入 `error_response` 輔助函數，但在這裡我將其重命名為 `api_error_response()`，以便清楚它的功能和來源。

### 最後的話

恭喜你完成了 Flask Mega-Tutorial！我希望你現在已經做好準備，可以構建自己的網頁應用並使用你所學的知識作為基礎繼續你的學習之旅。祝你好運！