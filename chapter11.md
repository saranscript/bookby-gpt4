Chapter 11: Trends:

25.1 Use React components to create a trends page:
Create a new component called Trends which will be used to display trending hashtags and topics.

// src/components/trends/Trends.js
import React, { useEffect } from 'react';
import { useSelector, useDispatch } from 'react-redux';
import axios from 'axios';
import TrendItem from './TrendItem';
import { fetchTrends } from '../../store/trendsSlice';

const Trends = () => {
  const trends = useSelector((state) => state.trends);
  const dispatch = useDispatch();

  useEffect(() => {
    dispatch(fetchTrends());
  }, [dispatch]);

  return (
    <div className="trends-container">
      <h2>Trending Hashtags & Topics</h2>
      {trends.map((trend) => (
        <TrendItem key={trend.id} trend={trend} />
      ))}
    </div>
  );
};

export default Trends;


25.2 Use axios to make HTTP requests to the back end API for retrieving trending hashtags and topics:
Create a Redux slice for the trends state management.

// src/store/trendsSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import axios from 'axios';

export const fetchTrends = createAsyncThunk('trends/fetchTrends', async () => {
  const response = await axios.get('/api/trends/');
  return response.data;
});

const trendsSlice = createSlice({
  name: 'trends',
  initialState: [],
  reducers: {},
  extraReducers: (builder) => {
    builder.addCase(fetchTrends.fulfilled, (state, action) => {
      return action.payload;
    });
  },
});

export default trendsSlice.reducer;

Add the trends reducer to the store:

// src/store/store.js
import { configureStore } from '@reduxjs/toolkit';
import authReducer from './authSlice';
import userProfileReducer from './userProfileSlice';
import trendsReducer from './trendsSlice';

export default configureStore({
  reducer: {
    auth: authReducer,
    userProfile: userProfileReducer,
    trends: trendsReducer,
  },
});

25.3 Use Redux to manage the state of the trending hashtags and topics:
Create a TrendItem component to render individual trends.

// src/components/trends/TrendItem.js
import React from 'react';

const TrendItem = ({ trend }) => {
  return (
    <div className="trend-item">
      <h4>{trend.hashtag}</h4>
      <p>{trend.topic}</p>
    </div>
  );
};

export default TrendItem;

Update the App.js file to include the new Trends component:


import React from 'react';
import './App.css';
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import Home from './components/Home';
import SignUp from './components/SignUp';
import SignIn from './components/SignIn';
import UserProfile from './components/userProfile/UserProfile';
import Notifications from './components/notifications/Notifications';
import Trends from './components/trends/Trends';

function App() {
  return (
    <Router>
      <div className="App">
        <Switch>
          <Route path="/" exact component={Home} />
          <Route path="/signup" component={SignUp} />
          <Route path="/signin" component={SignIn} />
          <Route path="/profile" component={UserProfile} />
          <Route path="/notifications" component={Notifications} />
          <Route path="/trends" component={Trends} />
       
    </Switch>
  </div>
</Router>
);
}
export default App;


Now, your social media application has a trends page that displays trending hashtags and topics. Users can visit the trends page to see what's popular on the platform. This feature helps users discover new content and stay informed about current events and popular discussions.

In this guide, we covered the process of creating various features for a social media application using React, Redux, Django, and PostgreSQL. We touched upon authentication, user profiles, notifications, and trends. The application can now be further expanded by adding more features like direct messaging, groups, and multimedia content support.

