As we have covered user search and user profile management, let's move forward with implementing the friend request system. This will enable users to send and accept friend requests, allowing them to grow their network within the platform.

Chapter 5: Friend Request System

5.1 Create a FriendRequest model to store friend request data:
In your Django app, create a new model called FriendRequest to store information about friend requests. This model should have fields for the sender, receiver, and the timestamp of when the request was sent.

# models.py
from django.db import models
from django.contrib.auth.models import User

class FriendRequest(models.Model):
    sender = models.ForeignKey(User, on_delete=models.CASCADE, related_name='sent_requests')
    receiver = models.ForeignKey(User, on_delete=models.CASCADE, related_name='received_requests')
    timestamp = models.DateTimeField(auto_now_add=True)


5.2 Create API endpoints for sending and accepting friend requests:
Create Django Rest Framework views and serializers for the FriendRequest model. Create an endpoint to allow users to send friend requests and another endpoint to accept friend requests.

# serializers.py
from rest_framework import serializers
from .models import FriendRequest

class FriendRequestSerializer(serializers.ModelSerializer):
    class Meta:
        model = FriendRequest
        fields = ('id', 'sender', 'receiver', 'timestamp')

# views.py
from rest_framework import generics
from .models import FriendRequest
from .serializers import FriendRequestSerializer

class SendFriendRequestView(generics.CreateAPIView):
    queryset = FriendRequest.objects.all()
    serializer_class = FriendRequestSerializer

class AcceptFriendRequestView(generics.UpdateAPIView):
    queryset = FriendRequest.objects.all()
    serializer_class = FriendRequestSerializer


5.3 Update the Django URL patterns for the new API endpoints:

# urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('send_friend_request/', views.SendFriendRequestView.as_view(), name='send_friend_request'),
    path('accept_friend_request/<int:pk>/', views.AcceptFriendRequestView.as_view(), name='accept_friend_request'),
]

5.4 Create React components for sending and accepting friend requests:
In your React application, create components to handle sending and accepting friend requests. Use axios to make HTTP requests to the back-end API for sending and accepting friend requests. Update the user's friends list when a friend request is accepted.

// SendFriendRequest.js
import React, { useState } from 'react';
import axios from 'axios';

const SendFriendRequest = ({ sender, receiver }) => {
  const [sent, setSent] = useState(false);

  const sendRequest = async () => {
    const response = await axios.post('/api/send_friend_request/', { sender, receiver });
    setSent(true);
  };

  return (
    <button onClick={sendRequest} disabled={sent}>
      {sent ? 'Request Sent' : 'Send Friend Request'}
    </button>
  );
};

// AcceptFriendRequest.js
import React, { useState } from 'react';
import axios from 'axios';

const AcceptFriendRequest = ({ requestId }) => {
  const [accepted, setAccepted] = useState(false);

  const acceptRequest = async () => {
    const response = await axios.patch(`/api/accept_friend_request/${requestId}/`);
    setAccepted(true);
  };

  return (
    <button onClick={acceptRequest} disabled={accepted}>
      {accepted ? 'Request Accepted' : 'Accept Friend Request'}
    </button>
  );
};


With these components in place, you can now integrate them into your existing user search and profile pages. This will allow users to send and accept friend requests and grow their network on the platform.

5.5 Integrating friend request components with user search and profile pages:
Update your user search and profile components to include the new friend request components. This will allow users to send friend requests directly from the search results or user profile pages.

// UserSearchResult.js
import React from 'react';
import SendFriendRequest from './SendFriendRequest';

const UserSearchResult = ({ user, currentUser }) => {
  return (
    <div>
      <h3>{user.username}</h3>
      <p>{user.email}</p>
      {currentUser.id !== user.id && (
        <SendFriendRequest sender={currentUser.id} receiver={user.id} />
      )}
    </div>
  );
};

// UserProfile.js
import React, { useState, useEffect } from 'react';
import axios from 'axios';
import AcceptFriendRequest from './AcceptFriendRequest';

const UserProfile = ({ user }) => {
  const [friendRequests, setFriendRequests] = useState([]);

  useEffect(() => {
    const fetchFriendRequests = async () => {
      const response = await axios.get(`/api/user/${user.id}/friend_requests/`);
      setFriendRequests(response.data);
    };

    fetchFriendRequests();
  }, [user.id]);

  return (
    <div>
      <h2>{user.username}'s Profile</h2>
      <p>{user.email}</p>
      <h3>Friend Requests</h3>
      {friendRequests.map((request) => (
        <div key={request.id}>
          <p>{request.sender.username} sent you a friend request.</p>
          <AcceptFriendRequest requestId={request.id} />
        </div>
      ))}
    </div>
  );
};


