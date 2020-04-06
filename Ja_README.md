#概要
[Instagram](https://www.instagram.com/)の画像を全部一気にダウンロードするコードを書きました。
ログインや画面スクロールの部分は手動にすることで安定した動作が期待できます。

#デモ動画
こんな感じに動きます。

![instaimgdownload.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/435380/bddaaa8c-30b8-46c7-a5f1-738fa70b65b0.gif)


⚠️Instagramの[利用規約](https://www.facebook.com/help/instagram/478745558852511)では，自動化された手段を用いて情報を取得する行為は禁止されています。
この記事の内容をもとに自動化ツールを作ることはしないでください。

#Seleniumとは
[Selenium](https://www.seleniumhq.org/) とは，Webアプリケーションのテスト自動化に特化した機能を持つツール群です。
詳しくは以前に「[Instagramのフォロワーぜんぶ抜く大作戦〜PythonでSeleniumとPyAutoGUIを使いこなす！！〜](https://qiita.com/ekkyu/items/a3b79b3c6b8fce5b2fe9)」という記事で触れているのでご覧ください。

#準備
Python3が使える環境を用意してください。

`pip install selenium`でSeleniumをインストールします。

[ChromeDriver - WebDriver for Chrome](https://sites.google.com/a/chromium.org/chromedriver/downloads) から
いま使っているChromeのversionに対応したChromeDriverをダウンロードします。
Chromeのversionは，Chromeで`chrome://version/`を検索すると確認できます。便利ですね！

ChromeDriverを最新版にする場合は`brew cask reinstall chromedriver`を実行してください。

#コードの流れ
[1.Instagramへのログイン](#1instagramへのログイン)
[2.画像URLの取得](#2画像URLの取得)
[3.画像のダウンロード](#3画像のダウンロード)

流れなんてどうでもいい，とりあえず動かしたい
という場合は，一番下に[コード全体](#全体コード)を載せてありますのでどうぞ！

##1.Instagramへのログイン

ログインは手動で行ってもらいます。
Instagramアカウントは事前に取得しておいてください。

```python

self.login_time = 20

self.options = webdriver.ChromeOptions()
self.options.add_argument('--no-sandbox')

self.driver = webdriver.Chrome(executable_path='/usr/local/bin/chromedriver', options=self.options)
self.action = ActionChains(self.driver)

def login(self):
    url = "https://www.instagram.com/"
    self.driver.get(url)

    sleep(self.login_time)
```

`options.add_argument('--no-sandbox')`を追加することで、実際にログイン画面が別ウインドウで立ち上がります。

ログインにかかる時間は`login_time = 20`で指定します。今回は20秒としました。



##2.画像URLの取得

続いて、画像のURLを取得します。プロフィールページを一番下まで手動でスクロールしてください。

Instagramの仕様上、画面スクロールに対応して表示される画像を取得するようになっています。

動きのイメージがつかみづらい場合は、（`Command`＋`Option`＋`I`）キーを押すと表示される、`デベロッパーツール`の`Element`パネルを表示させながら、プロフィールページをスクロールしてみてください。

よくわからない方は，いまの画面のまま（`Command`＋`Option`＋`I`）キーを押してみてください。
このページのHTMLソースを見ることができるはずです。


```python

self.img_url_list = []

self.window_width = 0
self.window_height = 0

self.scroll_time = 30

def return_img_pattern(self):
    html_source = self.driver.page_source
    pattern = '><a href="/p/(.*?)/"><div class="'
    results = re.findall(pattern, html_source, re.S)
    return results

def fetch_img_url(self, target_username) -> list:
    url = "https://www.instagram.com/{}".format(target_username)
    self.driver.get(url)
    self.driver.maximize_window()

    for i in range(self.scroll_time):
        sleep(1)
        url_list = self.return_img_pattern()
        new_url = [i for i in url_list if i not in self.img_url_list]
        self.img_url_list.extend(new_url)

    return self.img_url_list
```

スクロールにかかる時間は`scroll_time = 30`で指定します。今回は30秒としました。

1秒ごとにスクロールで変化したHTMLコードを読み取ります。
この際、正規表現のパターンを使用してHTMLコード中から画像URL（の一部）のみを抜き出します。

##3.画像のダウンロード

最後に、取得した画像URLを使用して、画像を一気にダウンロードします。

```python

self.download_img_size = "l"
#                         l: 640×640px
#                         m: 306×306px
#                         t: 150×150px

def download_img(self, url, save_file_path):
    full_url = "https://www.instagram.com/p/" + str(url) +  "/media/?size=" + self.download_img_size
    r = requests.get(full_url, stream=True)

    if r.status_code == 200:
        with open(save_file_path, 'wb') as f:
            f.write(r.content)
```

ダウンロードする画像のサイズは、

- 640×640px
- 306×306px
- 150×150px

の中から選べます。`download_img_size`を指定してください。


#全体コード
```python

import re
import json
import requests
from time import sleep
from selenium import webdriver
from selenium.webdriver.common.action_chains  import ActionChains
from selenium.webdriver.common.keys import Keys

class InstaImgCollector:

    def __init__(self):

        self.img_url_list = []
        self.window_width = 0
        self.window_height = 0
        
        self.login_time = 20
        self.scroll_time = 30
        
        self.download_img_size = "l"
#      　　　　　　　    　　　　　   l: 640×640px
#      　　　　　　　　　    　　  　 m: 306×306px
#       　　　　　　　　　　　    　  t: 150×150px
        
        self.options = webdriver.ChromeOptions()
        self.options.add_argument('--no-sandbox')

        self.driver = webdriver.Chrome(executable_path='/usr/local/bin/chromedriver', options=self.options)
        self.action = ActionChains(self.driver)
    
    def return_img_pattern(self):
        html_source = self.driver.page_source

        pattern = '><a href="/p/(.*?)/"><div class="'
        results = re.findall(pattern, html_source, re.S)
        return results

    def login(self):
        url = "https://www.instagram.com/"
        self.driver.get(url)

        sleep(self.login_time)

    def fetch_img_url(self, target_username) -> list:
        url = "https://www.instagram.com/{}".format(target_username)
        self.driver.get(url)
        self.driver.maximize_window()
        
        for i in range(self.scroll_time):
            sleep(1)
            url_list = self.return_img_pattern()
            new_url = [i for i in url_list if i not in self.img_url_list]
            self.img_url_list.extend(new_url)
        
        return self.img_url_list
        
    def download_img(self, url, save_file_path):
        full_url = "https://www.instagram.com/p/" + str(url) +  "/media/?size=" + self.download_img_size
        r = requests.get(full_url, stream=True)
        
        if r.status_code == 200:
            with open(save_file_path, 'wb') as f:
                f.write(r.content)
    
    def get_post_url_from_id(self, id_):
        self.login()
        
        self.img_url_list = self.fetch_img_url(target_username=id_)

        self.driver.quit()
        return self.img_url_list
    
    def flatten(self, alist):
        return [ flatten for inner in alist for flatten in inner ]


if __name__ == '__main__':
    id_ = "ここにインスタIDを指定する"
    iic = InstaImgCollector()
    url_list = iic.get_post_url_from_id(id_)
    
    for url_i in url_list:
        iic.download_img(url_i, "img/{}.png".format(url_i))
```

#参考
[Seleniumとは](https://e-algorithm.xyz/selenium/)
[Selenium webdriverよく使う操作メソッドまとめ](https://qiita.com/mochio/items/dc9935ee607895420186)
[Google Chromeのバージョンを確認する方法【Mac/Windows共通】](https://digitalnews365.com/chrome-version-check)
