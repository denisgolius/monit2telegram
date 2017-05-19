# monit2telegram

A simple script to send Monit alerts using Telegram bot.

By default, Monit only sends alert notifications via email. But we can also set [a hook to execute a script](https://mmonit.com/monit/documentation/monit.html#action). When executing the script, Monit sets a few environment variables about the alert.

This tiny script transforms those variables into a text message and pipes them to Telegram using a bash script for delivery.

This script is inspired by [monit2telegram](https://github.com/matriphe/monit2telegram).

## Requirements

* Bash
* CURL
* [jq](https://stedolan.github.io/jq/)
* Telegram Bot
* Monit

## Create Telegram Bot

If you don't have a Telegram Bot, just [create one](https://core.telegram.org/bots#create-a-new-bot). By using a Telegram bot you don’t have to use a real Telegram client or reuse your Telegram account. 

### Getting Bot Token

You will get a **Telegram Bot Token** after bot created. Keep this token, we will use it later. The bot token is looked like this.

```nginx
123456789:aBcDeFgHiJkLmN-OpQrStUvWXyZ12345678
```

### Getting Chat ID

To send messages to a Telegram chat, you must first needs to start a chat with the bot. Clicking on the bot link after creation should be enough, it will automatically send a message of `/start` to the bot.

To get the **Chat ID** from Telegram bot, execute this command using [getUpdates](https://core.telegram.org/bots/api#getupdates) function of Telegram API.

```console
$ curl --silent "https://api.telegram.org/bot{TOKEN}/getUpdates" | jq '.'
{
  "result": [
    {
      "message": {
        "entities": [
          {
            "length": 6,
            "offset": 0,
            "type": "bot_command"
          }
        ],
        "text": "/start",
        "date": 1495192923,
        "chat": {
          "type": "private",
          "username": "YourUserName",
          "first_name": "YourName",
          "id": 592043555
        },
        "from": {
          "username": "YourUserName",
          "first_name": "YourName",
          "id": 592043555
        },
        "message_id": 1
      },
      "update_id": 920385551
    },
    {
      "channel_post": {
        "text": "Test01",
        "date": 1495197747,
        "chat": {
          "type": "channel",
          "title": "company_monitoring",
          "id": -1001141383926
        },
        "message_id": 6
      },
      "update_id": 920385558
    }
  ],
  "ok": true
}
```

In this example the **Chat ID** to look out for is **-1001141383926**. Replace `{TOKEN}` with your Telegram bot token.

## Usage

Clone this repo or download the zipped file. 

```console
# git clone https://github.com/vainkop/monit2telegram.git
# cd monit2telegram
```

Put your Telegram Bot ID and Chat ID in `telegramrc` and save it to the `/etc`  directory (`/etc/telegramrc`).

```console
# cp telegramrc /etc/telegramrc
```

Put `sendtelegram.sh` and `monit2telegram.sh` to `/usr/local/bin` and make them executable.

```console
# cp sendtelegram.sh /usr/local/bin/sendtelegram
# chmod +x /usr/local/bin/sendtelegram
# cp monit2telegram.sh /usr/local/bin/monit2telegram
# chmod +x /usr/local/bin/monit2telegram
```

Test the `sendtelegram` script by running this command.

```console
# sendtelegram -c /etc/telegramrc -m "Test01"
Sending message 'Test01' to -1001141383926
Done!
#
```
You should see Telegram message sent by your Telegram bot.

## Set Up Monit

Now you can add Monit alert by adding this line to Monit configuration file.

```google.com.conf
check host www.google.com with address www.google.com
    if failed port 80 protocol http with timeout 5 seconds for 1 cycles then exec "/bin/bash -c ' \
    /usr/local/bin/monit2telegram
    '"
```

## Additional
It is very convenient to use Telegram channels for notifications. 
You just add certain people who are responsible for the service to the corresponding private channel and make your bot send his messages there by chat_id.
Noone but the admin user can respond or send messages to the bot.


You should consider disabling the ability to add your bot to any groups. You can do that by writing /setjoingroups to @BotFather and then Disable.



Best regards,
Valerii Vainkop