5.6 Updating friends list when a friend request is accepted:
When a user accepts a friend request, the friends list should be updated to reflect the new friendship. Update your user profile component to fetch the user's friends list and re-render the component when a friend request is accepted.

// UserProfile.js
import React, { useState, useEffect } from 'react';
import axios from 'axios';
import AcceptFriendRequest from './AcceptFriendRequest';

const UserProfile = ({ user }) => {
  const [friendRequests, setFriendRequests] = useState([]);
  const [friends, setFriends] = useState([]);

  useEffect(() => {
    const fetchFriendRequests = async () => {
      const response = await axios.get(`/api/user/${user.id}/friend_requests/`);
      setFriendRequests(response.data);
    };

    const fetchFriends = async () => {
      const response = await axios.get(`/api/user/${user.id}/friends/`);
      setFriends(response.data);
    };

    fetchFriendRequests();
    fetchFriends();
  }, [user.id]);

  const handleFriendRequestAccepted = (requestId) => {
    setFriendRequests(friendRequests.filter((request) => request.id !== requestId));
    // Fetch friends list again to reflect the new friendship
    fetchFriends();
  };

  return (
    <div>
      <h2>{user.username}'s Profile</h2>
      <p>{user.email}</p>
      <h3>Friend Requests</h3>
      {friendRequests.map((request) => (
        <div key={request.id}>
          <p>{request.sender.username} sent you a friend request.</p>
          <AcceptFriendRequest requestId={request.id} onAccepted={handleFriendRequestAccepted} />
        </div>
      ))}
      <h3>Friends</h3>
      {friends.map((friend) => (
        <div key={friend.id}>
          <p>{friend.username}</p>
        </div>
      ))}
   
</div>
  );
};

5.7 Implementing friend removal functionality:
To give users the ability to remove friends from their friends list, create a new API endpoint for friend removal and a corresponding React component.

First, add a new view in the Django back-end to handle the friend removal request:

# views.py
from rest_framework import generics
from .models import FriendRequest
from .serializers import FriendRequestSerializer

class RemoveFriendView(generics.DestroyAPIView):
    queryset = FriendRequest.objects.all()
    serializer_class = FriendRequestSerializer

    def get_object(self):
        user1 = self.request.user
        user2 = self.kwargs.get('user_id')
        friendship = FriendRequest.objects.get_friendship(user1, user2)
        return friendship


Then, update the Django URL patterns for the new friend removal API endpoint:

# urls.py
from django.urls import path
from . import views

urlpatterns = [
    # ... existing urlpatterns
    path('remove_friend/<int:user_id>/', views.RemoveFriendView.as_view(), name='remove_friend'),
]

Now, create a new React component for removing friends:

// RemoveFriend.js
import React from 'react';
import axios from 'axios';

const RemoveFriend = ({ user1, user2, onRemoved }) => {
  const removeFriend = async () => {
    const response = await axios.delete(`/api/remove_friend/${user2.id}/`);
    onRemoved(user2.id);
  };

  return (
    <button onClick={removeFriend}>
      Remove Friend
    </button>
  );
};


Finally, update the user profile component to include the RemoveFriend component, allowing users to remove friends directly from their friends list:
// UserProfile.js
import React, { useState, useEffect } from 'react';
import axios from 'axios';
import AcceptFriendRequest from './AcceptFriendRequest';
import RemoveFriend from './RemoveFriend';

// ... existing UserProfile component

const handleFriendRemoved = (friendId) => {
  setFriends(friends.filter((friend) => friend.id !== friendId));
};

return (
  <div>
    {/* ... existing JSX */}
    <h3>Friends</h3>
    {friends.map((friend) => (
      <div key={friend.id}>
        <p>{friend.username}</p>
        <RemoveFriend user1={user.id} user2={friend.id} onRemoved={handleFriendRemoved} />
      </div>
    ))}
  </div>
);

With these updates, users can now remove friends from their friends list. This concludes the implementation of the friend request system, including sending, accepting, and removing friends.






