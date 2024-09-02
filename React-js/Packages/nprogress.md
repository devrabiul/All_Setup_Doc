### nprogress

For a top-of-the-page line loader (similar to the loading indicators seen on YouTube or Medium), you can use a package like nprogress. It is a small, customizable progress bar library that works well with React.

How to Use nprogress in a React Application
#### Step 1: Install nprogress
First, you need to install nprogress via npm:

```bash
npm install nprogress --save
```

#### Step 2: Import nprogress in Your React Component
You can set up nprogress to show a loading bar at the top of your page. Here's a simple example of how to integrate it in a React component:

```bash
import React, { useEffect } from 'react';
import NProgress from 'nprogress';
import 'nprogress/nprogress.css'; // Import the CSS to style the progress bar

const App = () => {
  useEffect(() => {
    // Start the progress bar
    NProgress.start();

    // Simulate a delay for demonstration
    const timer = setTimeout(() => {
      NProgress.done(); // Stop the progress bar
    }, 2000);

    // Clean up the timer on component unmount
    return () => clearTimeout(timer);
  }, []);

  return (
    <div>
      <h1>My React App with a Top Line Loader</h1>
      <p>Content of the app goes here...</p>
    </div>
  );
};

export default App;
```

#### Step 3: Customize nprogress (Optional)
nprogress provides several options for customization, such as speed and color. You can set these options globally in your app:

```bash
NProgress.configure({
  minimum: 0.1, // The minimum percentage used upon starting
  easing: 'ease', // CSS easing to use when animating the progress bar
  speed: 500, // Speed of the progress bar in milliseconds
  showSpinner: false, // Hide the spinner
  trickleSpeed: 200, // How often to trickle in milliseconds
  parent: 'body', // The parent to append the progress bar to
});
```
To apply these settings, you should add the configure call before NProgress.start().

#### Step 4: Using the Loader Based on Route Change (Optional)
If you want the line loader to show when navigating between routes in a React Router application, you can add a listener to route changes:

```bash
import React, { useEffect } from 'react';
import { BrowserRouter as Router, Route, Switch, useLocation } from 'react-router-dom';
import NProgress from 'nprogress';
import 'nprogress/nprogress.css';

const RouteChangeTracker = () => {
  const location = useLocation();

  useEffect(() => {
    NProgress.start();
    return () => {
      NProgress.done();
    };
  }, [location]);

  return null;
};

const App = () => {
  return (
    <Router>
      <RouteChangeTracker />
      <Switch>
        <Route path="/" exact>
          <h1>Home Page</h1>
        </Route>
        <Route path="/about">
          <h1>About Page</h1>
        </Route>
      </Switch>
    </Router>
  );
};

export default App;
```

Conclusion
Using nprogress is a simple and effective way to add a line loader to the top of your React website. The setup is minimal, and it provides a smooth user experience by indicating loading states clearly.