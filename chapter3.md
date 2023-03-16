Chapter 3: Theeai Creation and Posting

In this chapter, we will implement the functionality for creating and posting Theeais (tweets) using Django, Django Rest Framework, and React.

3.1 Create a Theeai Model to Store Post Data

First, create a new Django app called theeais and add it to the INSTALLED_APPS list in your project's settings.py file. Inside the models.py file of the theeais app, define the Theeai model as follows:

# models.py
from django.db import models
from django.contrib.auth.models import User

class Theeai(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    content = models.CharField(max_length=280)
    image = models.ImageField(upload_to='theeai_images/', blank=True, null=True)
    video = models.FileField(upload_to='theeai_videos/', blank=True, null=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return self.content


In this code, we define the Theeai model with fields for the user, content, image, video, and timestamps. We also specify the upload paths for images and videos.

3.2 Use Django's Built-in Forms to Handle Post Creation and Validation

Next, create a new file called forms.py in the theeais app folder and add the following code:

# forms.py
from django import forms
from .models import Theeai

class TheeaiForm(forms.ModelForm):
    class Meta:
        model = Theeai
        fields = ['content', 'image', 'video']

    def clean_content(self):
        content = self.cleaned_data.get('content')
        if len(content) > 280:
            raise forms.ValidationError("Content cannot exceed 280 characters.")
        return content

In this code, we define a TheeaiForm class that inherits from forms.ModelForm. We specify the model and the fields to include in the form. We also add a clean_content method to validate the content length.

3.3 Use Django Rest Framework to Create API Endpoints for Post Creation and Posting

Create a new file called serializers.py in the theeais app folder and add the following code:

# serializers.py
from rest_framework import serializers
from .models import Theeai

class TheeaiSerializer(serializers.ModelSerializer):
    class Meta:
        model = Theeai
        fields = ['id', 'user', 'content', 'image', 'video', 'created_at', 'updated_at']


In this code, we define a TheeaiSerializer class that inherits from serializers.ModelSerializer. We specify the model and the fields to include in the serialization.

Next, create a new file called views.py in the theeais app folder and add the following code:

# views.py
from rest_framework import generics
from .models import Theeai
from .serializers import TheeaiSerializer

class TheeaiCreateView(generics.CreateAPIView):
    queryset = Theeai.objects.all()
    serializer_class = TheeaiSerializer

class TheeaiListView(generics.ListAPIView):
    queryset = Theeai.objects.all()
    serializer_class = TheeaiSerializer


In this code, we define two view classes: TheeaiCreateView and TheeaiListView. The TheeaiCreateView view handles the creation of new Theeai instances, while the TheeaiListView view handles the listing of all Theeais. Both views use the TheeaiSerializer class for serialization.

Now, create a new file called urls.py in the theeais app folder and add the following code:

# urls.py
from django.urls import path
from .views import TheeaiCreateView, TheeaiListView

urlpatterns = [
    path('create/', TheeaiCreateView.as_view(), name='theeai-create'),
    path('list/', TheeaiListView.as_view(), name='theeai-list'),
]


In this code, we define the URL patterns for the TheeaiCreateView and TheeaiListView views. Make sure to include these urlpatterns in your project's urls.py file.

3.4 Use Django Signals to Enable Real-time Post Updates Using Django Channels
In this section, we will use Django Signals to enable real-time updates for Theeai posts. First, install Django Channels and its dependencies:

pip install channels channels-redis

Next, add the following code to your project's settings.py file:
INSTALLED_APPS = [
    ...
    'channels',
]

CHANNEL_LAYERS = {
    "default": {
        "BACKEND": "channels_redis.core.RedisChannelLayer",
        "CONFIG": {
            "hosts": [("localhost", 6379)],
        },
    },
}


Create a new file called consumers.py in the theeais app folder and add the following code:

# consumers.py
import json
from channels.generic.websocket import AsyncWebsocketConsumer

class TheeaiConsumer(AsyncWebsocketConsumer):
    async def connect(self):
        self.room_group_name = 'theeai_updates'
        await self.channel_layer.group_add(self.room_group_name, self.channel_name)
        await self.accept()

    async def disconnect(self, close_code):
        await self.channel_layer.group_discard(self.room_group_name, self.channel_name)

    async def receive(self, text_data):
        text_data_json = json.loads(text_data)
        message = text_data_json['message']

        await self.channel_layer.group_send(
            self.room_group_name,
            {
                'type': 'theeai_message',
                'message': message
            }
        )

    async def theeai_message(self, event):
        message = event['message']
        await self.send(text_data=json.dumps({'message': message}))


In this code, we define a TheeaiConsumer class that inherits from AsyncWebsocketConsumer. This class handles WebSocket connections, disconnections, message receiving, and message broadcasting.

Next, update your project's urls.py file to include the WebSocket routing:

# urls.py
from django.urls import path, re_path
from channels.routing import ProtocolTypeRouter, URLRouter
from theeais.consumers import TheeaiConsumer

application = ProtocolTypeRouter(
    {
        "websocket": URLRouter(
            [
                re_path(r'ws/theeai_updates/$', TheeaiConsumer.as_asgi()),
            ]
        ),
    }
)


Finally, use Django Signals to trigger a WebSocket message when a new Theeai is created. Add the following code to the models.py file in the theeais app folder:

# models.py
from django.db.models.signals import post_save
from django.dispatch import receiver
from channels.layers import get_channel_layer
from asgiref.sync import async_to_sync

@receiver(post_save, sender=Theeai)
def send_theeai_update(sender, instance, created, **kwargs):
    if created:
        channel_layer = get_channel_layer()
        message = {
            'type': 'theeai_message',
            'message': {
                'id': instance.id,
                'user': instance.user.username,
                'content': instance.content,
                'image': instance.image.url if instance.image else None,
                'video': instance.video.url if instance.video else None,
                'created_at': instance.created_at.isoformat(),
                'updated_at': instance.updated_at.isoformat(),
            }
        }

        async_to_sync(channel_layer.group_send)('theeai_updates', message)


In this code, we define a signal receiver function send_theeai_update that listens for the post_save signal of the Theeai model. When a new Theeai instance is created, the function sends a WebSocket message containing the new Theeai's data to the 'theeai_updates' group.

3.5 Implement the Frontend Using React

Now, we will implement the frontend for posting and displaying Theeais using React. First, create a new React app using the following command:

npx create-react-app theeai_frontend


Install Axios for making API calls and Bootstrap for styling:

cd theeai_frontend
npm install axios bootstrap

Now, create a new file called TheeaiForm.js inside the src folder and add the following code:

// TheeaiForm.js
import React, { useState } from 'react';
import axios from 'axios';

const TheeaiForm = () => {
  const [content, setContent] = useState('');
  const [image, setImage] = useState(null);
  const [video, setVideo] = useState(null);

  const handleSubmit = async (e) => {
    e.preventDefault();

    const formData = new FormData();
    formData.append('content', content);
    if (image) formData.append('image', image);
    if (video) formData.append('video', video);

    try {
      const response = await axios.post('/api/theeais/create/', formData);
      console.log(response.data);
      setContent('');
      setImage(null);
      setVideo(null);
    } catch (error) {
      console.error(error);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* ... */}

      {/* Add input elements for content, image, and video here */}

      {/* ... */}
    </form>
  );
};

export default TheeaiForm;


In this code, we define a functional React component TheeaiForm that contains the state and logic for submitting a new Theeai. Add the necessary input elements for content, image, and video in the form.

Next, create a new file called TheeaiList.js inside the src folder and add the following code:

// TheeaiList.js
import React, { useState, useEffect } from 'react';
import axios from 'axios';

const TheeaiList = () => {
  const [theeais, setTheeais] = useState([]);

  useEffect(() => {
    const fetchTheeais = async () => {
      try {
        const response = await axios.get('/api/theeais/list/');
        setTheeais(response.data);
      } catch (error) {
        console.error(error);
      }
    };

    fetchTheeais();
  }, []);

  return (
    <div>
      {theeais.map((theeai) => (
        <div key={theeai.id}>
          <h3>{theeai.user}</h3>
          <p>{theeai.content}</p>
          {theeai.image && <img src={theeai.image} alt="Theeai" />}
          {theeai.video && <video src={theeai.video} controls />}
          <p>{theeai.created_at}</p>
        </div>
      ))}
    </div>
  );
};

export default TheeaiList;

In this code, we define a functional React component TheeaiList that fetches and displays a list of Theeais. We use the useEffect hook to fetch the Theeai data when the component mounts.

Now, update the App.js file to include both TheeaiForm and TheeaiList components:

// App.js
import React from 'react';
import 'bootstrap/dist/css/bootstrap.min.css';
import TheeaiForm from './TheeaiForm';
import TheeaiList from './TheeaiList';

const App = () => {
  return (
    <div className="container">
      <h1>Theeai</h1>
      <TheeaiForm />
      <TheeaiList />
    </div>
  );
};

export default App;


Finally, update your frontend to listen for WebSocket messages containing new Theeais and update the list in real-time. To do this, create a new file called TheeaiSocket.js inside the src folder and add the following code:

// TheeaiSocket.js
import { useEffect } from 'react';

const TheeaiSocket = ({ onNewTheeai }) => {
  useEffect(() => {
    const socket = new WebSocket('ws://localhost:8000/ws/theeai_updates/');

    socket.onmessage = (event) => {
      const data = JSON.parse(event.data);
      const message = data.message;
      onNewTheeai(message);
    };

    return () => {
      socket.close();
    };
  }, [onNewTheeai]);

  return null;
};

export default TheeaiSocket;


In this code, we define a functional React component TheeaiSocket that establishes a WebSocket connection and listens for new Theeai messages. When a new message is received, the onNewTheeai callback function is called with the new Theeai data.

Update the App.js file to include the TheeaiSocket component and handle new Theeai messages:

// App.js
import React, { useState } from 'react';
import 'bootstrap/dist/css/bootstrap.min.css';
import TheeaiForm from './TheeaiForm';
import TheeaiList from './TheeaiList';
import TheeaiSocket from './TheeaiSocket';

const App = () => {
  const [theeais, setTheeais] = useState([]);

  const handleNewTheeai = (newTheeai) => {
    setTheeais((prevTheeais) => [newTheeai, ...prevTheeais]);
  };

  return (
    <div className="container">
      <h1>Theeai</h1>
      <TheeaiForm />
      <TheeaiList theeais={theeais} />
      <TheeaiSocket onNewTheeai={handleNewTheeai} />
    </div>
  );
};

export default App;

In this updated App.js file, we include the TheeaiSocket component and handle new Theeai messages by updating the theeais state. We also pass the theeais state down to the TheeaiList component as a prop.
Now, run your Django server and React development server, and you should be able to create, post, and view Theeais in real-time.

To summarize, in this chapter, we implemented the functionality for creating and posting Theeais using Django, Django Rest Framework, and React. We also added real-time updates for Theeai posts using Django Channels and WebSockets.

