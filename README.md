# Building a basic Telegram Bot using Serverless (Ruby 2.5)

A basic tutorial and sample application that shows how to build a Telegram Bot using AWS Serverless Ruby.

## You need to have a word with the Bot Father!

In Telegram you need to search for the Bot Father (username is `BotFather`). Just search for him and start up the bot (button at the bottom of the chat window).

You will see a bunch of available commands appear on the screen. The one we want to run is `/newbot`. This starts a chat wizard that asks the following questions:

1. Please choose a name for your bot.
1. Now let's choose a username for your bot. It must end in `bot`.

Once you answer the above two questions you will be given an API key to interact with your bot. Keep that API key safe for use later and do not share it with anyone!

## Serverless App Setup

Install serverless

```
npm install -g serverless
```

Create a new application (optionally create a directory for this application boilerplate to live in or poss in `--path my-telegram-bot` as a parameter that will create that directory for you).

```
serverless create --template aws-ruby
```

## Deployment to AWS Lambda

Assuming the `handler.rb` and `serverless.yml` as well as any other files are ready and tested we can deploy this right away!

Make sure that you have your AWS credentials saved (you will need an Admin IAM account + API keys) and if they are not the `default` account then make sure that the `profile` key in `serverless.yml` is set correctly to your account!

Set a local environment variable `TELEGRAM_TOKEN` to the private token that was returned to you earlier by the Bot Father when you created the new bot.

```
-- Replace with your token!
export TELEGRAM_TOKEN="459903168:somthingrandom-z5uLS8"
```

Now run:

```
serverless deploy
```

Once this has deployed, make a note of the endpoints created for your application in AWS, something like:

```
endpoints:
  POST - https://somthingrandom.execute-api.ap-southeast-1.amazonaws.com/dev/hello
```

## Hook up your new Telegram Bot to the new Lambda Function

Now we need to register the Lambda endpoint we saved above with our Telegram Bot as a new Web Hook like so:

```
curl --request POST --url https://api.telegram.org/bot459903168:somthingrandom-z5uLS8/setWebHook --header 'content-type: application/json' --data '{"url": "https://somthingrandom.execute-api.ap-southeast-1.amazonaws.com/dev/hello"}'
```

Remember to replace your Telegram Bot Token in the first URL (that is the string immediately following `/bot` and before `/setWebHook`) and also to replace the data payload 'url' to match your Lambda Function endpoint.

When the above `curl` command executes successfully you should see the following response:

```
{
  "ok": true,
  "result": true,
  "description": "Webhook was set"
}
```

## Try out your new bot in a Telegram Client

When you created your bot, you would have also received a URL which you can invoke to register your Telegram client app with the bot, something like `https://t.me/mybot`

Load your own bot URL in the browser and start playing around with it!

## Testing via CURL

It's possible to test the Telegram API via CURL like so (update the TOKEN and CHAT_ID of course!):

```
curl --request POST --url https://api.telegram.org/bot<TOKEN>/sendMessage --header 'content-type: application/json' --data '{"text":"Wecome Darren! You said, why it not work","chat_id":CHAT_ID}'
```

## Testing locally by running the handler

It's possible to simply test the handler by running the handler directly at the terminal. In `handler.rb` there are some lines below the comment 'Quick Ruby Test' that can be uncommented which effectively call the `hello` method passing in a mocked `event` object. To get this to work you will need to set the `TELEGRAM_TOKEN` environment variable and update the `chat_id` in the mocked event object.