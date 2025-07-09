# Testing Notes


## Pre reqs

1. Python installed on your machine
2. Install the following pip libraries... (discord.py and requests)
        ---- Run this from your terminal "pip install discord.py requests"


## Testing fulfillment.py

1. Simply edit config section to your webhook and email information.

```python
# ──────────────────────────────────────────────
#  ⚙️  CONFIG  ─ update for production
# ──────────────────────────────────────────────
DATABASE_FILE = Path(__file__).with_name("codes.db")
SMTP_HOST     = "smtp.gmail.com"
SMTP_PORT     = 465
SMTP_USER     = "sender@gmail.com"       # email you send from
SMTP_PASS     = "app password"             # app‑password / token
FROM_EMAIL    = "Volt Proxies <sender@gmail.com>"

TEST_MODE     = True                       # set False in prod
TEST_EMAIL    = ""       # where test mails go

ADMIN_WEBHOOK_URL = ""
WEBHOOK_ENABLED = True
```

2. Run the script with Python via terminal: "python3 fulfillment.py"

3. If you have set up everything correctly, you should see a redemption code being generated for 2gb of data and a message sent to your Discord webhook.
(if the database doesn't exist, it will be created automatically on first run)


***if you can create a nice email template that would be AMAZING. Feel free to use html instead of a basic text email if you want. Can use things like GPT to help get the vibe you want and modify it from there***




#
#


## Testing redeem_bot.py

1. Edit the config section with your discord webhook url, your discord bot token, and the channel ID you want the panel to automatically post to on run.

```python
# ───────────────────────
# ⚙️  CONFIG
# ───────────────────────
ADMIN_REDEEM_WEBHOOK_URL = ""
TOKEN            = ""         # discord bot token
CODES_DB         = "codes.db"                         # same file used by fulfillment.py
API_ENDPOINT     = "https://www.test.com/endpoint"    # api endpoint to update plan data
REDEEM_CHANNEL   =   # channel where redeem panel is posted (posted automatically upon bot startup)
```

2. Run the script with Python via terminal: "python3 redeem_bot.py"

3. If you have it setup correctly you should see the output showing the discord bot has logged in successfully, and the panel should appear in the channel set via config. Upon redemption of a code the database will be updated and the webhook will be sent to admins. The api call isn't set yet so a failure to post error is expected behavior.


***If you can improve this panel to be more detailed and visually pleasing that would be incredibly helpful***
