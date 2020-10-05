---
title: Webex Teams Chatbot with Python
date: 2020-06-05T17:19:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
  - All
tags:
  - Webex Teams
  - Python
---
### Introduction
In this post, we will create a very simple chatbot with Webex Teams API using Python. This is a basic example, just to show you the principles a bit.

### Setting up Webex Teams

To start off, go to `https://developer.webex.com/`.

![webex](/images/2020-06-05-1.png)

Next, create a new App (select `Create a Bot`).

![webex](/images/2020-06-05-2.png)

In the details page, add the requested information:

![webex](/images/2020-06-05-3.png)

As a result, you will get back some information which you will need in future so take a note of it. 

![webex](/images/2020-06-05-4.png)

Next, go to Webex Teams and search for your bot based on the bot username. I called my bot blog-wim, so the username will be blogwim@webex.bot.
You will see the bot is listed now in your Webex Teams.

![webex](/images/2020-06-05-5.png)

Next, we need to find out what the room ID is. Therefore, the easiest is to go to https://developer.webex.com/docs/api/v1/rooms/list-rooms to retrieve a list of all the rooms available in your Webex Teams application.

![webex](/images/2020-06-05-6.png)

The first one you will see is the room ID of the chatbot you just added. Also note down the roomID as you will need it later.

### Python code: add message
The following Python script is pretty basic and essentially implements a POST API call to send a message to our Webex Teams bot.

```python
import requests

url = "https://api.ciscospark.com/v1/messages"

room_id = "Y2l***ZDU5"
bearer = "Yzg***10f"

message = "This is a test message from the Python application"

payload = {
    "roomId": room_id, 
    "text": message
    }

headers = {
    "Authorization": "Bearer %s " % bearer
    }

response = requests.post(url, headers=headers, data = payload).json()
print(response)
   
```

Let's execute the Python script:

```
wauterw@WAUTERW-M-65P7 Webex_Chatbot_Messages % python3 webex.py
{'id': 'Y2***mRl', 'roomId': 'Y2l***ZDU5', 'roomType': 'direct', 'text': 'This is a test message from the Python application', 'personId': 'Y2lzY29zc***MGVlYmE', 'personEmail': 'blogwim@webex.bot', 'created': '2020-04-29T14:54:43.380Z'}
```
Next, check your Webex Teams application and you will see the message appears there.

![webex](/images/2020-06-05-7.png)

### Python code: delete messages
```python
import requests

url = "https://api.ciscospark.com/v1/messages"

room_id = "Y2lz***RiZDU5"
bearer = "Yzg2***2e0e10f"

message_url = f"?roomId={room_id}"

headers = {
    "Authorization": "Bearer %s " % bearer
    }

response = requests.get(url + message_url, headers=headers).json()
messages = response['items']
   
for message in messages:
    delete_url = f"/{message['id']}"
    response = requests.delete(url + delete_url, headers=headers)
    if response.status_code == "403":
        print("Message could not be deleted")
        continue
    else:
        print(f"Deleted message with id {message['id']}")
```
That's how easy it is to implement a basic chatbot. In case you want to see the full example, check my repo [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/Webex_Chatbot_Messages).