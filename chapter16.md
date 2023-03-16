Chapter 16: Implementing User Authentication with Redux and API Integration

In this chapter, we will implement a user authentication system in our application using Redux for state management and a back-end API to handle user authentication.

1. First, let's set up Redux. Install the necessary packages:
npm install redux react-redux redux-thunk

2. Create a new folder called redux inside the src directory and create the following files inside it:

src
 └── redux
       ├── actions
       │    └── auth.js
       ├── constants
       │    └── auth.js
       ├── reducers
       │    └── auth.js
       └── store.js

3. Set up the Redux store in store.js:

// src/redux/store.js
import { createStore, applyMiddleware, combineReducers } from 'redux';
import thunk from 'redux-thunk';
import authReducer from './reducers/auth';

const rootReducer = combineReducers({
  auth: authReducer,
});

const store = createStore(rootReducer, applyMiddleware(thunk));

export default store;

4. Now, let's create the action types for user authentication in auth.js:

// src/redux/constants/auth.js
export const AUTH_REQUEST = 'AUTH_REQUEST';
export const AUTH_SUCCESS = 'AUTH_SUCCESS';
export const AUTH_FAILURE = 'AUTH_FAILURE';
export const LOGOUT = 'LOGOUT';

5. Next, let's create the authentication actions in auth.js:

// src/redux/actions/auth.js
import { AUTH_REQUEST, AUTH_SUCCESS, AUTH_FAILURE, LOGOUT } from '../constants/auth';

export const authRequest = () => ({
  type: AUTH_REQUEST,
});

export const authSuccess = (user) => ({
  type: AUTH_SUCCESS,
  payload: user,
});

export const authFailure = (error) => ({
  type: AUTH_FAILURE,
  payload: error,
});

export const logout = () => ({
  type: LOGOUT,
});

6. Let's create the authentication reducer in auth.js:

// src/redux/reducers/auth.js
import { AUTH_REQUEST, AUTH_SUCCESS, AUTH_FAILURE, LOGOUT } from '../constants/auth';

const initialState = {
  isAuthenticated: false,
  user: null,
  loading: false,
  error: null,
};

const authReducer = (state = initialState, action) => {
  switch (action.type) {
    case AUTH_REQUEST:
      return { ...state, loading: true, error: null };
    case AUTH_SUCCESS:
      return { ...state, isAuthenticated: true, user: action.payload, loading: false };
    case AUTH_FAILURE:
      return { ...state, isAuthenticated: false, user: null, loading: false, error: action.payload };
    case LOGOUT:
      return { ...state, isAuthenticated: false, user: null };
    default:
      return state;
  }
};

export default authReducer;


7. Now, let's connect our Redux store to the application by wrapping the App component with the Provider component:

// src/index.js
import React from 'react';
import ReactDOM from 'react-dom';
import { Provider } from 'react-redux';
import store from './redux/store';
import App from './App';

ReactDOM.render(
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>,
  document.getElementById('root')
);

8. Next, let's create a function to handle user authentication. We will assume that we have a back-end API available at https://api.example.com/auth that accepts a POST request with email and password as parameters and returns a JSON object containing user information and a token if the authentication is successful.

Create a new file called authApi.js inside the src directory:
src
 └── authApi.js

Add the following code to authApi.js:

// src/authApi.js
export const authenticateUser = async (email, password) => {
  try {
    const response = await fetch('https://api.example.com/auth', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ email, password }),
    });

    if (!response.ok) {
      throw new Error('Authentication failed');
    }

    const data = await response.json();
    return data;
  } catch (error) {
    throw error;
  }
};

9. Now, let's create a Redux action creator that utilizes the authenticateUser function to authenticate users. Update the auth.js file in the actions folder:

// src/redux/actions/auth.js
import { AUTH_REQUEST, AUTH_SUCCESS, AUTH_FAILURE, LOGOUT } from '../constants/auth';
import { authenticateUser } from '../../authApi';

export const authRequest = () => ({
  type: AUTH_REQUEST,
});

export const authSuccess = (user) => ({
  type: AUTH_SUCCESS,
  payload: user,
});

export const authFailure = (error) => ({
  type: AUTH_FAILURE,
  payload: error,
});

export const logout = () => ({
  type: LOGOUT,
});

export const loginUser = (email, password) => async (dispatch) => {
  dispatch(authRequest());

  try {
    const user = await authenticateUser(email, password);
    dispatch(authSuccess(user));
  } catch (error) {
    dispatch(authFailure(error.message));
  }
};

10. Finally, let's create a simple login form and connect it to our Redux store. Create a new file called LoginForm.js inside the src directory:
src
 └── LoginForm.js

Add the following code to LoginForm.js:

// src/LoginForm.js
import React, { useState } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { loginUser } from './redux/actions/auth';

const LoginForm = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const dispatch = useDispatch();
  const error = useSelector((state) => state.auth.error);

  const handleSubmit = (e) => {
    e.preventDefault();
    dispatch(loginUser(email, password));
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        placeholder="Email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />
      <input
        type="password"
        placeholder="Password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
      />
      {error && <p>{error}</p>}
      <button type="submit">Log in</button>
    </form>
  );
};

export default LoginForm;

Now, you can include the LoginForm component in your application and use it to authenticate users using Redux and the provided API.



11. In addition to the login form, let's create a simple protected component that can only be accessed by authenticated users. Create a new file called ProtectedComponent.js inside the src directory:
src
 └── ProtectedComponent.js

Add the following code to ProtectedComponent.js:

// src/ProtectedComponent.js
import React from 'react';
import { useSelector } from 'react-redux';

const ProtectedComponent = () => {
  const user = useSelector((state) => state.auth.user);

  if (!user) {
    return <div>You must be logged in to view this content.</div>;
  }

  return <div>Welcome, {user.name}! You have successfully accessed the protected content.</div>;
};

export default ProtectedComponent;


12. To manage route access based on authentication, let's create a higher-order component called PrivateRoute which wraps around Route from react-router-dom. Create a new file called PrivateRoute.js inside the src directory:

src
 └── PrivateRoute.js

Add the following code to PrivateRoute.js:

// src/PrivateRoute.js
import React from 'react';
import { Route, Redirect } from 'react-router-dom';
import { useSelector } from 'react-redux';

const PrivateRoute = ({ component: Component, ...rest }) => {
  const user = useSelector((state) => state.auth.user);

  return (
    <Route
      {...rest}
      render={(props) =>
        user ? (
          <Component {...props} />
        ) : (
          <Redirect
            to={{
              pathname: '/login',
              state: { from: props.location },
            }}
          />
        )
      }
    />
  );
};

export default PrivateRoute;

13. Update the App.js file to include the LoginForm, ProtectedComponent, and PrivateRoute. Replace the existing code with the following:

// src/App.js
import React from 'react';
import { BrowserRouter as Router, Switch, Route } from 'react-router-dom';
import LoginForm from './LoginForm';
import ProtectedComponent from './ProtectedComponent';
import PrivateRoute from './PrivateRoute';

const App = () => {
  return (
    <Router>
      <Switch>
        <Route path="/login" component={LoginForm} />
        <PrivateRoute path="/protected" component={ProtectedComponent} />
      </Switch>
    </Router>
  );
};

export default App;

Now, when you run your application, you will have a login form that authenticates users and a protected component that can only be accessed by authenticated users. If a user tries to access the protected component without being authenticated, they will be redirected to the login page.

You can further expand this front-end application by adding more components, connecting them to the Redux store, and using the API to fetch and manage data.


