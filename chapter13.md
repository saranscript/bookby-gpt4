Chapter 13: Analytics

In this chapter, we will create an analytics page that displays various insights and statistics related to a user's account. We will use React components to create the analytics page, axios to make HTTP requests to the back end API for retrieving user analytics data, and Redux to manage the state of the user's analytics data.

1. Create an Analytics model in the Django back end.
In the analytics/models.py file, create a new Analytics model to store user analytics data.

# analytics/models.py
from django.db import models
from django.contrib.auth.models import User

class Analytics(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    total_posts = models.IntegerField(default=0)
    total_likes = models.IntegerField(default=0)
    total_followers = models.IntegerField(default=0)
    total_following = models.IntegerField(default=0)

    def __str__(self):
        return f"{self.user.username}'s Analytics"


2. Create a view to retrieve the user analytics data.
In the analytics/views.py file, create a new view to handle retrieving user analytics data.

# analytics/views.py
from rest_framework.decorators import api_view, permission_classes
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response
from .models import Analytics
from .serializers import AnalyticsSerializer

@api_view(['GET'])
@permission_classes([IsAuthenticated])
def get_user_analytics(request):
    user = request.user
    analytics = Analytics.objects.get(user=user)
    serialized_analytics = AnalyticsSerializer(analytics).data
    return Response(serialized_analytics)


3. Add a new URL pattern to handle the get user analytics API endpoint.
In analytics/urls.py, add the following path to handle retrieving user analytics data.
# analytics/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('get_user_analytics/', views.get_user_analytics, name='get_user_analytics'),
]

4. Create a new React component for the analytics page.
In the src/components/Analytics folder, create a new AnalyticsPage.js file for the analytics page component.

// src/components/Analytics/AnalyticsPage.js
import React, { useEffect } from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { getUserAnalytics } from '../../redux/actions/analyticsActions';

const AnalyticsPage = () => {
  const dispatch = useDispatch();
  const analytics = useSelector((state) => state.analytics);

  useEffect(() => {
    dispatch(getUserAnalytics());
  }, [dispatch]);

  return (
    <div className="analytics-page">
      <h2>Your Analytics</h2>
      <div className="analytics-container">
        <div>Total Posts: {analytics.total_posts}</div>
        <div>Total Likes: {analytics.total_likes}</div>
        <div>Total Followers: {analytics.total_followers}</div>
        <div>Total Following: {analytics.total_following}</div>
      </div>
    </div>
  );
};

export default AnalyticsPage;

5. Add a new route for the analytics page in the src/App.js file.

// src/App.js
import React from 'react';
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import AnalyticsPage from './components/Analytics/AnalyticsPage';

// ...

function App() {
  return (
    <Router>
      {/* ... */}
      <Switch>
        {/* ... */}
        <Route path="/analytics" component={AnalyticsPage} />
      </Switch>
    </Router>
  );
}

export default App;

Now, users can access their analytics by navigating to the /analytics route in the application. The analytics page will display various insights and statistics related to a user's account, such as the total number of posts, likes, followers, and users they are following.

6. Create Redux actions for user analytics.
In the src/redux/actions/analyticsActions.js file, create a new action for fetching user analytics data.

// src/redux/actions/analyticsActions.js
import axios from 'axios';
import { GET_USER_ANALYTICS } from '../constants/analyticsConstants';

