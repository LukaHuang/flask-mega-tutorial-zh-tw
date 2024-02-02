## Chapter 4: Database

> Posted by on [Miguel Grinberg](https://blog.miguelgrinberg.com/author/Miguel%20Grinberg)

這章的主題極為重要。對於大多數應用程式來說，維護可有效檢索的持久數據是必要的，這正是資料庫的用途。

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

本章的 GitHub 連結為：[Browse](https://github.com/miguelgrinberg/microblog/tree/v0.4) [Zip](https://github.com/miguelgrinberg/microblog/archive/v0.4.zip) [Diff](https://github.com/miguelgrinberg/microblog/compare/v0.3...v0.4)

### Flask 中的資料庫
如你所知，Flask 並不內建支援資料庫。這是 Flask 故意保持中立的眾多領域之一，這很好，因為你可以自由選擇最適合你應用程式的資料庫，而不是被迫適應某一種。

在 Python 中有許多優秀的資料庫選擇，很多都有支援 Flask 的擴充功能，使其更好地整合到應用程式中。這些資料庫大致可以分為兩大類：遵循關聯模型的，和不遵循的。後者通常被稱為 NoSQL，表示它們不實現流行的關聯查詢語言 SQL。雖然這兩組中都有優秀的資料庫產品，但我認為，對於有結構化數據（如使用者列表、部落格文章等）的應用程式來說，關聯資料庫更合適，而對於結構不太明確的數據，NoSQL 資料庫則更適合。這個應用程式，像大多數其他應用程式一樣，可以使用任何一種資料庫實現，但基於上述原因，我將選擇關聯資料庫。

在第 3 章中，我向你展示了第一個 Flask 擴充功能。在本章中，我將使用另外兩個。第一個是 Flask-SQLAlchemy，它提供了對流行的 SQLAlchemy 套件的 Flask 友好封裝，這是一個物件關聯映射器（ORM）。ORM 允許應用程式使用高階實體（如類別、物件和方法）而非表格和 SQL 來管理資料庫。ORM 的工作是將高階操作翻譯成資料庫命令。

SQLAlchemy 的好處是它不僅是一個 ORM，而且支援多種關聯資料庫。SQLAlchemy 支援許多資料庫引擎，包括流行的 MySQL、PostgreSQL 和 SQLite。這非常強大，因為你可以在簡單的 SQLite 資料庫上進行開發，這不需要伺服器，然後當應用程式部署到生產伺服器時，你可以選擇更強大的 MySQL 或 PostgreSQL 伺服器，而不必更改應用程式。

要在你的虛擬環境中安裝 Flask-SQLAlchemy，請確保先啟動它，然後執行：

```python
(venv) $ pip install flask-sqlalchemy
```

### 資料庫遷移
我看過的大多數資料庫教程都涵蓋了資料庫的建立和使用，但沒有充分解決應用程式需求變化或增長時對現有資料庫進行更新的問題。這很難，因為關聯資料庫以結構化數據為中心，所以當結構改變時，資料庫中已有的數據需要遷移到修改後的結構。

本章我將介紹的第二個擴充功能是 Flask-Migrate，這其實是我自己建立的。這個擴充功能是 Alembic 的 Flask 封裝，Alembic 是 SQLAlchemy 的資料庫遷移框架。使用資料庫遷移在開始時會增加一些工作量，但這是為了將來能夠堅固地更改資料庫而付出的小代價。

安裝 Flask-Migrate 的過程與你之前看過的其他擴充功能類似：

```python
(venv) $ pip install flask-migrate
```

### Flask-SQLAlchemy 配置
在開發期間，我將使用 SQLite 資料庫。對於開發小型應用程式（有時甚至不是很小的應用程式）來說，SQLite 資料庫是最方便的選擇，因為每個資料庫都存儲在磁碟上的單一檔案中，並且不需要運行像 MySQL 和 PostgreSQL 這樣的資料庫伺服器。

Flask-SQLAlchemy 需要在配置檔案中加入一個新的配置項：

config.py：Flask-SQLAlchemy 配置

```python
import os
basedir = os.path.abspath(os.path.dirname(__file__))

class Config(object):
    ## ...
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
        'sqlite

:///'+ os.path.join(basedir,'app.db')
```

Flask-SQLAlchemy 擴充功能從 SQLALCHEMY_DATABASE_URI 配置變數中取得應用程式的資料庫位置。如你在第 3 章回憶的，通常將配置從環境變數中設置是一個好做法，並在環境未定義該變數時提供一個回退值。在這個案例中，我從 DATABASE_URL 環境變數中取得資料庫 URL，如果沒有定義，我將配置一個名為 app.db 的資料庫，位於應用程式的主目錄中，存儲在 basedir 變數中。

資料庫將在應用程式中由資料庫實例代表。資料庫遷移引擎也將有一個實例。這些是需要在 app/__init__.py 檔案中在應用程式之後建立的物件：

app/__init__.py：Flask-SQLAlchemy 和 Flask-Migrate 初始化

```python
from flask import Flask
from config import Config
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate

app = Flask(__name__)
app.config.from_object(Config)
db = SQLAlchemy(app)
migrate = Migrate(app, db)

from app import routes, models
```

我對 __init__.py 檔案進行了三項更改。首先，我加入了一個代表資料庫的 db 物件。然後加入了 migrate，來代表資料庫遷移引擎。希望你在處理 Flask 擴充功能時看到一個模式。大多數擴充功能都是像這兩個一樣初始化的。在最後一項更改中，我在底部導入了一個名為 models 的新模組。這個模組將定義資料庫的結構。

### 資料庫模型
將要存儲在資料庫中的數據將由一組通常稱為資料庫模型的類別來代表。SQLAlchemy 內部的 ORM 層將執行所需的轉換，將從這些類別建立的物件映射到適當資料庫表中的行。

讓我們從建立代表使用者的模型開始。使用 WWW SQL Designer 工具，我製作了以下圖表，來代表我們想在使用者表中使用的數據：

![](./img/2024-02-02-02-03-31.png)

id 欄位通常存在於所有模型中，並用作主鍵。資料庫中的每個使用者都將被分配一個唯一的 id 值，儲存在此欄位中。主鍵在大多數情況下都是由資料庫自動分配的，所以我只需要提供標記為主鍵的 id 欄位。

username、email 和 password_hash 欄位被定義為字串（或資料庫術語中的 VARCHAR），並指定了它們的最大長度，以便資料庫優化空間使用。雖然 username 和 email 欄位是不言自明的，但 password_hash 欄位值得關注。我想確保我正在構建的應用程式採用安全最佳實踐，出於這個原因，我不會以純文字形式儲存使用者密碼。存儲密碼的問題是，如果資料庫被破解，攻擊者將能夠訪問密碼，這對使用者來說可能是災難性的。我不是直接寫密碼，而是寫密碼雜湊，這大大提高了安全性。這將是另一章的主題，所以現在不用太擔心。

既然我知道我想要的使用者表，我就可以在新的 app/models.py 模組中將其翻譯成程式碼：

app/models.py：使用者資料庫模型

```python
from typing import Optional
import sqlalchemy as sa
import sqlalchemy.orm as so
from app import db

class User(db.Model):
    id: so.Mapped[int] = so.mapped_column(primary_key=True)
    username: so.Mapped[str] = so.mapped_column(sa.String(64), index=True,
                                                unique=True)
    email: so.Mapped[str] = so.mapped_column(sa.String(120), index=True,
                                             unique=True)
    password_hash: so.Mapped[Optional[str]] = so.mapped_column(sa.String(256))

    def __repr__(self):
        return '<User {}>'.format(self.username)
```

我從 `SQLAlchemy` 套件中匯入 `sqlalchemy` 和 `sqlalchemy.orm` 模組，這些模組提供了大多數與資料庫互動所需的元素。`sqlalchemy` 模組包含通用資料庫函式和類別，例如型態和查詢建構助手，而 `sqlalchemy.orm` 則提供了使用模型的支援。考慮到這兩個模組名稱很長，且需要經常引用，所以直接在匯入語句中定義 `sa` 和 `so` 的別名。還有從 Flask-SQLAlchemy 匯入 `db` 實例，以及從 Python 匯入 `Optional` 型態提示。

上面建立的 `User` 類別將代表儲存在資料庫中的使用者。該類別繼承自 `db.Model`，這是 Flask-SQLAlchemy 所有模型的基礎類別。`User` 模型定義了幾個作為類別變數的欄位。這些欄位將在對應的資料庫表格中被建立。

欄位使用 Python 型態提示指派型態，並用 SQLAlchemy 的 `so.Mapped` 泛型型態包裹。像是 `so.Mapped[int]` 或 `so.Mapped[str]` 這樣的型態聲明，不僅定義了欄位的型態，還使得值在資料庫術語中是必需的，或者不可為空。要定一個允許為空或可為空的欄位，也會加入 Python 的 `Optional` 輔助函式，正如 `password_hash` 所示。

在大多數情況下，定義表格欄位需要的不僅僅是欄位型態。SQLAlchemy 透過指派給每個欄位的 `so.mapped_column()` 函式呼叫提供這些額外配置。在上面的 `id` 欄位的情況，該欄位被配置為主鍵。對於字符串欄位，許多資料庫需要給定長度，因此也包含了這個配置。我還包括了其他可選參數，讓我可以指出哪些欄位是唯一的和已索引的，這對於保持資料庫的一致性和提高搜尋效率是重要的。

`__repr__` 方法告訴 Python 如何列印這個類別的物件，這對於除錯很有用。你可以在下方的 Python 解釋器會話中看到 `__repr__()` 方法的實際應用：

```python
>>> from app.models import User
>>> u = User(username='susan', email='susan@example.com')
>>> u
<User susan>
```

#### 建立遷移儲存庫

在前一節中建立的模型類別定義了這個應用的初始資料庫結構（或架構）。但隨著應用的持續發展，我可能需要對該結構進行更改，例如加入新項目，有時修改或移除項目。Alembic（Flask-Migrate 使用的遷移框架）將以不需要每次更改都重新建立資料庫的方式進行這些架構變更。

為了完成這看似困難的任務，Alembic 維護了一個遷移儲存庫，這是一個目錄，用於存儲其遷移腳本。每次對資料庫架構進行更改時，都會將一個遷移腳本加入到儲存庫中，並詳述更改的細節。要將遷移應用到資料庫，這些遷移腳本會按照它們建立的順序執行。

Flask-Migrate 通過 flask 命令暴露其指令。你已經看過 `flask run`，這是 Flask 原生的子命令。`flask db` 子命令是由 Flask-Migrate 加入的，用於管理與資料庫遷移相關的所有事物。那麼，讓我們透過執行 `flask db init` 為 microblog 建立遷移儲存庫：

```bash
(venv) $ flask db init
  建立目錄 /home/miguel/microblog/migrations ... 完成
  建立目錄 /home/miguel/microblog/migrations/versions ... 完成
  生成 /home/miguel/microblog/migrations/alembic.ini ... 完成
  生成 /home/miguel/microblog/migrations/env.py ... 完成
  生成 /home/miguel/microblog/migrations/README ... 完成
  生成 /home/miguel/microblog/migrations/script.py.mako ... 完成
  請在繼續之前編輯
  '/home/miguel/microblog/migrations/alembic.ini' 中的配置 / 連接 / 日誌設定。
```

記得 `flask` 命令依賴於 `FLASK_APP` 環境變數來知道 Flask 應用的位置。對於這個應用，你希望將 `FLASK_APP` 設為 `microblog.py` 的值，正如第一章中討論的那樣。如果你在項目中包含了一個 `.flaskenv` 檔案，那麼 flask 命令的所有子命令都將自動訪問應用。

在你執行 `flask db init` 命令後，你會發現一個新的 migrations 目錄，裡面有一些檔案和一個 versions 子目錄。從現在開始，所有這些檔案都應被視為你的項目的一部分，特別是應該與你的應用程式碼一起加入到原始碼控制中。

### 第一個資料庫遷移

有了遷移儲存庫後，就是時候建立第一個資料庫遷移，它將包括映射到 User 資料庫模型的 users 表格。建立資料庫遷移有兩種方式：手動或自動。為了自動生成遷移，Alembic 會比較資料庫模型定義的資料庫架構和資料庫中當前使用的實際資料庫架構。然後，它會將必要的更改填充到遷移腳本中，以使資料庫架構與應用模型匹配。在這種情況下，由於沒有之前的資料庫，自動遷移將把整個 User 模型加入到遷移腳本中。`flask db migrate` 子命令生成這些自動遷移：

```bash
(venv) $ flask db migrate -m "users table"
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.autogenerate.compare] Detected added table 'user'
INFO  [alembic.autogenerate.compare] Detected added index 'ix_user_email' on '['email']'
INFO  [alembic.autogenerate.compare] Detected added index 'ix_user_username' on '['username']'
  生成 /home/miguel/microblog/migrations/versions/e517276bb1c2_users_table.py ... 完成
```

命令的輸出讓你了解 Alembic 包含了哪些遷移。前兩行是資訊性的，通常可以忽略。然後它表示發現了一個 user 表格和兩個索引。然後告訴你它在哪裡寫了遷移腳本。`e517276bb1c2` 值是自動生成且唯一的遷移程式碼（對你來說會不同）。用 `-m` 選項給出的註釋是可選的，它只是為遷移加入了一個簡短的描述性文字。

生成的遷移腳本現在是你的項目的一部分，如果你正在使用 git 或其他原始碼控制工具，它需要作為一個額外的原始檔案與其他存儲在 migrations 目錄中的檔案一起整合。歡迎你檢查腳本，如果你好奇它的樣子。你會發現它有兩個函式 `upgrade()` 和 `downgrade()`。`upgrade()` 函式應用遷移，而 `downgrade()` 函式則移除它。這使得 Alembic 能夠將資料庫遷移到歷史中的任何點，甚至是更舊的版本，通過使用降級路徑。

`flask db migrate` 命令不會對資料庫進行任何更改，它只是生成遷移腳本。要將更改應用到資料庫，必須使用 `flask db upgrade` 命令。

```bash
(venv) $ flask db upgrade
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade e517276bb1c2 -> 780739b227a7, posts table
```

由於這個應用使用 SQLite，`upgrade` 命令將檢測到沒有資料庫並將其建立（在此命令完成後，你會注意到加入了一個名為 app.db 的檔案，那是 SQLite 資料庫）。在使用像是 MySQL 和 PostgreSQL 這樣的資料庫伺服器時，你必須在執行 `upgrade` 之前在資料庫伺服器中建立資料庫。

注意，Flask-SQLAlchemy 預設使用「蛇形命名法」來為資料庫表命名。對於上面的 User 模型，對應的資料庫表將被命名為 user。對於 AddressAndPhone 模型類別，表將被命名為 address_and_phone。如果你更喜歡選擇自己的表名，你可以在模型類別中加入一個名為 `__tablename__` 的屬性，將其設置為你希望的名稱字符串。

### 資料庫升級與降級工作流程
此時應用程式還處於起步階段，但討論接下來的資料庫遷移策略並無妨。假設你在開發機器上有你的應用程式副本，並且還部署了一份副本到線上使用的生產伺服器上。

假設在你的應用程式下一個版本中，你需要對模型進行更改，例如需要加入一個新表。沒有遷移的話，你需要弄清楚如何更改你的資料庫架構，無論是在你的開發機器還是再次在你的伺服器上，這可能會是大量的工作。

但有了資料庫遷移支援，當你修改了應用程式中的模型後，你生成一個新的遷移腳本（flask db migrate），你審查它以確保自動生成做了正確的事情，然後將更改應用於你的開發資料庫（flask db upgrade）。你將遷移腳本加入到原始碼控制中並提交它。

當你準備好將應用程式的新版本發布到你的生產伺服器時，你需要做的就是獲取你應用程式的更新版本，其中將包括新的遷移腳本，並運行 flask db upgrade。Alembic 將檢測到生產資料庫沒有更新到架構的最新修訂版本，並運行在上一個版本之後建立的所有新遷移腳本。

正如我之前提到的，你還有一個 flask db downgrade 命令，它可以撤銷上一次的遷移。雖然你在生產系統上不太可能需要這個選項，但在開發過程中你可能會發現它非常有用。你可能生成了一個遷移腳本並應用它，只是發現你所做的更改並不完全是你需要的。在這種情況下，你可以降級資料庫，刪除遷移腳本，然後生成一個新的來替換它。

## 資料庫關係
關聯式資料庫擅長儲存資料項之間的關係。考慮一個使用者撰寫部落格文章的情況。使用者將在 users 表中擁有一個記錄，而文章將在 posts 表中擁有一個記錄。記錄誰寫了一篇給定文章的最有效方法是連接這兩個相關記錄。

一旦在使用者和文章之間建立了連接，資料庫就可以回答有關這個連接的查詢。最簡單的一個是當你有一篇部落格文章並需要知道是哪個使用者寫的。更複雜的查詢是這個的反向。如果你有一個使用者，你可能想知道這個使用者寫了所有的文章。SQLAlchemy 在這兩種類型的查詢中都有幫助。

讓我們擴展資料庫以儲存部落格文章，以查看關係實際操作。這是新的 posts 表的架構：

![](./img/2024-02-03-00-36-55.png)

posts 表將擁有所需的 id、文章的內容和時間戳。但除了這些預期的欄位外，我還加入了一個 user_id 欄位，它將文章與其作者連接起來。你已經看到所有使用者都有一個唯一的 id 主鍵。將部落格文章連接到其作者的方式是加入對使用者 id 的引用，這正是 user_id 欄位的用途。這個 user_id 欄位被稱為外鍵，因為它引用了另一個表的主鍵。上面的資料庫圖顯示了外鍵作為字段和它索引用的表的 id 欄位之間的連接。這種關係被稱為一對多，因為「一個」使用者寫了「多個」文章。

修改後的 app/models.py 如下所示：

app/models.py：文章資料庫表和關係

```python
from datetime import datetime, timezone
from typing import Optional
import sqlalchemy as sa
import sqlalchemy.orm as so
from app import db

class User(db.Model):
    id: so.Mapped[int] = so.mapped_column(primary_key=True)
    username: so.Mapped[str] = so.mapped_column(sa.String(64), index=True,
                                                unique=True)
    email: so.Mapped[str] = so.mapped_column(sa.String(120), index=True,
                                             unique=True)
    password_hash: so.Mapped[Optional[str]] = so.mapped_column(sa.String(256))

    posts: so.WriteOnlyMapped['Post'] = so.relationship(
        back_populates='author')

    def __repr__(self):
        return '<User {}>'.format(self.username)

class Post(db.Model):
    id: so.Mapped[int] = so.mapped_column(primary_key=True)
    body: so.Mapped[str] = so.mapped_column(sa.String(140))
    timestamp: so.Mapped[datetime] = so.mapped_column(
        index=True, default=lambda: datetime.now(timezone.utc))
    user_id: so.Mapped[int] = so.mapped_column(sa.ForeignKey(User.id),
                                               index=True)

    author: so.Mapped[User] = so.relationship(back_populates='posts')

    def __repr__(self):
        return '<Post {}>'.format(self.body)
```

新的 Post 類別將代表由使用者撰寫的部落格文章。timestamp 欄位使用 datetime 類型提示並配置為索引，這在你想要高效地按時間順序檢索文章時非常有用。我還加入了一個 default 參數，並傳遞了一個返回 UTC 時區當前時間的 lambda 函式。當你將函式作為默認值傳遞時，SQLAlchemy 將設置該欄位為該函式返回的值。通常，在伺服器應用程式中使用 UTC 日期和時間而不是你所在地的本地時間。這確保了無論使用者和伺服器位於何處，你都在使用統一的時間戳。

新的 Post 類別將代表使用者撰寫的部落格文章。timestamp 欄位使用 datetime 類型提示定義，並配置為索引，這在你想要有效地按時間順序檢索文章時非常有用。我還加入了一個預設引數，並傳遞了一個 lambda 函式，該函式返回 UTC 時區的當前時間。當你將函式作為預設值傳遞時，SQLAlchemy 會將欄位設置為該函式返回的值。一般來說，你會想在伺服器應用程式中使用 UTC 日期和時間，而不是你所在地的當地時間。這確保了你使用的時間戳記不論使用者和伺服器位於何處都是統一的。這些時間戳記將在顯示時轉換為使用者的當地時間。

user_id 欄位被初始化為 User.id 的外鍵，這意味著它參考了 users 表的 id 欄位中的值。由於並非所有資料庫都會自動為外鍵建立索引，因此明確加入了 index=True 選項，以便優化基於此欄位的搜尋。

User 類別有一個新的 posts 欄位，使用 so.relationship() 初始化。這不是一個實際的資料庫欄位，但是使用者和文章之間關係的高層次視圖，因此不在資料庫圖表中。同樣，Post 類別有一個也作為關係初始化的 author 欄位。這兩個屬性允許應用程式訪問連接的使用者和文章條目。

so.relationship() 的第一個引數是代表關係另一側的模型類別。這個引數可以作為字串提供，當類別在模組中後面定義時這是必要的。back_populates 引數參考另一側關係屬性的名稱，以便 SQLAlchemy 知道這些屬性指的是同一關係的兩側。

posts 關係屬性使用不同的類型定義。它不使用 so.Mapped，而是使用 so.WriteOnlyMapped，將 posts 定義為一個包含 Post 物件的集合類型。如果這些細節還不太清楚，不用擔心，我將在本文末尾展示這方面的例子。

由於我對應用程式模型進行了更新，需要生成一個新的資料庫遷移：

```bash
(venv) $ flask db migrate -m "posts table"
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.autogenerate.compare] Detected added table 'post'
INFO  [alembic.autogenerate.compare] Detected added index 'ix_post_timestamp' on '['timestamp']'
  Generating /home/miguel/microblog/migrations/versions/780739b227a7_posts_table.py ... done
```

而且遷移需要應用到資料庫：

```bash
(venv) $ flask db upgrade
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade e517276bb1c2 -> 780739b227a7, posts table
```

如果你將你的專案儲存在原始碼控制中，也記得將新的遷移腳本加入到其中。

### 與資料庫互動
我讓你經歷了一個漫長的過程來定義資料庫，但我還沒有展示一切是如何工作的。由於應用程式還沒有任何資料庫邏輯，讓我們在 Python 直譯器中玩玩資料庫以熟悉它。在終端上執行 python 啟動 Python。在開始直譯器之前，確保你的虛擬環境已經啟動。

一旦進入 Python 提示符，讓我們導入應用程式、資料庫實例、模型和 SQLAlchemy 進入點：

```python
>>> from app import app, db
>>> from app.models import User, Post
>>> import sqlalchemy as sa
```

下一步有點奇怪。為了讓 Flask 及其擴充功能可以訪問 Flask 應用程式而不必將 app 作為引數傳入每個函式，必須建立和推送應用程式上下文。應用程式上下文將在教程後面更詳細地介紹，所以現在，請在你的 Python shell 會話中輸入以下程式碼：

```python
>>> app.app_context().push()
```

接下來，建立一個新使用者：

```python
>>> u = User(username='john', email='john@example.com')
>>> db.session.add(u)
>>> db.session.commit()
```

對資料庫的更改是在資料庫會話的上下文中進行的，可以作為 db.session 訪問。會話中可以累積多個更改，一旦所有更改都被註冊，你可以發出單一的 db.session.commit()，將所有更改原子化寫入。如果在處理會話時出現任何錯誤，調用 db.session.rollback() 將中止會話並移除其中儲存的任何更改。重要的是要記住，只有在用 db.session.commit() 發出提交時，更改才會寫入資料庫。會話保證資料庫永遠不會處於不一致狀態。

你是否想知道所有這些資料庫操作是如何知道要使用哪個資料庫的？上面推送的應用程式上下文允許 Flask-SQLAlchemy 訪問 Flask 應用程式實例 app 而不必將其作為引數接收。擴充功能在 app.config 字典中尋找 SQLALCHEMY_DATABASE_URI 項，其中包含資料庫 URL。

讓我們加入另一個使用者：

```python
>>> u = User(username='susan', email='susan@example.com')
>>> db.session.add(u)
>>> db.session.commit()
```

資料庫可以回答返回所有使用者的查詢：

```python
>>> query = sa.select(User)
>>> users = db.session.scalars(query).all()
>>> users
[<User john>, <User susan>]
```

這個例子中的 query 變數被分配了一個基本查詢，選擇所有使用者。這是通過將模型類別傳遞給 SQLAlchemy sa.select() 查詢輔助函式實現的。你會發現大多數資料庫查詢都是從 sa.select() 呼叫開始的。

資料庫會話，在前文中用於定義和提交更動，也用於執行查詢。`db.session.scalars()` 方法執行資料庫查詢並回傳結果迭代器。調用結果物件的 `all()` 方法將結果轉換為普通列表。

在許多情況下，使用結果迭代器進行迴圈遍歷，而不是將其轉換為列表，會更有效率：

```python
users = db.session.scalars(query)
for u in users:
    print(u.id, u.username)
1 john
2 susan
```

注意，當這些使用者被新增時，id 欄位自動設為 1 和 2。這是因為 SQLAlchemy 配置整數主鍵欄位為自動遞增。

這裡是另一種執行查詢的方法。如果你知道某個使用者的 id，你可以如下檢索該使用者：

```python
u = db.session.get(User, 1)
u
<User john>
```

現在讓我們新增一篇部落格文章：

```python
u = db.session.get(User, 1)
p = Post(body='my first post!', author=u)
db.session.add(p)
db.session.commit()
```

我不需要為 timestamp 欄位設定值，因為這個欄位有預設值，你可以在模型定義中看到。那麼 user_id 欄位呢？回想一下我在 Post 類別中建立的 so.relationship，它為文章增加了一個 author 屬性。我使用這個 author 欄位而不是處理使用者 ID 來指派文章的作者。SQLAlchemy 在這方面很棒，因為它提供了高階的抽象，覆蓋了關係和外鍵。

為了完成這個會話，讓我們看幾個更多的資料庫查詢：

```python
## 取得一位使用者所寫的所有文章
u = db.session.get(User, 1)
u
<User john>
query = u.posts.select()
posts = db.session.scalars(query).all()
posts
[<Post my first post!>]

## 同上，但對於沒有文章的使用者
u = db.session.get(User, 2)
u
<User susan>
query = u.posts.select()
posts = db.session.scalars(query).all()
posts
[]

## 為所有文章打印作者和內容
query = sa.select(Post)
posts = db.session.scalars(query)
for p in posts:
    print(p.id, p.author.username, p.body)
1 john my first post!

## 取得所有使用者，按姓名反向字母順序
query = sa.select(User).order_by(User.username.desc())
db.session.scalars(query).all()
[<User susan>, <User john>]

## 取得所有姓名以 "s" 開頭的使用者
query = sa.select(User).where(User.username.like('s%'))
db.session.scalars(query).all()
[<User susan>]
```

注意在上面前兩個例子中，如何使用使用者和文章之間的關係。回想一下，User 模型有一個 posts 關係屬性，這是用 WriteOnlyMapped 泛型型別配置的。這是一種特殊的關係型別，增加了一個 select() 方法，回傳相關項目的資料庫查詢。`u.posts.select()` 表達式負責生成連接使用者和其部落格文章的查詢。

最後一個查詢展示了如何使用條件篩選表格的內容。`where()` 子句用於建立篩

選器，只從所選實體中選取一部分行。在這個例子中，我使用 like() 運算子根據模式選擇使用者。

SQLAlchemy 文檔是學習有關資料庫查詢的許多選項的最佳地方。

最後，退出 Python 殼並使用以下指令清除上面建立的測試使用者和文章，以便資料庫乾淨並準備好進入下一章：

```bash
(venv) $ flask db downgrade base
(venv) $ flask db upgrade
```

第一個指令告訴 Flask-Migrate 以相反順序應用資料庫遷移。當沒有給定目標時，降級命令會降級一個修訂。base 目標會導致所有遷移被降級，直到資料庫處於初始狀態，沒有表格。

升級命令以正向順序重新應用所有遷移。升級的預設目標是 head，這是最近遷移的捷徑。這個命令有效地恢復了上面降級的表格。由於資料庫遷移不保留資料庫中儲存的數據，降級然後升級的效果是快速清空所有表格。

殼上下文
記得你在上一節開始時做了什麼，就在啟動 Python 解釋器後嗎？開始時你輸入了一些導入，然後推送了一個應用上下文：

```python
from app import app, db
from app.models import User, Post
import sqlalchemy as sa
app.app_context().push()
```

當你在應用上工作時，你會經常需要在 Python 殼中測試東西，所以每次都重複上述陳述會變得很繁瑣。這是解決這個問題的好時機。

flask shell 子命令是 flask 指令集中另一個非常有用的工具。shell 命令是 Flask 實現的第二個 “核心” 命令，繼 run 之後。這個命令的目的是在應用上下文中啟動 Python 解釋器。這是什麼意思？看下面的例子：

```bash
(venv) $ python
>>> app
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'app' is not defined
>>>

(venv) $ flask shell
>>> app
<Flask 'app'>
```

在一般的解釋器會話中，除非明確導入，否則不知道 app 符號，但在使用 flask shell 時，該命令會為你預導入應用實例並推送其應用上下文。flask shell 的好處不僅在於它預導入 app，而且你還可以配置 “殼上下文”，即預導入其他符號的列表。

microblog.py 中的以下函式建立了一個殼上下文，將資料庫實例和模型加入到殼會話中：

```python
import sqlalchemy as sa
import sqlalchemy.orm as so
from app import app, db
from app.models import User, Post

@app.shell_context_processor
def make_shell_context():
    return {'db': db, 'User': User, 'Post': Post}
```

此程式碼使用 `app.shell_context_processor` 裝飾器來註冊函式為 shell 上下文函式。當執行 Flask 的 shell 指令時，會呼叫這個函式並將其返回的項目註冊到 shell 會話中。這個函式之所以返回一個字典而不是列表，是因為你需要為每個項目提供一個名稱，以便在 shell 中引用，這個名稱由字典的鍵來指定。

在你加入了 shell 上下文處理器函式之後，你可以在不必引入它們的情況下操作資料庫實體：

```shell
(venv) $ flask shell
>>> db
<SQLAlchemy sqlite:////home/miguel/microblog/app.db>
>>> User
<class 'app.models.User'>
>>> Post
<class 'app.models.Post'>
```

如果你嘗試上述操作並在嘗試訪問 sa、so、db、User 和 Post 時遇到 NameError 異常，那麼 make_shell_context() 函式可能沒有被註冊到 Flask。這最有可能的原因是你沒有在環境中設定 FLASK_APP=microblog.py。如果是這種情況，請回到第一章複習如何設定 FLASK_APP 環境變數。如果你經常忘記在開新的終端機視窗時設定這個變數，你可以考慮在你的專案中加入一個 .flaskenv 檔案，如該章節末尾所述。

