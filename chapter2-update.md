Chapter 6: Implementing a User Profile Page
Now that we have a user search functionality, let's create a user profile page where users can view each other's profile information. We will create a new route for the profile page and display the user's details.

6.1 Create a new Django view for the user profile:

In your views.py file, add a new view to fetch the user's details by their ID:

from rest_framework import generics
from .models import User
from .serializers import UserProfileSerializer

class UserProfileView(generics.RetrieveAPIView):
    serializer_class = UserProfileSerializer
    queryset = User.objects.all()
    lookup_field = 'id'

This UserProfileView extends the RetrieveAPIView from the Django Rest Framework and will return a user's details based on their ID.

Next, create the UserProfileSerializer in your serializers.py file:

class UserProfileSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'username', 'first_name', 'last_name', 'email', 'bio']


Add the new view to your urls.py file:



from .views import UserProfileView

urlpatterns = [
    # ...
    path('api/users/<int:id>/', UserProfileView.as_view(), name='user-profile'),
]


6.2 Create a React component for the user profile page:

In your React project, create a new file called user-profile.js and create the user profile component:


import React, { useState, useEffect } from 'react';

function UserProfile({ match }) {
    const [user, setUser] = useState(null);
    const userId = match.params.id;

    useEffect(() => {
        async function fetchUser() {
            const response = await fetch(`/api/users/${userId}/`);
            const data = await response.json();
            setUser(data);
        }
        fetchUser();
    }, [userId]);

    if (!user) {
        return <div>Loading...</div>;
    }

    return (
        <div>
            <h1>{user.username}</h1>
            <p>{user.first_name} {user.last_name}</p>
            <p>Email: {user.email}</p>
            <p>Bio: {user.bio}</p>
        </div>
    );
}

export default UserProfile;

This UserProfile component fetches the user's data from the /api/users/:id endpoint and displays their information on the page.

6.3 Update your React Router configuration to include the user profile route:

In your main application file or wherever you have set up your routes, import the UserProfile component and add a new route for the user profile page:

import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import UserProfile from './user-profile';

function App() {
    return (
        <Router>
            <Switch>
                {/* Other routes */}
                <Route path="/users/:id" component={UserProfile} />
            </Switch>
        </Router>
    );
}

export default App;

Now, users can view each other's profiles by visiting the /users/:id URL. To make it easier to navigate to the profile pages, you can update the UserSearch component to include links to the user profiles in the search results:

import { Link } from 'react-router-dom';

// ...

{searchResults.map((user) => (
    <div key={user.id}>
        <h4>
            <Link to={`/users/${user.id}`}>{user.username}</Link>
        </h4>
        <p>{user.first_name} {user.last_name}</p>
    </div>
))}
// ...

This code snippet adds a Link component from react-router-dom for each user in the search results. Now, when users click on the username in the search results, they will be taken to the corresponding user's profile page.

With the user search and user profile functionality in place, you have successfully built a feature-rich social networking application using Django and React. You can continue to enhance your application by adding features such as user authentication, friend requests, and user posts.

Here's a summary of the steps we took in creating our social networking application:

Set up a Django project and create a new app
Define the User model and create serializers for the API
Implement Django views for user search and user profile
Configure the Django URL patterns for the API endpoints
Set up a React project and create components for user search and user profile
Implement user search functionality in the React application
Create a user profile page in the React application
Update the React Router configuration to include the user profile route
Add links to the user profile pages in the search results

As you continue to develop your application, consider implementing additional features such as user authentication, private messaging, and real-time updates using websockets.

