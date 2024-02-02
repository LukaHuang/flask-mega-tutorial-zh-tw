## Chapter 10: Email Support

> Posted by on [Miguel Grinberg](https://blog.miguelgrinberg.com/author/Miguel%20Grinberg)

這是 Flask Mega-Tutorial 系列的第十部分，在本章中，我將告訴你如何讓你的應用程式向使用者發送電子郵件，以及如何在電子郵件支持的基礎上構建一個密碼恢復功能。

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

現在，應用程式在資料庫方面做得相當好，所以在這一章中，我想離開這個主題，加入另一個大多數網路應用程式需要的重要部分，那就是發送電子郵件。

為什麼應用程式需要向使用者發送電子郵件？原因有很多，但一個常見的原因是解決與身份驗證相關的問題。在本章中，我將為那些忘記密碼的使用者加入一個密碼重置功能。當使用者請求重置密碼時，應用程式將發送一封包含特製連結的電子郵件。然後，使用者需要點擊該連結以訪問一個設置新密碼的表單。

本章的 GitHub 連結為：[Browse](https://github.com/miguelgrinberg/microblog/tree/v0.10) [Zip](https://github.com/miguelgrinberg/microblog/archive/v0.10.zip) [Diff](https://github.com/miguelgrinberg/microblog/compare/v0.9...v0.10)


### Flask-Mail 簡介
就實際發送電子郵件而言，Flask 有一個受歡迎的擴充功能稱為 Flask-Mail。像往常一樣，這個擴充功能可以通過 pip 安裝：

```bash
(venv) $ pip install flask-mail
```
密碼重置連結將包含一個安全令牌。為了生成這些令牌，我將使用 JSON Web Tokens，它也有一個受歡迎的 Python 套件：

```bash
(venv) $ pip install pyjwt
```
Flask-Mail 擴充功能是通過 app.config 物件進行配置的。還記得在第 7 章中，我加入了電子郵件配置，用於在生產中發生錯誤時給自己發送電子郵件嗎？我當時沒有告訴你，但我的配置變數選擇是根據 Flask-Mail 的要求模型化的，所以實際上並不需要任何額外的工作，應用程式中已經有了配置變數。

像大多數 Flask 擴充功能一樣，你需要在建立 Flask 應用程式後立即建立一個實例。在這種情況下，這是一個 Mail 類的物件：

app/__init__.py: Flask-Mail 實例。

```python
## ...
from flask_mail import Mail

app = Flask(__name__)
## ...
mail = Mail(app)
```
如果你打算測試發送電子郵件，你有我在第 7 章中提到的相同選項。如果你想使用模擬電子郵件伺服器，那麼你可以在第二個終端中啟動相同的 SMTP 調試伺服器，使用以下命令：

```bash
(venv) $ aiosmtpd -n -c aiosmtpd.handlers.Debugging -l localhost:8025
```
要配置應用程式使用這個伺服器，你需要設置兩個環境變數：

```bash
(venv) $ export MAIL_SERVER=localhost
(venv) $ export MAIL_PORT=8025
```
如果你更喜歡讓電子郵件真正發送，你需要使用一個真正的電子郵件伺服器。如果你有一個，那麼你只需要為它設置 MAIL_SERVER、MAIL_PORT、MAIL_USE_TLS、MAIL_USERNAME 和 MAIL_PASSWORD 環境變數。如果你想要一個快速的解決方案，你可以使用 Gmail 帳戶來發送電子郵件，使用以下設置：

```bash
(venv) $ export MAIL_SERVER=smtp.googlemail.com
(venv) $ export MAIL_PORT=587
(venv) $ export MAIL_USE_TLS=1
(venv) $ export MAIL_USERNAME=<your-gmail-username>
(venv) $ export MAIL_PASSWORD=<your-gmail-password>
```
如果你正在使用 Microsoft Windows，你需要在上述每個 export 語句中將 export 替換為 set。

不幸的是，你的 Gmail 帳戶中的安全功能可能會阻止應用程式通過它發送電子郵件。一些帳戶允許當你明確允許 “較不安全的應用程式” 訪問你的 Gmail 帳戶時這樣做，但這並不總是可用的。你可以在這裡閱讀有關此事的訊息。

如果你想使用一個真正的電子郵件伺服器，但不想為 Gmail 配置自己添麻煩，SendGrid 是一個不錯的選擇，它允許你使用免費帳戶每天發送 100 封電子郵件。

### Flask-Mail 使用方式
為了學習 Flask-Mail 的工作方式，我將向你展示如何從 Python shell 會話中發送一封電子郵件。啟動帶有 flask shell 的 Python，然後運行以下命令：

```python
>>> from flask_mail import Message
>>> from app import mail
>>> msg = Message('test subject', sender=app.config['ADMINS'][0],
... recipients=['your-email@example.com'])
>>> msg.body = 'text body'
>>> msg.html = '<h1>HTML body</h1>'
>>> mail.send(msg)
```
上面的程式碼片段將向你放在 recipients 參數中的電子郵件地址列表發送電子郵件。我將發件人設為第一個配置的管理員（我在第 7 章中加入了 ADMINS 配置變數）。電子郵件將有純文本和 HTML 版本，因此根據你的電子郵件客戶端的配置，你可能會看到其中一個或另一個。

現在讓我們將電子郵件整合到應用程式中。

### 簡單的電子郵件框架
我將從編寫一個發送電子郵件的輔助函數開始，這基本上是上一節中 shell 練習的泛化版本。我將這個函數放在一個名為 app/email.py 的新模組中：

app/email.py: 電子郵件發送包裝函數。

```python
from flask_mail import Message
from app import mail

def send_email(subject, sender, recipients, text_body, html_body):
    msg = Message(subject, sender=sender, recipients=recipients)
    msg.body = text_body
    msg.html = html_body
    mail.send(msg)
```

Flask-Mail 支援我這裡沒有使用的一些功能，例如副本和密件副本列表。如果你對這些選項感興趣，務必查看 Flask-Mail 文本。

#### 請求重設密碼

如我上面所提到的，我希望使用者有選項要求重設他們的密碼。為此目的，我將在登入頁面加入一個連結：

app/templates/login.html：登入表單中的密碼重設連結。

```html
<p>
    忘記你的密碼了嗎？
    <a href="{{ url_for('reset_password_request') }}"> 點擊這裡重設 </a>
</p>
```

當使用者點擊連結時，一個新的網頁表單將出現，要求使用者的電子郵件地址，作為啟動密碼重設流程的方式。這是表單類別：

app/forms.py：重設密碼請求表單。

```python
class ResetPasswordRequestForm(FlaskForm):
    email = StringField('Email', validators=[DataRequired(), Email()])
    submit = SubmitField('請求重設密碼')
```

這是相應的 HTML 範本：

app/templates/reset_password_request.html：重設密碼請求範本。

```html
{% extends "base.html" %}

{% block content %}
    <h1> 重設密碼 </h1>
    <form action=""method="post">
        {{form.hidden_tag() }}
        <p>
            {{form.email.label}}<br>
            {{form.email(size=64) }}<br>
            {% for error in form.email.errors %}
            <span style="color: red;">[{{ error }}]</span>
            {% endfor %}
        </p>
        <p>{{ form.submit() }}</p>
    </form>
{% endblock %}
```

我還需要一個視圖函式來處理這個表單：

app/routes.py：重設密碼請求視圖函式。

```python
from app.forms import ResetPasswordRequestForm
from app.email import send_password_reset_email

@app.route('/reset_password_request', methods=['GET', 'POST'])
def reset_password_request():
    if current_user.is_authenticated:
        return redirect(url_for('index'))
    form = ResetPasswordRequestForm()
    if form.validate_on_submit():
        user = db.session.scalar(
            sa.select(User).where(User.email == form.email.data))
        if user:
            send_password_reset_email(user)
        flash('請檢查你的電子郵件以獲取重設密碼的指示')
        return redirect(url_for('login'))
    return render_template('reset_password_request.html',
                           title='重設密碼', form=form)
```

這個視圖函式與處理表單的其他函式非常相似。我首先確保使用者未登入。如果使用者已經登入，則使用密碼重設功能沒有意義，所以我重定向到索引頁面。

當表單提交且有效時，我按表單中使用者提供的電子郵件查找使用者。如果我找到了使用者，我就發送一封重設密碼的電子郵件。send_password_reset_email() 輔助函式執行此任務。接下來我會向你展示這個函式。

在電子郵件發送後，我會顯示一個消息，指引使用者查看電子郵件以獲取進一步指示，然後重新導向回登入頁面。你可能會注意到，即使使用者提供的電子郵件未知，也會顯示閃現的消息。這是為了客戶無法使用此表單來確定某

個使用者是否是會員。

#### 密碼重設令牌 Password Reset Tokens

在我實現 send_password_reset_email() 函式之前，我需要一種生成密碼請求連結的方式。這將是透過電子郵件發送給使用者的連結。當點擊該連結時，將向使用者展示一個可以設置新密碼的頁面。這個計劃的棘手之處是確保只有有效的重設連結可以用來重設帳戶的密碼。

這些連結將附帶一個令牌，並且在允許更改密碼之前會驗證此令牌，作為請求電子郵件的使用者能夠訪問帳戶上的電子郵件地址的證明。這種類型流程的一個非常流行的令牌標準是 JSON Web Token，或 JWT。JWT 的好處是它們是自包含的。你可以在電子郵件中向使用者發送一個令牌，當使用者點擊將令牌反饋給應用程式的連結時，可以自行驗證它。

JWT 是如何工作的？沒有什麼比快速的 Python shell 會話更能理解它們了：

```python
>>> import jwt
>>> token = jwt.encode({'a': 'b'}, 'my-secret', algorithm='HS256')
>>> token
'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJhIjoiYiJ9.dvOo58OBDHiuSHD4uW88nfJik_sfUHq1mDi4G0'
>>> jwt.decode(token, 'my-secret', algorithms=['HS256'])
{'a': 'b'}
```

`{'a': 'b'}`` 字典是要寫入令牌的一個範例負載。為了使令牌安全，需要提供一個秘密金鑰，用於建立加密簽名。在這個範例中，我使用了字串 `'my-secret'`，但對於應用程式，我將使用 Flask 配置中的 SECRET_KEY。algorithm 參數指定如何生成令牌簽名。HS256 算法是最廣泛使用的。

正如你所見，結果令牌是一長串字符。但不要以為這是一個加密令牌。令牌的內容，包括負載，可以輕鬆地被任何人解碼（不相信我嗎？複製上面的令牌，然後在 JWT 調試器中輸入以查看其內容）。令牌安全的是負載已經簽名。如果有人試圖偽造或篡改令牌中的負載，那麼簽名將被無效，並且要生成新的簽名需要秘密金鑰。當一個令牌被驗證時，負載的內容被解碼並返回給呼叫者。如果令牌的簽名被驗證，那麼負載可以被信任為真實的。

我打算用於密碼重設令牌的有效負載將會是這樣的格式：`{'reset_password': user_id, 'exp': token_expiration}`。這裡的 `exp` 欄位是 JWT 的標準，如果出現它，表示令牌的到期時間。如果一個令牌有有效的簽名，但已經過了它的到期時間戳記，那麼它也會被視為無效。對於密碼重設功能，我將給這些令牌 10 分鐘的壽命。

當使用者點擊電子郵件中的連結時，令牌將作為 URL 的一部分被發送回應用程式，處理這個 URL 的視圖函式首先要做的就是驗證它。如果簽名有效，則可以通過有效負載中儲存的 ID 識別使用者。一旦知道了使用者的身份，應用程式就可以要求設定一個新密碼並將其設定在使用者帳號上。

#### 令牌的生成與驗證

由於這些令牌屬於使用者，我將會把令牌生成和驗證函式寫成 `User` 模型中的方法：

```python
## app/models.py: 重設密碼令牌方法

from time import time
import jwt
from app import app

class User(UserMixin, db.Model):
    ## ...

    def get_reset_password_token(self, expires_in=600):
        return jwt.encode(
            {'reset_password': self.id, 'exp': time() + expires_in},
            app.config['SECRET_KEY'], algorithm='HS256')

    @staticmethod
    def verify_reset_password_token(token):
        try:
            id = jwt.decode(token, app.config['SECRET_KEY'],
                            algorithms=['HS256'])['reset_password']
        except:
            return
        return db.session.get(User, id)
```

`get_reset_password_token()` 函式返回一個 JWT 令牌，這個令牌是直接由 `jwt.encode()` 函式生成的字符串。

`verify_reset_password_token()` 是一個靜態方法，這意味著它可以直接從類中調用。靜態方法類似於類方法，唯一的區別是靜態方法不接收類作為第一個參數。這個方法接收一個令牌，嘗試通過調用 PyJWT 的 `jwt.decode()` 函式來解碼它。如果令牌無法驗證或已過期，將引發異常，在這種情況下我會捕捉它以防止錯誤，然後向呼叫者返回 None。如果令牌有效，則令牌有效負載中 `reset_password` 鍵的值是使用者的 ID，所以我可以加載使用者並返回它。

#### 發送密碼重設電子郵件

`send_password_reset_email()` 函式依賴於我之前寫的 `send_email()` 函式來生成密碼重設電子郵件。

```python
## app/email.py: 發送密碼重設電子郵件函式

from flask import render_template
from app import app



## ...

def send_password_reset_email(user):
    token = user.get_reset_password_token()
    send_email('[Microblog] 重設您的密碼',
               sender=app.config['ADMINS'][0],
               recipients=[user.email],
               text_body=render_template('email/reset_password.txt',
                                         user=user, token=token),
               html_body=render_template('email/reset_password.html',
                                         user=user, token=token))
```

這個函式有趣的部分是，電子郵件的文本和 HTML 內容是使用熟悉的 `render_template()` 函式從範本生成的。範本接收使用者和令牌作為參數，以便生成個性化的電子郵件訊息。

為了區分電子郵件範本和常規 HTML 範本，我們在 templates 中建立一個 email 子目錄：

```bash
(venv) $ mkdir app/templates/email
```

以下是重設密碼電子郵件的文本範本：

```python
## app/templates/email/reset_password.txt: 密碼重設電子郵件的文本

親愛的 {{user.username}}，

點擊以下連結以重設您的密碼：

{{url_for('reset_password', token=token, _external=True) }}

如果您沒有要求重設密碼，請忽略此訊息。

誠摯地，

Microblog 團隊
```

以下是相同電子郵件的更好的 HTML 版本：

```html
## app/templates/email/reset_password.html: 密碼重設電子郵件的 HTML

<!doctype html>
<html>
    <body>
        <p> 親愛的 {{ user.username }}，</p>
        <p>
            要重設您的密碼，
            <a href="{{ url_for('reset_password', token=token, _external=True) }}">
                點擊這裡
            </a>。
        </p>
        <p> 或者，您可以將以下連結貼到您瀏覽器的地址欄：</p>
        <p>{{ url_for('reset_password', token=token, _external=True) }}</p>
        <p> 如果您沒有要求重設密碼，請忽略此訊息。</p>
        <p> 誠摯地，</p>
        <p>Microblog 團隊 </p>
    </body>
</html>
```

在這兩個電子郵件範本中引用的 `reset_password` 路由在 `url_for()` 調用中還不存在，這將在下一節中加入。我在兩個範本中的 `url_for()` 調用中包含的_external=True 參數也是新的。`url_for()` 生成的 URL 默認是相對 URL，只包括 URL 的路徑部分。這對於在網頁中生成的連結通常是足夠的，因為網頁瀏覽器會通過從地址欄中的 URL 獲取缺失的部分來完成 URL。然而，通過電子郵件發送 URL 時，這種上下文並不存在，所以需要使用完整的 URL。當作為參數傳遞_external=True 時，會生成完整的 URL，所以前面的例子會返回 http://localhost:5000/user/susan，或者當應用程式部署在域名上時的適當 URL。

### 重設使用者密碼

當使用者點擊電子郵件連結時，將觸發與此功能相關的第二個路由。以下是密碼請求視圖函式：

app/routes.py: 密碼重設視圖函式。

```python
from app.forms import ResetPasswordForm

@app.route('/reset_password/<token>', methods=['GET', 'POST'])
def reset_password(token):
    if current_user.is_authenticated:
        return redirect(url_for('index'))
    user = User.verify_reset_password_token(token)
    if not user:
        return redirect(url_for('index'))
    form = ResetPasswordForm()
    if form.validate_on_submit():
        user.set_password(form.password.data)
        db.session.commit()
        flash('Your password has been reset.')
        return redirect(url_for('login'))
    return render_template('reset_password.html', form=form)
```

在這個視圖函式中，我首先確保使用者未登入，然後通過調用 User 類中的 token 驗證方法來確定使用者身份。這個方法如果 token 有效則返回使用者，如果無效則返回 None。如果 token 無效，我會重定向到首頁。

如果 token 有效，則我向使用者展示第二個表單，要求輸入新密碼。這個表單的處理方式與之前的表單類似，如果表單提交有效，我會調用 User 的 set_password() 方法來更改密碼，然後重定向到登入頁面，使用者現在可以登入了。

這是 ResetPasswordForm 類：

app/forms.py: 密碼重設表單。

```python
class ResetPasswordForm(FlaskForm):
    password = PasswordField('Password', validators=[DataRequired()])
    password2 = PasswordField(
        'Repeat Password', validators=[DataRequired(), EqualTo('password')])
    submit = SubmitField('Request Password Reset')
```

這是相應的 HTML 模板：

app/templates/reset_password.html: 密碼重設表單模板。

```html
{% extends "base.html" %}

{% block content %}
    <h1>Reset Your Password</h1>
    <form action=""method="post">
        {{form.hidden_tag() }}
        <p>
            {{form.password.label}}<br>
            {{form.password(size=32) }}<br>
            {% for error in form.password.errors %}
            <span style="color: red;">[{{ error }}]</span>
            {% endfor %}
        </p>
        <p>
            {{form.password2.label}}<br>
            {{form.password2(size=32) }}<br>
            {% for error in form.password2.errors %}
            <span style="color: red;">[{{ error }}]</span>
            {% endfor %}
        </p>
        <p>{{ form.submit() }}</p>
    </form>
{% endblock %}
```

密碼重設功能現已完成，請確保嘗試它。

### 非同步電子郵件

如果你使用的是調試電子郵件伺服器，你可能沒有注意到這一點，但實際發送電子郵件會大大減慢應用程式。發送電子郵件時需要進行的所有交互使任務變慢，通常需要幾秒鐘才能發出一封電子郵件，如果收件人的電子郵件伺服器較慢，或者有多個收件人，可能需要更長時間。

我真正想要的是 send_email() 函式變為非同步。這是什麼意思？這意味著當調用此函式時，發送電子郵件的任務被安排在後台進行，釋放 send_email() 函式立即返回，以便應用程式可以在發送電子郵件的同時繼續運行。

Python 支持運行非同步任務，實際上有不止一種方式。threading 和 multiprocessing 模組都可以做到這一點。為發送的電子郵件啟動一個背景執行緒比啟動一個新進程資源消耗要少得多，所以我選擇了這種方法：

app/email.py: 非同步發送電子郵件。

```python
from threading import Thread
## ...

def send_async_email(app, msg):
    with app.app_context():
        mail.send(msg)


def send_email(subject, sender, recipients, text_body, html_body):
    msg = Message(subject, sender=sender, recipients=recipients)
    msg.body = text_body
    msg.html = html_body
    Thread(target=send_async_email, args=(app, msg)).start()
```

`send_async_email` 函式現在在一個背景執行緒中運行，通過 send_email() 的最後一行中的 Thread 類調用。隨著這個變化，發送電子郵件將在執行緒中運行，當進程完成時，執行緒將結束並自行清理。如果你配置了一個真正的電子郵件伺服器，當你按下密碼重設請求表單上的提交按鈕時，你肯定會注意到速度的提升。

你可能期望只有 msg 參數被發送到執行緒，但正如你在程式碼中看到的，我還發送了應用程式實例。在使用執行緒時，需要牢記 Flask 的一個重要設計方面。Flask 使用上下文來避免跨函式傳遞參數。我不打算在這裡詳細介紹，但你要知道有兩種類型的上下文，應用程式上下文和請求上下文。在大多數情況下，這些上下文由 Flask 自動管理，但當應用程式啟動自定義執行緒時，可能需要手動建立這些執行緒的上下文。

許多擴展需要設置應用程式上下文才能工作，因為這使它們能夠在不將其作為參數傳遞的情況下找到 Flask 應用程式實例。許多擴展需要知道應用程式實例的原因是因為它們的配置存儲在 app.config 對象中。這正是 Flask-Mail 的情況。mail.send() 方法需要訪問電子郵件伺服器的配置值，這只能通過知道應用程式是什麼來做到。通過 with app.app_context() 調用建立的應用程式上下文使當前應用程式變數從 Flask 中可訪問。

