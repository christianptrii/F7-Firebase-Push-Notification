<template>
  <f7-page name="home">
    <!-- Top Navbar -->
    <f7-navbar large :sliding="false">
      <f7-nav-left>
        <f7-link icon-ios="f7:menu" icon-md="material:menu" panel-open="left"></f7-link>
      </f7-nav-left>
      <f7-nav-title-large>Framework7 Auth Example</f7-nav-title-large>
    </f7-navbar>
    
    <!-- Page content -->
    <f7-block v-if="user" strong>
      <f7-block-header>You are logged in as {{user.displayName || user.email}}</f7-block-header>
      <f7-button v-on:click="onLogoutClicked">Logout</f7-button>
    </f7-block>

    <f7-block v-if="user" strong>
      <f7-block-header>Push Notification Status: {{ pushNotificationStatus }}</f7-block-header>
      <f7-button v-if="pushNotificationStatus === 'Not requested'" @click="requestPushNotificationPermission">Request Push Notifications</f7-button>
    </f7-block>

    <f7-block v-if="!user" strong>
      <form @submit.prevent="onLoginWithEmailClicked">
        <f7-list class="login-list" no-hairlines-md>
          <f7-list-input
            class="email-input"
            :value="email"
            @input="email = $event.target.value"
            label="Login with your email address"
            type="email"
            placeholder="Email address"
          />
          <f7-list-input
            :value="password"
            @input="password = $event.target.value"
            label="Provide your password"
            type="password"
            placeholder="Password"
          />
        </f7-list>
        <f7-button fill type="submit">Login</f7-button>
      </form>
    </f7-block>

    <f7-block v-if="!user" strong>
      <f7-block-title>No account yet?</f7-block-title>
      <f7-button v-if="!showSignupForm" fill @click="showSignupForm = true">Create new account</f7-button>
      <form v-if="showSignupForm" @submit.prevent="onSignupClicked">
        <f7-list class="login-list" no-hairlines-md>
          <f7-list-input
            class="email-input"
            :value="email_signup"
            @input="email_signup = $event.target.value"
            label="Type in your email address"
            type="email"
            placeholder="E-mail"
          />
          <f7-list-input
            :value="password_signup"
            @input="password_signup = $event.target.value"
            label="Choose a password"
            type="password"
            placeholder="Password"
          />
        </f7-list>
        <f7-button fill type="submit">Sign up</f7-button>
      </form>
    </f7-block>
  </f7-page>
</template>

<script>
import { initializeApp } from 'firebase/app';
import { getAuth, onAuthStateChanged, createUserWithEmailAndPassword, signInWithEmailAndPassword } from 'firebase/auth';

export default {
  data() {
    return {
      user: null,
      email: '',
      password: '',
      email_signup: '',
      password_signup: '',
      showSignupForm: false,
      pushNotificationStatus: 'Not requested',
    };
  },
  methods: {
    onLoginWithEmailClicked() {
      const auth = getAuth();
      signInWithEmailAndPassword(auth, this.email, this.password)
        .then((userCredential) => {
          const user = userCredential.user;
          console.log('Login successful:', user);
          this.$f7.dialog.alert('Login successful!', 'Success');
          this.user = user;
          this.requestPushNotificationPermission();
        })
        .catch((error) => {
          console.error('Login failed:', error);
          this.$f7.dialog.alert('Login failed: ' + error.message + '. Please try again.', 'Error');
        });
    },

    onSignupClicked() {
      const auth = getAuth();
      createUserWithEmailAndPassword(auth, this.email_signup, this.password_signup)
        .then((userCredential) => {
          console.log('User created successfully', userCredential);
          this.$f7.dialog.alert('User created successfully!', 'Success');
          this.showSignupForm = false;
        })
        .catch((error) => {
          console.error('Failed to create user', error);
          this.$f7.dialog.alert('Failed to create user: ' + error.message + '. Please try again.', 'Error');
        });
    },

    onLogoutClicked() {
      const auth = getAuth();
      auth.signOut()
        .then(() => {
          this.user = null;
          this.pushNotificationStatus = 'Not requested';
          this.$f7.dialog.alert('You have successfully logged out.', 'Success');
        })
        .catch((error) => {
          console.error('Logout failed:', error);
          this.$f7.dialog.alert('Logout failed. Please try again.', 'Error');
        });
    },

    requestPushNotificationPermission() {
      if ('Notification' in window && 'serviceWorker' in navigator) {
        Notification.requestPermission().then((permission) => {
          if (permission === 'granted') {
            this.pushNotificationStatus = 'Granted';
            this.$f7.dialog.alert('Push notification permission granted!', 'Success');
            // Here you would typically send the device token to your server
            console.log('Would send device token to server here');
          } else {
            this.pushNotificationStatus = 'Denied';
            this.$f7.dialog.alert('Push notification permission denied.', 'Notice');
          }
        });
      } else {
        this.pushNotificationStatus = 'Not supported';
        this.$f7.dialog.alert('Push notifications are not supported in this browser.', 'Notice');
      }
    },
  },

  mounted() {
    const firebaseConfig = {
      apiKey: "AIzaSyARGcSAibEQNpdAyhFOryiy_nf_BM4dlMI",
      authDomain: "login-2927d.web.app",
    };

    const app = initializeApp(firebaseConfig);
    const auth = getAuth(app);

    onAuthStateChanged(auth, (user) => {
      if (user) {
        console.log('User is logged in:', user);
        this.user = user;
        // Check if we need to request push notification permission
        if (this.pushNotificationStatus === 'Not requested') {
          this.requestPushNotificationPermission();
        }
      } else {
        console.log('No user is logged in.');
        this.user = null;
        this.pushNotificationStatus = 'Not requested';
      }
    });
  },
};
</script>