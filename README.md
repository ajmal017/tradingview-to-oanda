# TradingView to OANDA

## Summary
I'm setting up a [web.py](https://webpy.org/) server that can run on PythonAnywhere (uploaded there through GitHub) which listens to TradingView alerts on a URL (i.e. webhook) and posts orders (and sets stop-losses, etc.) on OANDA through their API

## To-do
1. ~~Set up development environment~~
1. ~~Install and set up the web.py server~~
1. ~~Have it listen to POST requests (test with Postman)~~
1. ~~Parse the parameters sent in the request to variables that can be sent along to OANDA~~
1. ~~Authenticate with the OANDA API~~
1. ~~Send a test order to OANDA's paper trader~~
1. ~~Add stop-losses and other risk management techniques to the OANDA call~~
1. ~~Connect the local web.py server with the OANDA API call~~
1. ~~Test if a request from Postman to the web.py server running locally reaches OANDA correctly~~
1. ~~Find out why my limit order didn't go through in the beginning, but cancelled because it reaches the GTD. Maybe the market was closed? If that was the case the market won't move and there won't be any signals, right?~~
1. ~~Add some kind of verification so that only I can send orders through my webhook~~
1. ~~Move the server to PythonAnywhere so that TradingView can reach it~~
1. ~~Setup TradingView to send an alert to the server on PythonAnywhere~~
1. Have the server return the result of a POST request for easier Postman debugging
1. Have the server translate what TradingView sends as `{{ticker}}` and `{{close}}` to what OANDA expects as `instrument` and `price`
1. Make `size` optional with a default value set server-side
1. Add different processing for buy and sell alerts
1. Test if an alert from TradingView reaches OANDA correctly
1. Add a mechanism that notifies me of every order (email, maybe?)
1. Add more alerts to TradingView once I have ironed out the JSON in the alerts

## Prerequisites
* `pip install web.py`
* `conda install requests`
* a `credentials.json` file containing a JSON like this:

```json
{
  "practice_credentials": {
    "oanda_key": "<YOUR PRACTICE ACCOUNT API KEY>",
    "account_id": "<YOUR PRACTICE ACCOUNT ID>"
  },
  "live_credentials": {
    "oanda_key": "<YOUR LIVE ACCOUNT API KEY>",
    "account_id": "<YOUR LIVE ACCOUNT ID>"
  }
}
```

## Authentication
The webhook is only available on `/webhook/<ACCESS_TOKEN>` where `<ACCESS_TOKEN>` is one of the items in the JSON list in `access_tokens.json`. Create that file with the following contents:

```json
[
  "<ACCESS_TOKEN>"
]
```

Make sure to replace `<ACCESS_TOKEN>` with an access token of your choice (e.g. [have DuckDuckGo generate one for you](https://duckduckgo.com/?q=password+64))

Note that these are the only valid endpoints. The server won't tell you anything if you're trying to reach it somewhere else.

## Set up PythonAnywhere
1. Create an account, if you don't have one already
1. `git clone` this repository through a Bash console. (I'll assume you clone into `/home/<USERNAME>/tradingview-to-oanda` for the rest of the setup.)
1. Go to the "Files" tab and upload your `credentials.json` and `access_tokens.json` to the repository's directory
1. Create a virtualenv (I named it "tradingview-to-oanda" and used Python 3.7, see the [docs](https://help.pythonanywhere.com/pages/Virtualenvs)):
  ```
  mkvirtualenv tradingview-to-oanda --python=/usr/bin/python3.7
  ```
1. Open the "Web" tab and create a custom web app
1. Set the following parameters:
  * __Source code__ = The directory you cloned the repository into
  * __Working directory__ = The same directory
  * __Virtualenv__ = The directory of the virtualenv you created, probably something like `/home/<USERNAME>/.virtualenvs/tradingview-to-oanda`
1. Then open the __WSGI configuration file__ linked from the Web tab
1. Comment the `HELLO WORLD` bit (⌘+/ works here)
1. Add this to the bottom:
  ```
  # +++++++++++ web.py +++++++++++
  # Added by me, based on https://help.pythonanywhere.com/pages/WebDotPyWSGIConfig
  import sys
  sys.path.append('/home/<USERNAME>/tradingview-to-oanda')
  from server import app
  application = app.wsgifunc()
  ```
1. Save the file and go back to the Web tab
1. Click the green Reload button
1. Send a valid POST request to https://<USERNAME>.pythonanywhere.com/webhook/<ACCESS_TOKEN> and see if it reaches OANDA

## Add alerts to TradingView
1. Add an alert to whatever condition you want, for example [a custom-built `study`](https://www.tradingview.com/pine-script-docs/en/v4/annotations/Alert_conditions.html)
1. Check the __Webhook URL__ box and paste the URL of your PythonAnywhere webhook including access token
1. Make sure that the __Message__ is a valid JSON containing at least:
  ```json
  {"action":"buy","ticker":"{{ticker}}","close":{{close}}}
  ```
  Note:
  * The server is built in such a way that what TradingView sends as `{{ticker}}` is translated to what OANDA expects as `instrument` and `close` is translated to `price`
  * If you want to sell every position you have for this ticker, use "`sell`" as `action`
