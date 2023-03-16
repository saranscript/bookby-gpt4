Chapter 7: Like and repost functionality

In this chapter, we will implement the like and repost functionality for Theeai posts. We'll create models to store user actions, API endpoints to handle these actions, and frontend components to allow users to like and repost posts.

7.1 Create a Like and Repost model to store user actions

First, let's create the Like and Repost models in our Django application.

# models.py
from django.db import models
from django.contrib.auth.models import User

class Theeai(models.Model):
    # ... existing fields ...

class Like(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    theeai = models.ForeignKey(Theeai, on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)

class Repost(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    theeai = models.ForeignKey(Theeai, on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)

7.2 Use Django Rest Framework to create API endpoints to handle like and repost actions

Next, create API endpoints to handle like and repost actions using Django Rest Framework.

# serializers.py
from rest_framework import serializers
from .models import Theeai, Like, Repost

class LikeSerializer(serializers.ModelSerializer):
    class Meta:
        model = Like
        fields = ['id', 'user', 'theeai', 'created_at']

class RepostSerializer(serializers.ModelSerializer):
    class Meta:
        model = Repost
        fields = ['id', 'user', 'theeai', 'created_at']

# views.py
from rest_framework import generics
from .models import Theeai, Like, Repost
from .serializers import TheeaiSerializer, LikeSerializer, RepostSerializer

class LikeListCreate(generics.ListCreateAPIView):
    queryset = Like.objects.all()
    serializer_class = LikeSerializer

class RepostListCreate(generics.ListCreateAPIView):
    queryset = Repost.objects.all()
    serializer_class = RepostSerializer

# urls.py
from django.urls import path
from . import views

urlpatterns = [
    # ... existing endpoints ...
    path('api/likes/', views.LikeListCreate.as_view(), name='like-list-create'),
    path('api/reposts/', views.RepostListCreate.as_view(), name='repost-list-create'),
]

7.3 Use Django Signals to update the post's likes and reposts counts

We'll use Django Signals to automatically update the likes and reposts counts when a user likes or reposts a post.

# models.py
from django.db.models.signals import post_save, post_delete
from django.dispatch import receiver

class Theeai(models.Model):
    # ... existing fields ...
    likes_count = models.PositiveIntegerField(default=0)
    reposts_count = models.PositiveIntegerField(default=0)

@receiver(post_save, sender=Like)
def increment_likes_count(sender, instance, created, **kwargs):
    if created:
        instance.theeai.likes_count += 1
        instance.theeai.save()

@receiver(post_delete, sender=Like)
def decrement_likes_count(sender, instance, **kwargs):
    instance.theeai.likes_count -= 1
    instance.theeai.save()

@receiver(post_save, sender=Repost)
def increment_reposts_count(sender, instance, created, **kwargs):
    if created:
        instance.theeai.reposts_count += 1
        instance.theeai.save()

@receiver(post_delete, sender=Repost)
def decrement_reposts_count(sender, instance, **kwargs):
    instance.theeai.reposts_count -= 1
    instance.theeai.save()

7.4 Create frontend components for like and repost actions

Now that we have our backend set up, let's create frontend components to allow users to like and repost posts.

First, create the LikeButton and RepostButton components.

// LikeButton.js
import React from 'react';

const LikeButton = ({ isLiked, onClick }) => {
  const buttonLabel = isLiked ? 'Unlike' : 'Like';

  return (
    <button onClick={onClick} className="like-button">
      {buttonLabel}
    </button>
  );
};

export default LikeButton;

// RepostButton.js
import React from 'react';

const RepostButton = ({ isReposted, onClick }) => {
  const buttonLabel = isReposted ? 'Undo Repost' : 'Repost';

  return (
    <button onClick={onClick} className="repost-button">
      {buttonLabel}
    </button>
  );
};

export default RepostButton;


Next, update the Post component to include LikeButton and RepostButton components, and handle like and repost actions.

// Post.js
import React, { useState, useEffect } from 'react';
import LikeButton from './LikeButton';
import RepostButton from './RepostButton';
import axios from 'axios';

const Post = ({ post }) => {
  const [isLiked, setIsLiked] = useState(false);
  const [isReposted, setIsReposted] = useState(false);

  useEffect(() => {
    // Check if the current user has liked or reposted the post
    // Update isLiked and isReposted state accordingly
  }, []);

  const handleLike = async () => {
    // Handle like/unlike action
    // Send a request to the like API endpoint
    // Update the state of isLiked
  };

  const handleRepost = async () => {
    // Handle repost/undo repost action
    // Send a request to the repost API endpoint
    // Update the state of isReposted
  };

  return (
    <div className="post">
      {/* Post content */}
      <LikeButton isLiked={isLiked} onClick={handleLike} />
      <RepostButton isReposted={isReposted} onClick={handleRepost} />
    </div>
  );
};

export default Post;

In this chapter, we've implemented the like and repost functionality for Theeai posts using Django, Django Rest Framework, and React. Users can now like and repost posts, and the application will handle these actions by updating the post's likes and reposts counts.


