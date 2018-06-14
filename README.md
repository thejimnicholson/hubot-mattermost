
# hubot-mattermost

[Hubot](https://github.com/github/hubot) adapter for [Mattermost](http://www.mattermost.org/).

This adapter uses the webhook interface and allows for sending [Mattermost Attachments](https://docs.mattermost.com/developer/message-attachments.html).

This version is a fork of and largely based on an adapter written by [Renan Vicente](https://github.com/renanvicente). It includes slash command support
written by [Stéphane Alnet](https://github.com/shimaore).


## Installation

* Install `yo` and `generator-hubot` globally if you haven't already.
* Run `yo hubot` as per the [hubot instructions](https://hubot.github.com/docs/). Specify "shell" as the adapter.
* Once you have a hubot set up, run `npm install git+https://git@github.com/thejimnicholson/hubot-mattermost.git` to install the adapter.
* Create both an [incoming](https://docs.mattermost.com/developer/webhooks-incoming.html)

   Set the __MATTERMOST_INCOME_URL__ environment variable to the URL for the incoming webhook. It will look something like this:

   `http://mattermost.server:8065/hooks/incoming-token`

* Also create an [outgoing](https://docs.mattermost.com/developer/webhooks-outgoing.html) webhook in Mattermost. The URL should look something like this:

   `http://hubot.hostname:8080/hubot/incoming`

   where `hubot.hostname` is the host name or IP address where hubot will run, `8080` is the port that hubot will listen on, and `/hubot/incoming` is the URL where hubot will receive notifications from Mattermost.

   You can override the path portion by setting the __MATTERMOST_ENDPOINT__ environment variable. The port can be set via the __PORT__ envorinment variable.

   Be sure your Mattermost instance is able to reach the host and port where hubot will be running, and that your hubot instance will also be able to reach Mattermost.

* Set the __MATTERMOST_TOKEN__ environment variable to the token that Mattermost provides when you create the outgoing webhook.

   __Note:__ It is possible to have multiple outgoing webhooks, and in fact it is required if you want your hubot instance to respond to messages from different rooms in Mattermost. If you need to do this, you can provide a comma-separated list of tokens in the __MATTERMOST_TOKEN__ variable.


## Example Installation

  ```sh
npm install -g yo generator-hubot
mkdir mybot
cd mybot
yo hubot --adapter shell
npm install ggit+https://git@github.com/thejimnicholson/hubot-mattermost.git
  ```

## Environment variables

The adapter requires the following environment variables to be defined prior to run a Hubot instance:

* `MATTERMOST_ENDPOINT` _string, default: none_ - URI that you want hubot to listen, need to be the uri you specified when creating your outgoing webhook on mattermost. Example: if you create your outgoing webhook with http://127.0.0.1:8080/hubot/incoming you should set it with /hubot/incoming.
* `MATTERMOST_INCOME_URL` _string, default: none_ - Your incoming webhook url. Example: http://<your mattermost instance>:<port>/hooks/ncwc66caqf8d7c4gnqby1196qo
* `MATTERMOST_TOKEN` _string, default: none_ - Token from your outgoing webhook.

In addition, the following optional variables can be set:

* `MATTERMOST_CHANNEL` _string, default: none_ - Override the channel that you want to reply to. This overrides the default channel, but hubot scripts can still target a specific channel if they require it.
* `MATTERMOST_ICON_URL` _string, default: none_ - If Enable Overriding of Icon from Webhooks is enabled you can set a url with the icon that you want for your hubot.
* `MATTERMOST_HUBOT_USERNAME` _string, default: Hubot's name_ - You can set a custom username to respond in mattermost. If Enable Overriding of Usernames from Webhooks, this name is shown in mattermost.
* `MATTERMOST_SELFSIGNED_CERT` _boolean, default: none_ - If true it will ignore if MATTERMOST_ENDPOINT has a self signed certificate.

## Example for Environment variables
  ```sh
export MATTERMOST_ENDPOINT=/hubot/incoming # listen endpoint
export MATTERMOST_CHANNEL=town-square # optional: if you want to override your channel
export MATTERMOST_INCOME_URL=http://<your mattermost instance>:<port>/hooks/ncwc66caqf8d7c4gnqby1196qo # your mattermost income url
export MATTERMOST_TOKEN=oqwx9d4khjra8cw3zbis1w6fqy # your mattermost token
export MATTERMOST_ICON_URL=https://s3-eu-west-1.amazonaws.com/renanvicente/toy13.png # optional: if you want to override hubot icon
export MATTERMOST_HUBOT_USERNAME="matterbot" # optional: if you want to override hubot name
export MATTERMOST_SELFSIGNED_CERT=true # optional: if you want to ignore self signed certificate

  ```

## Sending attachments

Mattermost Attachments are supported. The adapter's `send` method has been extended to allow you to pass a full envelope as the first parameter. For example:

```
robot.respond /give me an attachment/i, (res) ->
  envelope =
    username: 'TheBoss'
    icon_url: 'http://www.someiconfarm.com/theboss.png'
    attachments: [
      color: #000080
      text: "Here's a blue one."
      fallback: "Here's a blue one that you can't see."
    ]
```

## Example with Hubot sending to multiple specific channels only

Although Mattermost doesn't allow multiple channels on a single Incoming/Outgoing hook you can do the following in order to allow Hubot to listen to multiple channels:

* Create an Outgoing Hook for each channel to wish to have Hubot. That will give you multiple tokens
* Set MATTERMOST_TOKEN global variable with multiple tokens separated by comma

Example:
```sh
export MATTERMOST_ENDPOINT=/hubot/incoming # listen endpoint
export MATTERMOST_INCOME_URL=http://localhost:8065/hooks/3eo1wjwyxibnmd5rsusk4h4pgh # your mattermost income url
export MATTERMOST_TOKEN="epboqd78ufyi58nxktgzq9zpho,7ftco7zg5fdkixw7j3okmuo3eo" # your mattermost token for **each Channel**
export MATTERMOST_ICON_URL=https://s3-eu-west-1.amazonaws.com/renanvicente/toy13.png # optional: if you want to override hubot icon
export MATTERMOST_HUBOT_USERNAME="matterbot" # optional: if you want to override hubot name
```

Note that there is ***no*** need to create multiple Incoming Hooks as we can use a single Incoming Hook but specify what channel we want to send the message to as [described in the documentation](http://docs.mattermost.org/integrations/webhooks/Incoming-Webhooks.html).


Run hubot with mattermost adapter.
  ```sh
bin/hubot -a mattermost
  ```

## Example with Hubot sending to ANY public channel

As pointed out by [Andre](https://github.com/devTechi) there's a [new Giphy implementation](https://github.com/mattermost/mattermost-integration-giphy) that leverages an Outgoing hook with **no channel set**, in which Mattermost allows us to send messages to any channel based on **Trigger Words** feature only.

Therefore, if all you want to do is to have Hubot to send/reply to all public channels, all you will need to do is:

* Create an Outgoing Hook, leave Channel untouched (blank), set the <Hubot Name> (e.g. *matterbot*) as Trigger Word and specify the callback URL as you would normally
* Set *MATTERMOST_TOKEN* global variable with the token given by the newly Outgoing hook created

Example of a hook created using this pattern:
```sh
URLs: http://localhost:8080/hubot/incoming
Trigger Words: matterbot
Token: 15r8ybrxhpgifc3rycdjrf6m8e
```

Example of global variables set that will send to **any public channel** if message starts with **matterbot**:
```sh
export MATTERMOST_ENDPOINT=/hubot/incoming # listen endpoint
export MATTERMOST_INCOME_URL=http://localhost:8065/hooks/3eo1wjwyxibnmd5rsusk4h4pgh # your mattermost income url
export MATTERMOST_TOKEN="epboqd78ufyi58nxktgzq9zpho,7ftco7zg5fdkixw7j3okmuo3eo" # your mattermost token
export MATTERMOST_ICON_URL=https://s3-eu-west-1.amazonaws.com/renanvicente/toy13.png # optional: if you want to override hubot icon
export MATTERMOST_HUBOT_USERNAME="matterbot" # optional: if you want to override hubot name
```

### Known issues

With this approach [**Hubot.hear method**](https://hubot.github.com/docs/scripting/#hearing-and-responding) will be invalidated, since a POST message will only be sent from Mattermost if Hubot name (_MATTERMOST_HUBOT_USERNAME="matterbot"_) is mentioned.

So, if you have the following in any of your custom scripts -- that will **no longer work**:

```coffeescript
robot.hear /HEY$/i, (msg) ->
	msg.reply "Yo!"
```

#### Workaround

In order to have both working (send messages to any public channel + hubot actively listening to certain messages) you would need to:

* A) Have an Outgoing Hook with **no channel set** with each 'Hear' (e.g 'hey') regex separated by comma
* B) Have an Outgoing Hook to each Channel you want 'Hear' method to work
* Once you define which option you prefer you must update MATTERMOST_TOKEN with the additional token you got (_MATTERMOST_TOKEN="<token1>,<token2>"), otherwise Hubot will simply ignore the incoming event

Example of Outgoing Hook created using option A:
```sh
URLs: http://localhost:8080/hubot/incoming
Trigger Words: hey
Token: 9r8i6s86hbgc8r57hqc5ywijac
```

By simply typing: *"hey"* in any channel Hubot should be able to respond with *"Yo!"* as Mattermost now sends a POST to Hubot.

## License
The MIT License. See `LICENSE` file.
