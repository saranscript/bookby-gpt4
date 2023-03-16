Chapter 12: Privacy and security.

In this chapter, we will focus on implementing privacy and security features in our social media application. We will ensure data transmission is secure, create a privacy and security settings page, and enable users to update their privacy and security settings.

1. Use HTTPS for all data transmission to ensure privacy and security.
To ensure that all data transmission is secure and private, we will use HTTPS. HTTPS encrypts the data between the user's browser and the server, preventing unauthorized access to sensitive information.

In production, you should configure your server to use HTTPS. If you are using a cloud service like AWS or Digital Ocean, refer to their documentation on how to configure HTTPS for your server. Additionally, make sure to obtain an SSL certificate from a certificate authority (CA) like Let's Encrypt.

2. Use React components to create a privacy and security settings page.

Create a new React component called PrivacySecuritySettings.js in the src/components folder. This component will render a form for users to update their privacy and security settings.

// src/components/PrivacySecuritySettings.js
import React, { useState } from 'react';
import { useDispatch } from 'react-redux';
import { updatePrivacySettings } from '../redux/actions/userActions';

const PrivacySecuritySettings = () => {
  const [privacySettings, setPrivacySettings] = useState({
    privateAccount: false,
    twoFactorAuth: false,
  });
  const dispatch = useDispatch();

  const handleChange = (event) => {
    setPrivacySettings({
      ...privacySettings,
      [event.target.name]: event.target.checked,
    });
  };

  const handleSubmit = (event) => {
    event.preventDefault();
    dispatch(updatePrivacySettings(privacySettings));
  };

  return (
    <div className="privacy-security-settings">
      <form onSubmit={handleSubmit}>
        <label>
          <input
            type="checkbox"
            name="privateAccount"
            checked={privacySettings.privateAccount}
            onChange={handleChange}
          />
          Private Account
        </label>
        <label>
          <input
            type="checkbox"
            name="twoFactorAuth"
            checked={privacySettings.twoFactorAuth}
            onChange={handleChange}
          />
          Two Factor Authentication
        </label>
        <button type="submit">Save Settings</button>
      </form>
    </div>
  );
};

export default PrivacySecuritySettings;

3. Use axios to make HTTP requests to the back end API for updating user privacy and security settings.
In src/redux/actions/userActions.js, add the following action creator to handle updating user privacy and security settings.

// src/redux/actions/userActions.js
import axios from 'axios';
import { UPDATE_PRIVACY_SETTINGS } from './types';

export const updatePrivacySettings = (privacySettings) => async (dispatch) => {
  try {
    const config = {
      headers: {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${localStorage.getItem('token')}`,
      },
    };

    const body = JSON.stringify(privacySettings);

    await axios.patch('/api/v1/users/update_privacy_settings/', body, config);

    dispatch({
      type: UPDATE_PRIVACY_SETTINGS,
      payload: privacySettings,
    });
  } catch (error) {
    console.log('Error updating privacy settings:', error);
  }
};

4. Use Redux to manage the state of the user's privacy and security settings.
In src/redux/reducers/userReducer.js, add a case for the UPDATE_PRIVACY_SETTINGS action type.

// src/redux/reducers/userReducer.js
import { UPDATE_PRIVACY_SETTINGS } from '../actions/types';

const initialState = {
  // ...
  privacySettings: {
    privateAccount: false,
    twoFactorAuth: false,
  },
};

const userReducer = (state = initialState, action) => {
  switch (action.type) {
    // ...
    case UPDATE_PRIVACY_SETTINGS:
      return {
        ...state,
        privacySettings: action.payload,
      };
    default:
      return state;
  }
};

export default userReducer;

5. Update the Django back end to handle updating user privacy and security settings.
In the users/views.py file, create a new view to handle updating user privacy and security settings.

# users/views.py
from django.views.decorators.csrf import csrf_exempt
from rest_framework.decorators import api_view, permission_classes
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response

# ...

@api_view(['PATCH'])
@permission_classes([IsAuthenticated])
@csrf_exempt
def update_privacy_settings(request):
    user = request.user
    data = request.data

    user.private_account = data.get('privateAccount', user.private_account)
    user.two_factor_auth = data.get('twoFactorAuth', user.two_factor_auth)
    user.save()

    return Response({'detail': 'Privacy settings updated successfully'})

6. Add a new URL pattern to handle the update privacy settings API endpoint.
In users/urls.py, add the following path to handle updating user privacy and security settings.

# users/urls.py
from django.urls import path
from . import views

urlpatterns = [
  # ...
  path('update_privacy_settings/', views.update_privacy_settings, name='update_privacy_settings'),
]


Now, users can update their privacy and security settings through the privacy and security settings page in the frontend, and the backend will handle updating the user's privacy and security settings in the database.