export const getUserAnalytics = () => async (dispatch, getState) => {
  try {
    const { userLogin: { userInfo } } = getState();
    const config = {
      headers: {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${userInfo.token}`,
      },
    };

    const { data } = await axios.get('/api/analytics/get_user_analytics/', config);
    dispatch({ type: GET_USER_ANALYTICS, payload: data });
  } catch (error) {
    console.error(error);
  }
};

7. Create Redux constants for user analytics.
In the src/redux/constants/analyticsConstants.js file, create a new constant for the GET_USER_ANALYTICS action.

// src/redux/constants/analyticsConstants.js
export const GET_USER_ANALYTICS = 'GET_USER_ANALYTICS';

8. Create a Redux reducer for user analytics.
In the src/redux/reducers/analyticsReducer.js file, create a new reducer to handle the GET_USER_ANALYTICS action.

// src/redux/reducers/analyticsReducer.js
import { GET_USER_ANALYTICS } from '../constants/analyticsConstants';

const initialState = {
  total_posts: 0,
  total_likes: 0,
  total_followers: 0,
  total_following: 0,
};

export const analyticsReducer = (state = initialState, action) => {
  switch (action.type) {
    case GET_USER_ANALYTICS:
      return {
        ...state,
        ...action.payload,
      };
    default:
      return state;
  }
};


9. Combine the analytics reducer with the other reducers.
In the src/redux/store.js file, add the analytics reducer to the combineReducers call.

// src/redux/store.js
import { createStore, combineReducers, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import { composeWithDevTools } from 'redux-devtools-extension';
import { analyticsReducer } from './reducers/analyticsReducer';

const reducer = combineReducers({
  // ...
  analytics: analyticsReducer,
});

// ...

Now, your application has a fully functional analytics page that displays various insights and statistics related to a user's account. The analytics data is fetched from the Django back end and stored in the Redux store.


10. Create the Analytics component.
In the src/components/Analytics.js file, create a new functional component that will display the user's analytics information.

// src/components/Analytics.js
import React, { useEffect } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { getUserAnalytics } from '../redux/actions/analyticsActions';

const Analytics = () => {
  const dispatch = useDispatch();
  const { total_posts, total_likes, total_followers, total_following } = useSelector(state => state.analytics);

  useEffect(() => {
    dispatch(getUserAnalytics());
  }, [dispatch]);

  return (
    <div className="analytics">
      <h2>User Analytics</h2>
      <div className="analytics-info">
        <p>Total Posts: {total_posts}</p>
        <p>Total Likes: {total_likes}</p>
        <p>Total Followers: {total_followers}</p>
        <p>Total Following: {total_following}</p>
      </div>
    </div>
  );
};

export default Analytics;

11. Add the Analytics component to the App component.

In the src/App.js file, import the Analytics component and add it as a route in the application.

// src/App.js
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import Analytics from './components/Analytics';
// ...

const App = () => {
  return (
    <Router>
      {/* ... */}
      <Switch>
        {/* ... */}
        <Route path="/analytics" component={Analytics} />
      </Switch>
    </Router>
  );
};

export default App;

12. Add a link to the Analytics page in the navigation bar.

In the src/components/Header.js file, add a new link to the Analytics page.

// src/components/Header.js
import { Link } from 'react-router-dom';
// ...

const Header = () => {
  // ...
  return (
    <header>
      {/* ... */}
      <nav>
        <ul>
          {/* ... */}
          <li>
            <Link to="/analytics">Analytics</Link>
          </li>
        </ul>
      </nav>
    </header>
  );
};

export default Header;

With these changes, your application now includes a fully functional analytics page that displays user-specific insights and statistics. The analytics data is fetched from the Django back end, stored in the Redux store, and displayed using React components. Users can access their analytics by navigating to the /analytics route in the application, and the navigation bar now includes a link to the Analytics page for quick access.

Now, let's add some interactivity to the analytics page by allowing users to filter their statistics by date.

13. Update the Analytics component to include a date filter.
Modify the src/components/Analytics.js file to include a date input field that users can use to filter their analytics data.

// src/components/Analytics.js
import React, { useState, useEffect } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { getUserAnalytics } from '../redux/actions/analyticsActions';

const Analytics = () => {
  const dispatch = useDispatch();
  const { total_posts, total_likes, total_followers, total_following } = useSelector(state => state.analytics);
  const [filterDate, setFilterDate] = useState('');

  useEffect(() => {
    dispatch(getUserAnalytics(filterDate));
  }, [dispatch, filterDate]);

  const handleDateChange = (event) => {
    setFilterDate(event.target.value);
  };

  return (
    <div className="analytics">
      <h2>User Analytics</h2>
      <div className="analytics-filter">
        <label htmlFor="filterDate">Filter by date:</label>
        <input
          type="date"
          id="filterDate"
          name="filterDate"
          value={filterDate}
          onChange={handleDateChange}
        />
      </div>
      <div className="analytics-info">
        <p>Total Posts: {total_posts}</p>
        <p>Total Likes: {total_likes}</p>
        <p>Total Followers: {total_followers}</p>
        <p>Total Following: {total_following}</p>
      </div>
    </div>
  );
};

export default Analytics;

14. Update the getUserAnalytics action creator to accept a filterDate parameter.
Modify the src/redux/actions/analyticsActions.js file to include a filterDate parameter in the getUserAnalytics action creator. This parameter will be passed to the API call as a query parameter.

// src/redux/actions/analyticsActions.js
// ...
export const getUserAnalytics = (filterDate) => async (dispatch) => {
  try {
    dispatch({ type: GET_USER_ANALYTICS_REQUEST });

    const { data } = await axios.get(`/api/analytics?filterDate=${filterDate}`);

    dispatch({ type: GET_USER_ANALYTICS_SUCCESS, payload: data });
  } catch (error) {
    dispatch({
      type: GET_USER_ANALYTICS_FAIL,
      payload: error.response && error.response.data.message
        ? error.response.data.message
        : error.message,
    });
  }
};


15. Update the Django back end to handle the filterDate query parameter.
Update the views.py file in your Django application to filter the analytics data based on the filterDate query parameter.

# views.py
from datetime import datetime
# ...

@api_view(['GET'])
def get_user_analytics(request):
    # ...
    filter_date = request.GET.get('filterDate', None)
    if filter_date:
        filter_date = datetime.strptime(filter_date, '%Y-%m-%d').date()
        total_posts = Post.objects.filter(user=user, created_at__date=filter_date).count()
        total_likes = Like.objects.filter(post__user=user, post__created_at__date=filter_date).count()
    # ...

    return Response({
        'total_posts': total_posts,
        'total_likes': total_likes,
        'total_followers': total_followers,
        'total_following': total_following,
    })

With these updates, your application now allows users to filter their analytics data by date. When a



user selects a date, the front end sends the filter date to the back end, which filters the data accordingly and returns the updated analytics information to the user.

16. Add some styling to the analytics page.
To make the analytics page more visually appealing, you can add some basic styling to the src/components/Analytics.css file.

/* src/components/Analytics.css */
.analytics {
  width: 100%;
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  box-sizing: border-box;
  background-color: #f1f1f1;
  border-radius: 5px;
  font-family: Arial, sans-serif;
}

.analytics h2 {
  margin-top: 0;
}

.analytics-filter {
  margin-bottom: 20px;
}

.analytics-filter label {
  display: inline-block;
  margin-right: 10px;
}

.analytics-info p {
  margin-bottom: 10px;
}

Now, the analytics page should have a cleaner layout and look more polished.

You have successfully implemented a basic analytics feature into the React and Django application. Users can now view their total posts, likes, followers, and following and filter the results by date. This analytics feature can be further expanded by adding more metrics, visualizations, and filters to provide users with even more insights into their social media activity.




