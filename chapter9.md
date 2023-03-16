Chapter 9: Direct messaging:

In this chapter, we'll implement the direct messaging feature, allowing users to send and receive private messages.

9.1 Create a Message model to store message data

Start by creating a Message model in your models.py file:

from django.db import models
from django.contrib.auth.models import User

class Message(models.Model):
    sender = models.ForeignKey(User, on_delete=models.CASCADE, related_name='sent_messages')
    recipient = models.ForeignKey(User, on_delete=models.CASCADE, related_name='received_messages')
    content = models.TextField(max_length=1000)
    timestamp = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return f"{self.sender.username} -> {self.recipient.username}: {self.content[:20]}..."

9.2 Use Django Rest Framework to create API endpoints to handle sending and receiving messages

Create a serializer for the Message model:

# serializers.py
from rest_framework import serializers
from .models import Message

class MessageSerializer(serializers.ModelSerializer):
    class Meta:
        model = Message
        fields = ['id', 'sender', 'recipient', 'content', 'timestamp']

Now, create a view to handle creating and listing messages between users:

# views.py
from rest_framework import generics
from .models import Message
from .serializers import MessageSerializer

class MessageView(generics.ListCreateAPIView):
    queryset = Message.objects.all()
    serializer_class = MessageSerializer

    def get_queryset(self):
        user = self.request.user
        recipient_id = self.request.query_params.get('recipient_id', None)

        if recipient_id:
            return Message.objects.filter(
                (models.Q(sender=user) & models.Q(recipient__id=recipient_id)) |
                (models.Q(sender__id=recipient_id) & models.Q(recipient=user))
            ).order_by('timestamp')
        return Message.objects.none()

Add the new view to your urlpatterns:

# urls.py
from django.urls import path
from .views import MessageView

urlpatterns = [
    path('messages/', MessageView.as_view(), name='messages'),
]

9.3 Use Django Channels to enable real-time messaging

To enable real-time messaging, we'll use Django Channels. Begin by installing Django Channels:

pip install channels

Create a consumers.py file in your theeai app directory:


import json
from channels.generic.websocket import AsyncWebsocketConsumer
import asyncio

class MessageConsumer(AsyncWebsocketConsumer):
    async def connect(self):
        self.room_group_name = f"messages_{self.scope['user'].id}"

        await self.channel_layer.group_add(
            self.room_group_name,
            self.channel_name
        )

        await self.accept()

    async def disconnect(self, close_code):
        await self.channel_layer.group_discard(
            self.room_group_name,
            self.channel_name
        )

    async def receive(self, text_data):
        text_data_json = json.loads(text_data)
        message = text_data_json['message']

        await self.channel_layer.group_send(
            self.room_group_name,
            {
                'type': 'chat_message',
                'message': message
            }
        )

    async def chat_message(self, event):
        message = event['message']

        await self.send(text_data=json.dumps({
            'message': message
        }))

In your project's settings, configure Django Channels:

# settings.py
INSTALLED_APPS = [
    ...
    'channels',
]

# Use the channels layer as the default backend for Django's ASGI interface
ASGI_APPLICATION = 'myproject.routing.application


Create a `routing.py` file in your project's root directory and configure the routing:

# routing.py
from channels.auth import AuthMiddlewareStack
from channels.routing import ProtocolTypeRouter, URLRouter
import theeai.routing

application = ProtocolTypeRouter({
    'http': get_asgi_application(),
    'websocket': AuthMiddlewareStack(
        URLRouter(
            theeai.routing.websocket_urlpatterns
        )
    ),
})

Create a routing.py file in your theeai app directory:


# theeai/routing.py
from django.urls import path
from . import consumers

websocket_urlpatterns = [
    path('ws/messages/', consumers.MessageConsumer.as_asgi()),
]

9.4 Implement front-end components to send and receive messages

Create a new React component to handle sending and receiving messages:
// MessageList.js
import React, { useEffect, useState } from 'react';
import { ListGroup } from 'react-bootstrap';

const MessageList = ({ messages }) => {
  return (
    <ListGroup>
      {messages.map((message, index) => (
        <ListGroup.Item key={index}>
          {message.sender.username}: {message.content}
        </ListGroup.Item>
      ))}
    </ListGroup>
  );
};

export default MessageList;

Create another React component to handle the chat input:


// ChatInput.js
import React, { useState } from 'react';
import { InputGroup, FormControl, Button } from 'react-bootstrap';

const ChatInput = ({ onSend }) => {
  const [message, setMessage] = useState('');

  const handleSend = () => {
    if (message) {
      onSend(message);
      setMessage('');
    }
  };

  return (
    <InputGroup>
      <FormControl
        placeholder="Type your message..."
        value={message}
        onChange={(e) => setMessage(e.target.value)}
        onKeyPress={(e) => {
          if (e.key === 'Enter') {
            handleSend();
          }
        }}
      />
      <InputGroup.Append>
        <Button onClick={handleSend}>Send</Button>
      </InputGroup.Append>
    </InputGroup>
  );
};

export default ChatInput;

Update your main component to include the new components and handle WebSocket connections:

import React, { useEffect, useState } from 'react';
import MessageList from './MessageList';
import ChatInput from './ChatInput';

const ChatApp = () => {
  const [messages, setMessages] = useState([]);
  const [socket, setSocket] = useState(null);

  useEffect(() => {
    const ws = new WebSocket('ws://localhost:8000/ws/messages/');
    ws.onmessage = (e) => {
      const data = JSON.parse(e.data);
      setMessages((messages) => [...messages, data.message]);
    };
    setSocket(ws);
    return () => ws.close();
  }, []);

  const handleSend = (message) => {
    if (socket) {
      socket.send(JSON.stringify({ message }));
    }
  };

  return (
    <div>
      <MessageList messages={messages} />
      <ChatInput onSend={handleSend} />
    </div>
  );
};

export default ChatApp;

Now, users can send and receive messages in real time. You have successfully implemented direct messaging in your web application!

At this point, you have a fully functional chat application where users can send and receive messages in real-time. You can now work on adding more features and refining the user experience. Here are a few ideas to consider:

1. User authentication: Implement user authentication using Django's built-in authentication system or a third-party library like django-allauth. This way, users can log in, sign up, and have their own unique usernames.

2. Private chat rooms: Allow users to create private chat rooms where they can invite others to join. You'll need to modify your WebSocket consumers, models, and routing to handle chat rooms and manage access.

3. Message history: Store messages in the database so users can view the chat history when they log in or refresh the page. You can use Django models and serializers to store and retrieve messages.

4. Online status: Display a user's online status by tracking when they connect and disconnect from the WebSocket. You can achieve this by modifying the WebSocket consumers to manage user connections and updating the front-end components to show the online status.

5. Typing indicators: Show typing indicators when users are typing a message. You can send a WebSocket message from the client to the server when the user starts and stops typing, and then update the front-end components to display the typing indicators.

6. Message read receipts: Add message read receipts to show when a message has been read by the recipient. You can update the message model to include a read status, and then modify the WebSocket consumers to handle read receipt updates.

7. Notifications: Implement real-time notifications for new messages or chat room invitations. You can use WebSockets to send notifications to the user's browser and display them using a front-end component.

8. File sharing: Allow users to share files, images, and other media in chat rooms. You can use Django's file handling capabilities to store and serve files, and update the front-end components to handle file uploads and display the shared files.

As you continue to develop your chat application, consider performance and scalability issues that may arise with a growing number of users and messages. You can optimize your Django server, WebSocket consumers, and database to handle increased load and ensure a smooth user experience.

