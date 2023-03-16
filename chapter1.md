Chapter 1: User Registration and Authentication
In this chapter, we will focus on creating a user registration and authentication system for the Thee.ai social media app. We will use Django's built-in User model, Django-allauth for social media login, and Django-rest-auth to handle token-based authentication for the API. We will also create custom serializers and views to manage user registration and authentication.

1.1 Using Django's Built-in User Model

Django comes with a built-in User model that includes fields for username, email, password, first name, and last name. We will use this model to handle user registration and authentication. First, let's create a new Django app called accounts to manage user-related functionality.

Open a terminal and navigate to the root directory of your project, then run:

python manage.py startapp accounts


This will create a new folder called accounts in your project directory, with the necessary files for a Django app. Now, let's add the new app to the INSTALLED_APPS list in your project's settings.py file:

# settings.py
INSTALLED_APPS = [
    # ...
    'accounts',
]


1.2 Implementing Social Media Login Using Django-allauth

Django-allauth is a popular package for handling social media authentication in Django. We will use it to allow users to log in using their social media accounts. First, install Django-allauth using pip:

pip install django-allauth

Next, add allauth and allauth.account to the INSTALLED_APPS list in your project's settings.py file:
# settings.py
INSTALLED_APPS = [
    # ...
    'allauth',
    'allauth.account',
]


Now, include the allauth.urls in your project's urls.py file:
# urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('accounts/', include('allauth.urls')),
]


Finally, configure Django-allauth settings in your settings.py file:

# settings.py
AUTHENTICATION_BACKENDS = (
    'django.contrib.auth.backends.ModelBackend',
    'allauth.account.auth_backends.AuthenticationBackend',
)

# For demonstration purposes, we set this to False
ACCOUNT_EMAIL_VERIFICATION = 'none'

# Set this to your desired login URL
LOGIN_URL = '/accounts/login/'

# Set this to your desired logout URL
LOGOUT_URL = '/accounts/logout/'


You can now configure specific social media providers (such as Google, Facebook, or Twitter) by following the Django-allauth documentation for each provider.

1.3 Using Django-rest-auth for Token-based Authentication

Django-rest-auth is a package that makes it easy to integrate Django-rest-framework authentication with Django-allauth. We will use it to handle token-based authentication for the API. First, install Django-rest-auth using pip:

pip install django-rest-auth

Next, add rest_auth to the INSTALLED_APPS list in your project's settings.py file:

# settings.py
INSTALLED_APPS = [
    # ...
    'rest_auth',
]


Now, include the rest_auth.urls in your project's urls.py file:

# urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('accounts/', include('allauth.urls')),
    path('api/rest-auth/', include('rest_auth.urls')),
]

1.4 Creating Custom Serializers and Views for User Registration and Authentication

In this section, we will create custom serializers and views to handle user registration and authentication using Django Rest Framework (DRF). This will allow us to customize the registration process and provide additional functionality, such as returning a token upon successful registration.

First, create a new file called serializers.py in the accounts app folder, and add the following code:

# serializers.py
from django.contrib.auth.models import User
from rest_framework import serializers


class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ('id', 'username', 'email', 'first_name', 'last_name')


class UserRegistrationSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ('username', 'email', 'password', 'first_name', 'last_name')
        extra_kwargs = {'password': {'write_only': True}}

    def create(self, validated_data):
        user = User.objects.create_user(
            username=validated_data['username'],
            email=validated_data['email'],
            password=validated_data['password'],
            first_name=validated_data['first_name'],
            last_name=validated_data['last_name']
        )
        return user

The UserSerializer class serializes the User model for use in our API, while the UserRegistrationSerializer class handles user registration by creating a new User instance with the provided data.

Next, create a new file called views.py in the accounts app folder and add the following code:

# views.py
from django.contrib.auth.models import User
from rest_framework import generics, permissions
from rest_framework.authtoken.models import Token
from rest_framework.response import Response
from .serializers import UserSerializer, UserRegistrationSerializer


class UserList(generics.ListAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer
    permission_classes = [permissions.IsAuthenticated]


class UserRegister(generics.CreateAPIView):
    queryset = User.objects.all()
    serializer_class = UserRegistrationSerializer

    def post(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        if serializer.is_valid():
            user = self.perform_create(serializer)
            headers = self.get_success_headers(serializer.data)
            token = Token.objects.get_or_create(user=user)
            return Response({'token': token.key}, status=201, headers=headers)
        return Response(serializer.errors, status=400)
        

    def perform_create(self, serializer):
        return serializer.save()


The UserList view returns a list of all registered users, while the UserRegister view handles user registration and returns a token upon successful registration.

Finally, update the urls.py file in the accounts app folder to include the new views:

# urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('api/users/', views.UserList.as_view(), name='user-list'),
    path('api/register/', views.UserRegister.as_view(), name='user-register'),
]


Now you should be able to register new users and authenticate them using the API endpoints you've created. In the following chapters, we will build upon this foundation to create additional functionality for our Thee.ai social media app.


