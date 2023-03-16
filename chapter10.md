Chapter 10: Notifications:

24.1 Use React components to create a notifications page:
Start by creating a new folder named notifications in the src/components directory. Inside this folder, create a new file called Notifications.js. This file will contain the main component for displaying the user's notifications.

src/components/notifications/Notifications.js:

import React, { useEffect, useState } from 'react';
import axios from 'axios';
import NotificationItem from './NotificationItem';

const Notifications = () => {
  const [notifications, setNotifications] = useState([]);

  useEffect(() => {
    // Fetch notifications from the API
    const fetchNotifications = async () => {
      try {
        const response = await axios.get('/api/notifications/');
        setNotifications(response.data);
      } catch (error) {
        console.error('Error fetching notifications:', error);
      }
    };

    fetchNotifications();
  }, []);

  return (
    <div className="notifications-container">
      <h2>Notifications</h2>
      {notifications.map((notification) => (
        <NotificationItem key={notification.id} notification={notification} />
      ))}
    </div>
  );
};

export default Notifications;


Next, create a new file named NotificationItem.js in the src/components/notifications folder. This file will contain a functional component that takes a notification object as a prop and renders the notification item.

src/components/notifications/NotificationItem.js:

import React from 'react';

const NotificationItem = ({ notification }) => {
  return (
    <div className="notification-item">
      <p>{notification.message}</p>
      <span>{new Date(notification.timestamp).toLocaleString()}</span>
    </div>
  );
};

export default NotificationItem;


24.2 Use axios to make HTTP requests to the back-end API for retrieving notifications:
In the Notifications.js file, we are using axios to fetch notifications from the back-end API. The API should return an array of notification objects, which we save in the notifications state variable.

24.3 Use Redux to manage the state of the user's notifications:
To manage the notifications state globally, create a new Redux slice called notificationsSlice.js in the src/store folder. This slice will handle actions related to fetching and updating notifications.

src/store/notificationsSlice.js:

import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import axios from 'axios';

// Async thunk for fetching notifications
export const fetchNotifications = createAsyncThunk(
  'notifications/fetchNotifications',
  async () => {
    const response = await axios.get('/api/notifications/');
    return response.data;
  }
);

const notificationsSlice = createSlice({
  name: 'notifications',
  initialState: [],
  reducers: {},
  extraReducers: (builder) => {
    builder.addCase(fetchNotifications.fulfilled, (state, action) => {
      return action.payload;
    });
  },
});

export default notificationsSlice.reducer;

Update the store.js file to include the notifications reducer:


import { configureStore } from '@reduxjs/toolkit';
import userReducer from './userSlice';
import notificationsReducer from './notificationsSlice';

export default configureStore({
  reducer: {
    user: userReducer,
    notifications: notificationsReducer,
  },
});

Now, update the Notifications.js component to use the Redux state and actions. Replace the useState and useEffect hooks with the useSelector and useDispatch hooks from react-redux.

import React, { useEffect } from 'react';
import { useSelector, useDispatch } from 'react-redux';
import axios from 'axios';
import NotificationItem from './NotificationItem';
import { fetchNotifications } from '../../store/notificationsSlice';

const Notifications = () => {
const notifications = useSelector((state) => state.notifications);
const dispatch = useDispatch();

useEffect(() => {
dispatch(fetchNotifications());
}, [dispatch]);

return (
<div className="notifications-container">
<h2>Notifications</h2>
{notifications.map((notification) => (
<NotificationItem key={notification.id} notification={notification} />
))}
</div>
);
};

export default Notifications;


24.4 Update the `App.js` file to include the new `Notifications` component:
Now that the `Notifications` component is ready, add it to the `App.js` file so users can view their notifications.

```javascript
import React from 'react';
import './App.css';
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import Home from './components/Home';
import SignUp from './components/SignUp';
import SignIn from './components/SignIn';
import UserProfile from './components/userProfile/UserProfile';
import Notifications from './components/notifications/Notifications';

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
        </Switch>
      </div>
    </Router>
  );
}

export default App;

With the completion of this chapter, users can now view their notifications in the application. The notifications state is managed globally with Redux, allowing for easy access and updates across the application.

