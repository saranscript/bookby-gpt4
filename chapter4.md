Chapter 4: Theeai feed:
In this chapter, we will create a Theeai feed that displays the posts of users and those they follow. We will retrieve the posts using Django Rest Framework and implement real-time updates using Django Channels.

4.1 Use Django Rest Framework to create API endpoints to retrieve posts:

First, let's create an API endpoint to retrieve the posts of the authenticated user and the users they follow. We'll use the Django Rest Framework for this task.

In your thee app's views.py file, create a new view to retrieve the Theeai feed:

from django.db.models import Q
from rest_framework import generics
from .models import Theeai
from .serializers import TheeaiSerializer

class TheeaiFeedView(generics.ListAPIView):
    serializer_class = TheeaiSerializer

    def get_queryset(self):
        user = self.request.user
        following_users = user.profile.following.all()
        return Theeai.objects.filter(Q(user=user) | Q(user__profile__in=following_users)).order_by('-created_at')

In the TheeaiFeedView, we filter the Theeai objects by the authenticated user and the users they follow using the Q object from django.db.models. We then order the results by the created_at field in descending order.

Now, let's add this view to the urls.py file of the thee app:

from django.urls import path
from . import views

urlpatterns = [
    # ...
    path('api/theeais/feed/', views.TheeaiFeedView.as_view(), name='theeai_feed'),
]


4.2 Implement a WebSocket using Django Channels to send real-time updates to the user's post feed:

To send real-time updates to the user's post feed, we need to create a WebSocket using Django Channels. We'll build upon the WebSocket we implemented in Chapter 3. Create a new FeedConsumer class in your consumers.py file:

from channels.generic.websocket import WebsocketConsumer
import json
from .models import Theeai

class FeedConsumer(WebsocketConsumer):

    def connect(self):
        self.user = self.scope["user"]
        self.accept()

    def disconnect(self, close_code):
        pass

    def receive(self, text_data):
        data = json.loads(text_data)
        action = data.get('action')

        if action == 'get_feed':
            theeais = Theeai.objects.filter(user__profile__in=self.user.profile.following.all()).order_by('-created_at')
            theeais_data = TheeaiSerializer(theeais, many=True).data
            self.send(text_data=json.dumps({'theeais': theeais_data}))


In the FeedConsumer, we implement the connect, disconnect, and receive methods. When the client requests the feed, the receive method retrieves the Theeai objects and sends them back to the client.

Next, update the routing.py file in your thee app to include the FeedConsumer:

from django.urls import path
from . import consumers

websocket_urlpatterns = [
    # ...
    path('ws/feed/', consumers.FeedConsumer.as_asgi()),
]

4.3 Create custom serializers and views to handle retrieving the post feed:

In the thee app's serializers.py file, create a new serializer to handle the post feed data:
from rest_framework import serializers
from .models import Theeai, UserProfile

class FeedTheeaiSerializer(serializers.ModelSerializer):
    user = serializers.StringRelatedField()
    profile = serializers.SerializerMethodField()

    class Meta:
        model = Theeai
        fields = ['id', 'content', 'user', 'profile', 'created_at']

    def get_profile(self, obj):
        return UserProfileSerializer(obj.user.profile).data

class UserProfileSerializer(serializers.ModelSerializer):
    class Meta:
        model = UserProfile
        fields = ['id', 'username', 'avatar']

In the FeedTheeaiSerializer, we're including the user and their profile data with the Theeai object. The get_profile method retrieves the user's profile and serializes it using the UserProfileSerializer.

Next, update the TheeaiFeedView in the views.py file to use the new FeedTheeaiSerializer:

from .serializers import TheeaiSerializer, FeedTheeaiSerializer

class TheeaiFeedView(generics.ListAPIView):
    serializer_class = FeedTheeaiSerializer
    # ...


4.4 Update the frontend to display the Theeai feed:

In your theeai-feed.js file, create a new function to request the Theeai feed from the API:
async function fetchTheeaiFeed() {
    const response = await fetch('/api/theeais/feed/', {
        headers: {
            'Authorization': `Token ${localStorage.getItem('token')}`
        }
    });
    const data = await response.json();
    return data;
}


Now, call the fetchTheeaiFeed function when the page loads, and display the Theeai feed in the HTML:

window.addEventListener('load', async () => {
    const feed = await fetchTheeaiFeed();
    displayTheeaiFeed(feed);
});

function displayTheeaiFeed(feed) {
    const feedContainer = document.getElementById('theeai-feed-container');
    feed.forEach(theeai => {
        const theeaiElement = createTheeaiElement(theeai);
        feedContainer.appendChild(theeaiElement);
    });
}

function createTheeaiElement(theeai) {
    const theeaiElement = document.createElement('div');
    theeaiElement.className = 'theeai';
    theeaiElement.innerHTML = `
        <img src="${theeai.profile.avatar}" alt="${theeai.user}'s avatar">
        <div class="theeai-info">
            <h4>${theeai.user}</h4>
            <p>${theeai.content}</p>
            <small>${new Date(theeai.created_at).toLocaleString()}</small>
        </div>
    `;
    return theeaiElement;
}

This will display the Theeai feed on the page when it loads. Now your users can view a feed of the posts from the users they follow.

