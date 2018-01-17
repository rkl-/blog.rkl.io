---
title: 'Simple twitter bot with python'
media_order: snake-390174_1920.jpg
published: true
date: '17-01-2018 16:19'
taxonomy:
    category:
        - blog
    tag:
        - setup
        - coding
        - python
        - twitter
        - bot
---

### Preface

If some of you know my [Twitter](https://twitter.com/rkleinwaechter)
and [Facebook](https://www.facebook.com/romano.kleinwachter) account,
you may wonder how can I have so much time to post all this crypto
currency informations. In fact, I often hear this question.  My answer
then is simple: *"It's my bot, not me."*. If the requesting person have
no technical background, their wondering is interesting. However, for
this ones with a technical background, there is no magic.

It was also not so much effort to do this. However, I will show you how
it is made.

### 1. You need a server

If you don't have already your own server (dedicated or virtual),
so should setup one. The machine you need don't needs a lot of power,
so a simple one core virtual server with at least 1 GB of RAM should do
it's work.  For non european users I suggest to have a look at [Digital
Ocean](https://www.digitalocean.com/). For european people, the german
provider [Domainfactory](https://www.df.eu/de/cloud-hosting/#) is worth
a look.  You'll find for both providers good online documention.

### 2. You need a twitter and facebook account

Or at least twitter. It's nothing more to say here, except that you should
link your twitter account with your facebook account, so that any tweet
is automatically posted on facebook too. This is part of my own setup.

### 3. Lets code the bot

In my case, there are two bots. One for the German [Bitcoin.de
Blog](https://bitcoinblog.de/) and one for the German [BTC Echo
Blog](https://www.btc-echo.de/). Under the hood, this are really trivial
python scripts which gets triggered periodically with two cron jobs of
my users local [crontab](https://en.wikipedia.org/wiki/Cron).

The whole filesystem structure is not more than this:
```
├── bitcoinDe2Twitter.py
├── btcEcho2Twitter.py
├── client
│   ├── __init__.py
│   ├── __init__.pyc
│   ├── __main__.py
│   ├── parameters.py
│   ├── parameters.pyc
│   ├── parameters.py.dist
│   ├── twitter.py
│   └── twitter.pyc
└── storage
    ├── bitcoin_de.pkl
    └── btc_echo.pkl
```

And my crontab looks like this:
```
PYTHONIOENCODING=utf8

*/10 * * * * cd /home/romano/rklCryptoBroadcast && ./btcEcho2Twitter.py
*/15 * * * * cd /home/romano/rklCryptoBroadcast && ./bitcoinDe2Twitter.py
```

If you are familiar with python, the structure is not surprising
to you. The line `PYTHONIOENCODING=utf8` in our crontab is
important, that python can handle UTF-8 strings when it's called by
[cron](https://en.wikipedia.org/wiki/Cron). UTF-8 is sometimes for the 2.7
releases a bit tricky and we use 2.7 for a better library compatibility
and because it's still far more common than the latest 3.x releases.

#### 3.1. Let's have a look into the files.

**bitcoinDe2Twitter.py**
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from client import twitter

twitter.rss2Tweet("https://bitcoinblog.de/feed/", "storage/bitcoin_de.pkl", "BTC_de_Blog")
```

and

**btcEcho2Twitter.py**
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from client import twitter

twitter.rss2Tweet("https://www.btc-echo.de/feed/", "storage/btc_echo.pkl", "BitcoinEcho")
```

This two files are most identical and we see that there is not much
inside. This are simple wrappers which provides an entry point for our
cron job for the two different kinds of blogs we use as source.

The next two important files are under `storage`. `bitcoin_de.pkl`
and `btc_echo.pkl`.  This files are binaries in the
[PKL](https://docs.python.org/2/library/pickle.html) format.  We use this
ones as simple storages for our already tweeted posts from the blogs to
prevent double posts and to not miss some posts.

The first bytes of **bitcoin_de.pkl** looks like this:
```
╭─romano@legend ~/rklCryptoBroadcast  ‹master› 
╰─$ hexdump -C storage/bitcoin_de.pkl |head                                                                                                       1 ↵
00000000  80 02 7d 71 00 28 58 51  00 00 00 68 74 74 70 3a  |..}q.(XQ...http:|
00000010  2f 2f 62 69 74 63 6f 69  6e 62 6c 6f 67 2e 64 65  |//bitcoinblog.de|
00000020  2f 32 30 31 37 2f 30 39  2f 31 38 2f 6e 65 75 2d  |/2017/09/18/neu-|
00000030  62 65 69 2d 61 6f 2d 68  6f 74 65 6c 73 2d 7a 61  |bei-ao-hotels-za|
00000040  68 6c 2d 64 65 69 6e 2d  7a 69 6d 6d 65 72 2d 6d  |hl-dein-zimmer-m|
00000050  69 74 2d 62 69 74 63 6f  69 6e 73 2f 71 01 58 31  |it-bitcoins/q.X1|
00000060  00 00 00 4e 65 75 20 62  65 69 20 41 26 4f 20 48  |...Neu bei A&O H|
00000070  6f 74 65 6c 73 3a 20 5a  61 68 6c 20 64 65 69 6e  |otels: Zahl dein|
00000080  20 5a 69 6d 6d 65 72 20  6d 69 74 20 42 69 74 63  | Zimmer mit Bitc|
00000090  6f 69 6e 73 71 02 58 46  00 00 00 68 74 74 70 73  |oinsq.XF...https|
```

This is a simple form of pythons native object serialization capability.

Lets dig into the `client` folder. The first important file here is
`parameters.py`.  This file contains your twitter API credentials:
```
import tweepy

keys = dict(
    consumer_key =        '<MY-CONSUMER-KEY>',
    consumer_secret =     '<MY-CONSUMER-SECRET>',
    access_token =        '<MY-ACCESS-TOKEN>',
    access_token_secret = '<MY-ACCESS-TOKEN-SECRET>',
)

auth = tweepy.OAuthHandler(keys['consumer_key'], keys['consumer_secret'])
auth.set_access_token(keys['access_token'], keys['access_token_secret'])

twitterAPI = tweepy.API(auth)
```

**SETUP HERE**

As you may noticed here, we use `import
tweepy`. [Tweepy](http://www.tweepy.org/) is a simple
python package which we can use to easily access the [twitter
API](https://developer.twitter.com/en/docs). You need to install tweepy
before the bot can work.

The last important file is `twitter.py`. This is our whole bot
implementation and it has only 75 lines of code.  In the current version,
it looks like this:
```
import time
import pickle
import feedparser
import os.path

from . import parameters


'''
Storage of already tweeted topics.
'''
class TweetStorage:
        _storage = {}
        _pkFile = ''

        def __init__(self, storageFile):
                self._pkFile = storageFile

        def addUrl(self, url, title):
                self._storage[url] = title

        def hasUrl(self, url):
                if url in self._storage:
                        return True
                return False

        def save(self):
                with open(self._pkFile, 'wb') as f:
                        pickle.dump(self._storage, f, pickle.HIGHEST_PROTOCOL)

        def load(self):
                if os.path.isfile(self._pkFile):
                        with open(self._pkFile, 'rb') as f:
                                self._storage = pickle.load(f)
'''
Common tweeter
'''
class Tweeter:
        _storage = None
        _viaUserRef = None

        def __init__(self, storage, viaUserRef):
                self._storage = storage
                self._viaUserRef = viaUserRef
                self._storage.load()

        def sendTweet(self, url, title):
                if not self._storage.hasUrl(url):
                        message = url

                        if self._viaUserRef:
                                message += " via @" + self._viaUserRef

                        parameters.twitterAPI.update_status(message)

                        print("new tweet: \"" + title + "\"")

                        self._storage.addUrl(url, title)
                        self._storage.save()

'''
Tweet from RSS resource
'''
def rss2Tweet(rssFeed, storageFile, viaUserRef):
        data = feedparser.parse(rssFeed)

        tweeter = Tweeter(TweetStorage(storageFile), viaUserRef)

        for i in range(len(data['entries'])):
                title = data['entries'][i].title
                url = data['entries'][i].link

                tweeter.sendTweet(url, title)
                time.sleep(5)
```

As I promised in the beginning of this article, this project is not much
magic and the code is straight forward without any complexity. What we
do is simple. The bot loads the last posts from any of the blogs. Then
it iterates with a delay of 5 seconds over all posts and check if the
current one is already stored in our PKL-File. If not, we post and store
it. That's all.

The other `*.pyc` files are just compiled python binaries after an
initial run of the bot.

The few code of this project can be found on
[Github](https://github.com/rkl-/simple-twitter-bot).
