Chapter 8: Hashtags and mentions:

8.1 Use Django's built-in search functionality to search for hashtags and mentions

First, let's create a utility function that will parse the hashtags and mentions from a post's text.
# utils.py
import re

def extract_hashtags_and_mentions(text):
    hashtags = re.findall(r'#\w+', text)
    mentions = re.findall(r'@\w+', text)

    return hashtags, mentions

Now, let's add a signal to update the Hashtag model whenever a new post with hashtags is created.

# signals.py
from django.db.models.signals import post_save
from django.dispatch import receiver
from .models import Theeai, Hashtag
from .utils import extract_hashtags_and_mentions

@receiver(post_save, sender=Theeai)
def update_hashtags(sender, instance, created, **kwargs):
    if created:
        hashtags, _ = extract_hashtags_and_mentions(instance.text)
        
        for hashtag in hashtags:
            hashtag_obj, created = Hashtag.objects.get_or_create(name=hashtag)
            if not created:
                hashtag_obj.count += 1
                hashtag_obj.save()


8.2 Create a Hashtag model to store hashtag data

# models.py
from django.db import models

class Hashtag(models.Model):
    name = models.CharField(max_length=255, unique=True)
    count = models.PositiveIntegerField(default=1)

    def __str__(self):
        return self.name


8.3 Use Django Signals to update hashtag counts when a new post with the hashtag is posted

Now, let's add a signal to update the Hashtag model whenever a new post with hashtags is created.

# signals.py
from django.db.models.signals import post_save
from django.dispatch import receiver
from .models import Theeai, Hashtag
from .utils import extract_hashtags_and_mentions

@receiver(post_save, sender=Theeai)
def update_hashtags(sender, instance, created, **kwargs):
    if created:
        hashtags, _ = extract_hashtags_and_mentions(instance.text)
        
        for hashtag in hashtags:
            hashtag_obj, created = Hashtag.objects.get_or_create(name=hashtag)
            if not created:
                hashtag_obj.count += 1
                hashtag_obj.save()

8.4 Create custom serializers and views to handle hashtag and mention search

Create a serializer for the Hashtag model.

# serializers.py
from rest_framework import serializers
from .models import Hashtag

class HashtagSerializer(serializers.ModelSerializer):
    class Meta:
        model = Hashtag
        fields = ('name', 'count')

Next, create a view to search for hashtags and mentions.


# views.py
from django.shortcuts import render
from rest_framework import generics
from rest_framework.response import Response
from .serializers import HashtagSerializer
from .models import Hashtag
from .utils import extract_hashtags_and_mentions

class SearchHashtagsAndMentionsView(generics.GenericAPIView):
    serializer_class = HashtagSerializer

    def get_queryset(self):
        query = self.request.query_params.get('query', '')
        hashtags, mentions = extract_hashtags_and_mentions(query)

        hashtag_qs = Hashtag.objects.filter(name__in=hashtags)
        # Handle mentions search here

        return hashtag_qs

    def get(self, request, *args, **kwargs):
        queryset = self.get_queryset()
        serializer = self.get_serializer(queryset, many=True)
        return Response(serializer.data)

Finally, add a URL pattern for the search view.

# urls.py
from django.urls import path
from .views import SearchHashtagsAndMentionsView

urlpatterns = [
    path('search/', SearchHashtagsAndMentionsView.as_view(), name='search_hashtags_and_mentions'),
]


8.5 Implement searching for mentions

To implement mention searching, we'll modify the SearchHashtagsAndMentionsView to return posts that have mentions.

First, import the necessary models and serializers:

# views.py
from .models import Theeai, User
from .serializers import TheeaiSerializer

Now, modify the get_queryset method to search for mentions:


# views.py
def get_queryset(self):
    query = self.request.query_params.get('query', '')
    hashtags, mentions = extract_hashtags_and_mentions(query)

    hashtag_qs = Hashtag.objects.filter(name__in=hashtags)

    mentioned_users = User.objects.filter(username__in=mentions)
    mention_qs = Theeai.objects.filter(mentioned_users__in=mentioned_users)

    return hashtag_qs, mention_qs

Update the get method to return both hashtags and mentions:

# views.py
def get(self, request, *args, **kwargs):
    hashtag_queryset, mention_queryset = self.get_queryset()
    hashtag_serializer = HashtagSerializer(hashtag_queryset, many=True)
    mention_serializer = TheeaiSerializer(mention_queryset, many=True)
    
    return Response({
        'hashtags': hashtag_serializer.data,
        'mentions': mention_serializer.data,
    })

With these changes, the API endpoint will now return the search results for both hashtags and mentions in the JSON response.

Remember to wire up the signals in the app configuration:
# apps.py
from django.apps import AppConfig

class TheeaiConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'theeai'

    def ready(self):
        import theeai.signals

And ensure your app configuration is used in the project's settings:


# settings.py
INSTALLED_APPS = [
    ...
    'theeai.apps.TheeaiConfig',
    ...
]

With these updates, your Theeai application now supports searching for hashtags and mentions. Users can search for hashtags and mentions, and the application will return relevant results.


