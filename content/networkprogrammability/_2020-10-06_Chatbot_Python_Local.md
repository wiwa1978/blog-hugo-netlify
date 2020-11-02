---
title: Cisco Webex Teams - Chatbot with Python
date: 2020-10-01T04:32:50+01:00
draft: true
categories:
  - Network Programming
  - Programming
  - All
tags:
  - Webex Teams
  - Python

---

### Introduction

### Download ngrok or localtunnel

In this post we will be using localtunnel (check out the website [here](https://theboroer.github.io/localtunnel-www/)). The code can be found [here](https://github.com/localtunnel/localtunnel).

Go ahead and install it as follows

```bash
~blog-hugo-netlify-code/WebexTeams_Chatbots/botkit-template ❯ npm install -g localtunnel
```
Next, create a tunnel:

```bash
~blog-hugo-netlify-code/WebexT/python_deckofcards/deckofcards_bot master ❯ lt --port 5005                                                                     
your url is: https://great-eel-95.loca.lt
```

### Create a webex teams bot

![webex](/images/2020-10-06-1.png)


### Add the bot to your Webex Teams space


### Create a webhook


![webex](/images/2020-10-06-2.png)


![webex](/images/2020-10-06-3.png)

![webex](/images/2020-10-06-4.png)

![webex](/images/2020-10-06-5.png)

### Create a Flask application

```python
from flask import Flask, request, json
import requests
from messenger import Messenger

app = Flask(__name__)
port = 5005

msg = Messenger()
local_url = 'https://great-eel-95.loca.lt'

@app.route('/', methods=['GET', 'POST'])
def index():

    """Receive a notification from Webex Teams and handle it"""
    if request.method == 'GET':
        return f'Request received on local port {port}'
    elif request.method == 'POST':
        if 'application/json' in request.headers.get('Content-Type'):
            # Notification payload, received from Webex Teams webhook
            data = request.get_json()

            # Loop prevention, ignore messages which were posted by bot itself.
            # The bot_id attribute is collected from the Webex Teams API
            # at object instatiation.
            
            if msg.bot_id == data.get('data').get('personId'):
                return 'Message from self ignored'
            else:
                # Print the notification payload, received from the webhook
                print(json.dumps(data,indent=4))

                # Collect the roomId from the notification,
                # so you know where to post the response
                # Set the msg object attribute.
                msg.room_id = data.get('data').get('roomId')

                # Collect the message id from the notification, 
                # so you can fetch the message content
                message_id = data.get('data').get('id')

                # Get the contents of the received message. 
                msg.get_message(message_id)

                # If message starts with '/server', relay it to the web server.
                # If not, just post a confirmation that a message was received.
                if msg.message_text.startswith('/cards'):
                    # Default action is to list send the 'status' command.
                    #try:
                    #    action = msg.message_text.split()[1]
                    #except IndexError:
                    #    action = 'status'
                    #headers = {'Content-Type': 'application/x-www-form-urlencoded'}
                    #data = f'action={action}'
                    #web_server = 'http://localhost:5005/'
                    #msg.reply = requests.post(web_server, headers=headers, data=data).text
                    reply = requests.get('https://deckofcardsapi.com/api/deck/new/shuffle/?deck_count=1').json()
                    msg.reply = reply['deck_id']
                    msg.post_message(msg.room_id, msg.reply)
                else:
                    msg.reply = f'Bot received message "{msg.message_text}"'
                    msg.post_message(msg.room_id, msg.reply)

                return data
        else: 
            return ('Wrong data format', 400)

if __name__ == '__main__':
    app.run(host="0.0.0.0", port=port, debug=False)

```

and

```python
import json
import requests

# API Key is obtained from the Webex Teams developers website.
api_key = 'NThiNDY0MDctMzFhYy00NzFmLWE4MmQtZWVhMjVjNzczYjM0MjAyNjIxNzMtZTQ0_PF84_1eb65fdf-9643-417f-9974-ad72cae0e10f'
# Webex Teams messages API endpoint
base_url = 'https://api.ciscospark.com/v1/'

class Messenger():
    def __init__(self, base_url=base_url, api_key=api_key):
        self.base_url = base_url
        self.api_key = api_key
        self.headers = {
            "Authorization": f"Bearer {api_key}",
            "Content-Type": "application/json"
        }
        self.bot_id = requests.get(f'{self.base_url}/people/me', headers=self.headers).json().get('id')

    def get_message(self, message_id):
        """ Retrieve a specific message, specified by message_id """
        received_message_url = f'{self.base_url}/messages/{message_id}'
        self.message_text = requests.get(received_message_url, headers=self.headers).json().get('text')
        print(self.message_text)


    def post_message(self, room_id, message):
        """ Post message to a Webex Teams space, specified by room_id """
        data = {
            "roomId": room_id,
            "text": message,
            }
        post_message_url = f'{self.base_url}/messages'
        post_message = requests.post(post_message_url,headers=self.headers,data=json.dumps(data))
```


### Running our code locally

~blog-hugo-netlify-code/WebexT/p/deckofcards_bot master ❯ python3 chatbot.py                                                                                
 * Serving Flask app "chatbot" (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://0.0.0.0:5005/ (Press CTRL+C to quit)

### Testing

![webex](/images/2020-10-06-6.png)

![webex](/images/2020-10-06-7.png)

[Code](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/WebexTeams_Chatbots/python_deckofcards/deckofcards_bot)