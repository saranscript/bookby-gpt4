Chapter 15: User registration and authentication:

15.1 Creating user registration and login forms using React components:
To create a user registration and login form, use functional components and React hooks. Begin by creating the necessary components and their corresponding state.

import React, { useState } from 'react';

const RegistrationForm = () => {
  const [email, setEmail] = useState('');
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  const [confirmPassword, setConfirmPassword] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    // handle form submission logic
  };

  return (
    <form onSubmit={handleSubmit}>
      // form fields and buttons go here
    </form>
  );
};

const LoginForm = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    // handle form submission logic
  };

  return (
    <form onSubmit={handleSubmit}>
      // form fields and buttons go here
    </form>
  );
};


15.2 Using axios to make HTTP requests for authentication and token handling:
Install axios and make HTTP requests to your back-end API for user registration and authentication.

import axios from 'axios';

// Registration request
const registerUser = async (email, username, password) => {
  try {
    const response = await axios.post('/api/register', {
      email,
      username,
      password,
    });
    return response.data;
  } catch (error) {
    console.error(error);
  }
};

// Authentication request
const authenticateUser = async (email, password) => {
  try {
    const response = await axios.post('/api/login', {
      email,
      password,
    });
    return response.data;
  } catch (error) {
    console.error(error);
  }
};


15.3 Using Redux to manage user authentication state:
Set up Redux to manage the state of user authentication in your application. Create the necessary actions and reducers to handle user registration, login, and logout.


// actions.js
export const userLoggedIn = (user) => ({
  type: 'USER_LOGGED_IN',
  payload: user,
});

export const userLoggedOut = () => ({
  type: 'USER_LOGGED_OUT',
});

// reducer.js
const initialState = {
  user: null,
  isAuthenticated: false,
};

export const authReducer = (state = initialState, action) => {
  switch (action.type) {
    case 'USER_LOGGED_IN':
      return {
        ...state,
        user: action.payload,
        isAuthenticated: true,
      };
    case 'USER_LOGGED_OUT':
      return initialState;
    default:
      return state;
  }
};

15.4 Using React Router to manage page routing based on user authentication status:
Leverage React Router to manage navigation in your application. Set up routes that require authentication using a higher-order component or a custom route component to protect them.


import React from 'react';
import { useSelector } from 'react-redux';
import { Route, Redirect } from 'react-router-dom';

const PrivateRoute = ({ component: Component, ...rest }) => {
  const isAuthenticated = useSelector((state) => state.auth.isAuthenticated);

  return (
    <Route
      {...rest}
      render={(props) =>
        isAuthenticated ? <Component {...props} /> : <Redirect to="/login" />
      }
    />
  );
};

export default PrivateRoute;

Use the PrivateRoute component to protect routes that require authentication:


import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import PrivateRoute from './PrivateRoute';
import Home from './pages/Home';
import Login from './pages/Login';
import Register from './pages/Register';

const App = () => {
  return (
    <Router>
      <Switch>
        <PrivateRoute exact path="/" component={Home} />
        <Route path="/login" component={Login} />
        <Route path="/register" component={Register} />
        {/* Add more routes as needed */}
      </Switch>
    </Router>
  );
};

export default App;

This code snippet demonstrates how to use React Router with the PrivateRoute component to manage routing based on user authentication status. The PrivateRoute component is used to protect the routes that require authentication, such as the home page. The login and registration routes are accessible without authentication, and you can add more routes as needed for your application.

Now that we have set up the basic routing for our application, we can focus on implementing the components for each page. Let's start with the Home component.

// src/pages/Home.js
import React from 'react';

const Home = () => {
  return (
    <div>
      <h1>Welcome to the Home Page</h1>
      {/* Add your home page content here */}
    </div>
  );
};

export default Home;

Next, let's create the Login component:


// src/pages/Login.js
import React, { useState } from 'react';

const Login = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    // Perform authentication logic here
  };

  return (
    <div>
      <h1>Login</h1>
      <form onSubmit={handleSubmit}>
        <label htmlFor="email">Email:</label>
        <input
          type="email"
          id="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
        />
        <br />
        <label htmlFor="password">Password:</label>
        <input
          type="password"
          id="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
        />
        <br />
        <button type="submit">Login</button>
      </form>
    </div>
  );
};

export default Login;

Finally, let's create the Register component:


// src/pages/Register.js
import React, { useState } from 'react';

const Register = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    // Perform registration logic here
  };

  return (
    <div>
      <h1>Register</h1>
      <form onSubmit={handleSubmit}>
        <label htmlFor="email">Email:</label>
        <input
          type="email"
          id="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
        />
        <br />
        <label htmlFor="password">Password:</label>
        <input
          type="password"
          id="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
        />
        <br />
        <button type="submit">Register</button>
      </form>
    </div>
  );
};

export default Register;

Now you have the basic structure of your application's front-end, with components for the home, login, and registration pages. You can continue to build out these components by adding more features, styles, and interactivity, as needed for your specific project.


In the next step, let's implement a simple navigation bar to help users navigate between the different pages. We will create a new component called NavBar.

// src/components/NavBar.js
import React from 'react';
import { Link } from 'react-router-dom';

const NavBar = () => {
  return (
    <nav>
      <ul>
        <li>
          <Link to="/">Home</Link>
        </li>
        <li>
          <Link to="/login">Login</Link>
        </li>
        <li>
          <Link to="/register">Register</Link>
        </li>
      </ul>
    </nav>
  );
};

export default NavBar;

Now, let's include the NavBar component in our App component:

// src/App.js
import React from 'react';
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import NavBar from './components/NavBar';
import Home from './pages/Home';
import Login from './pages/Login';
import Register from './pages/Register';

const App = () => {
  return (
    <Router>
      <NavBar />
      <Switch>
        <Route exact path="/" component={Home} />
        <Route path="/login" component={Login} />
        <Route path="/register" component={Register} />
      </Switch>
    </Router>
  );
};

export default App;

You now have a simple navigation bar that links to each of the pages in your application. You can further customize the appearance and functionality of the NavBar component by adding additional links, dropdown menus, and styling.

For a better user experience, you may want to implement a user authentication system that allows users to log in and out, and show different menu options based on the user's authentication status. In the next chapter, we will explore how to integrate a user authentication system into your application using Redux for state management and a back-end API to handle user authentication.