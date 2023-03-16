Chapter 6: Follow system:

In this chapter, we will implement a follow system that allows users to follow other users and receive updates from them in their post feed.

6.1 Create a Follow model to store follower/following relationships:

Start by creating a new Django model to represent the follower/following relationships between users.

# models.py
from django.db import models
from django.contrib.auth.models import User

class Follow(models.Model):
    follower = models.ForeignKey(User, on_delete=models.CASCADE, related_name='following')
    following = models.ForeignKey(User, on_delete=models.CASCADE, related_name='followers')

    class Meta:
        unique_together = ('follower', 'following')

    def __str__(self):
        return f"{self.follower} follows {self.following}"

6.2 Use Django Rest Framework to create API endpoints to handle follow/unfollow actions:

Now, create views and serializers for handling follow and unfollow actions using Django Rest Framework.

# serializers.py
from rest_framework import serializers
from .models import Follow

class FollowSerializer(serializers.ModelSerializer):
    class Meta:
        model = Follow
        fields = ('follower', 'following')

# views.py
from rest_framework import generics
from .models import Follow
from .serializers import FollowSerializer

class FollowUserView(generics.CreateAPIView):
    queryset = Follow.objects.all()
    serializer_class = FollowSerializer

class UnfollowUserView(generics.DestroyAPIView):
    queryset = Follow.objects.all()
    serializer_class = FollowSerializer

    def get_object(self):
        follower = self.request.user
        following = self.kwargs.get('following_id')
        follow_instance = Follow.objects.get(follower=follower, following=following)
        return follow_instance

Next, update the URL patterns to include the new API endpoints:

# urls.py
from django.urls import path
from . import views

urlpatterns = [
    # ... existing urlpatterns
    path('follow/', views.FollowUserView.as_view(), name='follow_user'),
    path('unfollow/<int:following_id>/', views.UnfollowUserView.as_view(), name='unfollow_user'),
]

6.3 Create custom serializers and views to handle follow/unfollow actions:

Now, create the corresponding React components for the follow and unfollow actions.

// FollowUser.js
import React from 'react';
import axios from 'axios';

const FollowUser = ({ follower, following, onFollowed }) => {
  const followUser = async () => {
    const response = await axios.post('/api/follow/', {
      follower: follower.id,
      following: following.id,
    });
    onFollowed();
  };

  return (
    <button onClick={followUser}>
      Follow
    </button>
  );
};

// UnfollowUser.js
import React from 'react';
import axios from 'axios';

const UnfollowUser = ({ follower, following, onUnfollowed }) => {
  const unfollowUser = async () => {
    const response = await axios.delete(`/api/unfollow/${following.id}/`);
    onUnfollowed();
  };

  return (
    <button onClick={unfollowUser}>
      Unfollow
    </button>
  );
};


Finally, update the user search component to include the follow and unfollow components, allowing users to follow and unfollow other users directly from the search results:


// UserSearch.js
import React, { useState } from 'react';
import axios from 'axios';
import FollowUser from './FollowUser';
import UnfollowUser from './UnfollowUser';

// ... existing UserSearch component

const handleUserFollowed = (userId) => {
  const updated

// UserSearch.js (continued)

const updatedUsers = users.map((user) => {
  if (user.id === userId) {
    return { ...user, is_followed: true };
  }
  return user;
});

setUsers(updatedUsers);
};

const handleUserUnfollowed = (userId) => {
  const updatedUsers = users.map((user) => {
    if (user.id === userId) {
      return { ...user, is_followed: false };
    }
    return user;
  });

  setUsers(updatedUsers);
};

return (
  <div>
    {/* ... existing search input and user list rendering */}
    {user.is_followed ? (
      <UnfollowUser
        follower={currentUser}
        following={user}
        onUnfollowed={() => handleUserUnfollowed(user.id)}
      />
    ) : (
      <FollowUser
        follower={currentUser}
        following={user}
        onFollowed={() => handleUserFollowed(user.id)}
      />
    )}
  </div>
);

6.4 Update the post feed to display posts from followed users:

Update the post feed component to fetch posts only from the users followed by the current user.

// PostFeed.js
import React, { useState, useEffect } from 'react';
import axios from 'axios';

const PostFeed = ({ currentUser }) => {
  const [posts, setPosts] = useState([]);

  useEffect(() => {
    const fetchPosts = async () => {
      const response = await axios.get('/api/posts/following/', {
        params: { user: currentUser.id },
      });

      setPosts(response.data);
    };

    fetchPosts();
  }, [currentUser]);

  return (
    <div>
      {posts.map((post) => (
        <div key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.content}</p>
        </div>
      ))}
    </div>
  );
};


Update the backend to support filtering posts by followed users:

# views.py
from rest_framework import filters
from django_filters import rest_framework as django_filters

class FollowingPostsFilter(django_filters.FilterSet):
    user = django_filters.NumberFilter(field_name='author__followers__follower', distinct=True)

    class Meta:
        model = Post
        fields = ['user']

class PostList(generics.ListCreateAPIView):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
    filter_backends = [filters.SearchFilter, django_filters.DjangoFilterBackend]
    filterset_class = FollowingPostsFilter
    search_fields = ['title', 'content']

Now, your social media platform allows users to follow and unfollow each other, and the post feed will only display posts from users they follow.



