Chapter 2: User Profile Management

In this chapter, we will create a separate UserProfile model that extends Django's User model to store additional user information. We will also use Django's form functionality to allow users to edit their profile information, and create custom serializers and views to handle user profile management using Django Rest Framework.

2.1 Create a UserProfile Model

First, create a new file called models.py in the accounts app folder and add the following code:

# models.py
from django.contrib.auth.models import User
from django.db import models
from django.db.models.signals import post_save
from django.dispatch import receiver


class UserProfile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    bio = models.TextField(blank=True, null=True)
    location = models.CharField(max_length=100, blank=True, null=True)
    profile_picture = models.ImageField(upload_to='profile_pictures/', blank=True, null=True)

    def __str__(self):
        return self.user.username


@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    if created:
        UserProfile.objects.create(user=instance)


@receiver(post_save, sender=User)
def save_user_profile(sender, instance, **kwargs):
    instance.userprofile.save()


In this code, we define a UserProfile model that has a one-to-one relationship with the User model. We also create two signal receivers to automatically create and save a UserProfile instance when a new User instance is created.

2.2 Use Django's Form Functionality to Edit Profile Information

Now, create a new file called forms.py in the accounts app folder and add the following code:

# forms.py
from django import forms
from django.contrib.auth.models import User
from .models import UserProfile


class UserUpdateForm(forms.ModelForm):
    email = forms.EmailField(required=True)

    class Meta:
        model = User
        fields = ['first_name', 'last_name', 'email']


class UserProfileUpdateForm(forms.ModelForm):
    class Meta:
        model = UserProfile
        fields = ['bio', 'location', 'profile_picture']


The UserUpdateForm and UserProfileUpdateForm classes allow users to edit their User and UserProfile information, respectively.

Next, create a new file called views.py in the accounts app folder (if it doesn't already exist) and add the following code to handle the profile update:

# views.py
from django.contrib import messages
from django.contrib.auth.decorators import login_required
from django.shortcuts import render, redirect
from .forms import UserUpdateForm, UserProfileUpdateForm


@login_required
def update_profile(request):
    if request.method == 'POST':
        user_form = UserUpdateForm(request.POST, instance=request.user)
        profile_form = UserProfileUpdateForm(request.POST, request.FILES, instance=request.user.userprofile)

        if user_form.is_valid() and profile_form.is_valid():
            user_form.save()
            profile_form.save()
            messages.success(request, 'Your profile has been updated.')
            return redirect('profile')

    else:
        user_form = UserUpdateForm(instance=request.user)
        profile_form = UserProfileUpdateForm(instance=request.user.userprofile)

    context = {
        'user_form': user_form,
        'profile_form': profile_form
    }
    return render(request, 'accounts/profile_update.html', context)


In this code, we create a update_profile view function that handles the profile update using the forms we defined earlier. Don't forget to update the urls.py file in the accounts app folder to include the new view:

# urls.py
from django.urls import path
from . import views

app_name = 'accounts'

urlpatterns = [
    path('update_profile/', views.update_profile, name='update_profile'),
]


In the urls.py file, we define a URL pattern for the update_profile view, which we will use to create the profile update form in the front-end.

2.3 Create Custom Serializers and Views to Handle User Profile Management

Now, we will create custom serializers and views to handle user profile management using Django Rest Framework. First, create a new file called serializers.py in the accounts app folder and add the following code:

# serializers.py
from rest_framework import serializers
from django.contrib.auth.models import User
from .models import UserProfile


class UserProfileSerializer(serializers.ModelSerializer):
    class Meta:
        model = UserProfile
        fields = ['bio', 'location', 'profile_picture']


class UserSerializer(serializers.ModelSerializer):
    userprofile = UserProfileSerializer()

    class Meta:
        model = User
        fields = ['username', 'email', 'first_name', 'last_name', 'userprofile']


In this code, we define two serializers: UserProfileSerializer and UserSerializer. The UserProfileSerializer serializes the UserProfile model, while the UserSerializer serializes the User model along with the related UserProfile instance.

Next, create a new file called views.py in the accounts app folder (if it doesn't already exist) and add the following code to handle the API endpoints for user profile management:

# views.py
from rest_framework import generics
from django.contrib.auth.models import User
from .models import UserProfile
from .serializers import UserSerializer, UserProfileSerializer


class UserProfileDetailView(generics.RetrieveUpdateAPIView):
    queryset = UserProfile.objects.all()
    serializer_class = UserProfileSerializer


class UserDetailView(generics.RetrieveUpdateAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer


In this code, we create two view classes: UserProfileDetailView and UserDetailView. The UserProfileDetailView view handles the retrieval and updating of UserProfile instances, while the UserDetailView view handles the retrieval and updating of User instances.

Finally, update the urls.py file in the accounts app folder to include the new views:

# urls.py
from django.urls import path
from . import views

app_name = 'accounts'

urlpatterns = [
    path('update_profile/', views.update_profile, name='update_profile'),
    path('userprofile/<int:pk>/', views.UserProfileDetailView.as_view(), name='userprofile_detail'),
    path('user/<int:pk>/', views.UserDetailView.as_view(), name='user_detail'),
]


In the urls.py file, we define URL patterns for the UserProfileDetailView and UserDetailView views, which we will use to create the API endpoints for user profile management.

With these changes, we have successfully implemented user profile management in our application using Django, Django Rest Framework, and React. Users can now edit their profile information, and the application will store the updated data in the UserProfile model.

