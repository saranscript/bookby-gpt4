Chapter 5: User search
In this chapter, we'll implement user search functionality, allowing users to search for other users by their username, name, or bio information.

5.1 Use Django Rest Framework to create API endpoints to search for users:

First, we need to  search for users. We'll use Django's built-in search functionality and Django Rest Framework to achieve this. In your views.py file, create a new view to handle user search:

from django.db.models import Q
from rest_framework import generics
from .models import User
from .serializers import UserSearchSerializer

class UserSearchView(generics.ListAPIView):
    serializer_class = UserSearchSerializer
    queryset = User.objects.all()

    def get_queryset(self):
        query = self.request.query_params.get('query', '')
        queryset = User.objects.filter(
            Q(username__icontains=query) | Q(first_name__icontains=query) | Q(last_name__icontains=query)
        )
        return queryset


Here, we create a UserSearchView that extends ListAPIView. We use the Q object to perform OR queries, searching the User model for matches in the username, first_name, or last_name fields. The get_queryset method retrieves the search query from the request and returns the filtered queryset.

Next, create the UserSearchSerializer in your serializers.py file:

class UserSearchSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'username', 'first_name', 'last_name']

Now, add the new view to your urls.py file:

from .views import UserSearchView

urlpatterns = [
    # ...
    path('api/users/search/', UserSearchView.as_view(), name='user-search'),
]

5.2 Use React components to create a user search form:

Create a new file called user-search.js and create a search form using React components:

import React, { useState } from 'react';

function UserSearch() {
    const [query, setQuery] = useState('');
    const [searchResults, setSearchResults] = useState([]);

    async function handleSearch(e) {
        e.preventDefault();

        const response = await fetch(`/api/users/search/?query=${query}`);
        const data = await response.json();
        setSearchResults(data);
    }

    return (
        <div>
            <form onSubmit={handleSearch}>
                <input
                    type="text"
                    placeholder="Search users..."
                    value={query}
                    onChange={(e) => setQuery(e.target.value)}
                />
                <button type="submit">Search</button>
            </form>
            <div>
                {searchResults.map((user) => (
                    <div key={user.id}>
                        <h4>{user.username}</h4>
                        <p>{user.first_name} {user.last_name}</p>
                    </div>
                ))}
            </div>
        </div>
    );
}

export default UserSearch;

In this component, we're using the useState hook to manage the search query and search results. When the form is submitted, the handleSearch function sends a request to the /api/users/search/ endpoint with the query parameter. The search results are then stored in the searchResults state and displayed on the page.

5.3 Integrate the user search component into your application:

Now that you have created the UserSearch component, import and use it in your main application file or wherever you want to display the user search functionality.

import UserSearch from './user-search';

function App() {
    return (
        <div>
            {/* Other components */}
            <UserSearch />
        </div>
    );
}

export default App;


Your users can now search for other users by their username, name, or bio information.


