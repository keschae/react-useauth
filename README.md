# react-useauth



Of course! This is a very common and robust pattern for handling authentication in modern React applications. It leverages the strengths of `useContext` for state management and `react-router-dom` for routing and navigation.

Here is a complete, step-by-step guide to a good authentication pattern, including how to handle non-existent routes (404 pages).

### Core Concepts of the Pattern

1.  **`AuthContext`**: A React Context to hold the global authentication state (e.g., the current user, loading status). This makes the user's authentication status available to any component in the app without prop drilling.
2.  **`AuthProvider`**: A component that wraps the entire application. It manages the authentication state (using `useState`) and provides the `login` and `logout` functions. It will also handle initial checks (like seeing if a user token is in `localStorage`).
3.  **`useAuth` Custom Hook**: A custom hook (`useAuth`) that provides a simple and clean way for components to consume the `AuthContext`.
4.  **`ProtectedRoute` Component**: A special "wrapper" component that checks if a user is authenticated. If they are, it renders the requested page (its `children`). If not, it uses `useNavigate` to redirect them to the login page.
5.  **`NotFoundPage` Component**: A simple component to be displayed when the user tries to access a URL that doesn't match any of our defined routes.

---

### Step 1: Project Structure

A good file structure makes this pattern easy to manage.

```
src/
├── components/
│   ├── ProtectedRoute.js
│   └── Layout.js          (Optional, for nav, footer, etc.)
├── contexts/
│   └── AuthContext.js
├── pages/
│   ├── DashboardPage.js
│   ├── HomePage.js
│   ├── LoginPage.js
│   └── NotFoundPage.js
├── App.js
└── index.js
```

---

### Step 2: Create the Authentication Context (`AuthContext.js`)

This is the heart of our authentication system. It manages the state and provides the logic.

```javascript
// src/contexts/AuthContext.js
import React, { createContext, useState, useContext, useEffect } from 'react';

// 1. Create the context
const AuthContext = createContext(null);

// 2. Create the provider component
export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  // On initial load, check if we have a user in localStorage
  useEffect(() => {
    const storedUser = localStorage.getItem('user');
    if (storedUser) {
      setUser(JSON.parse(storedUser));
    }
    setLoading(false); // Finished checking for a stored user
  }, []);

  // Login function
  const login = (userData) => {
    // In a real app, you'd make an API call here
    // For this example, we'll just simulate a successful login
    setUser(userData);
    localStorage.setItem('user', JSON.stringify(userData));
  };

  // Logout function
  const logout = () => {
    setUser(null);
    localStorage.removeItem('user');
  };

  const value = {
    user,
    login,
    logout,
    loading
  };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
};

// 3. Create a custom hook for easy access to the context
export const useAuth = () => {
  return useContext(AuthContext);
};
```

---

### Step 3: Create the `ProtectedRoute` Component

This component will guard our private pages.

```javascript
// src/components/ProtectedRoute.js
import React, { useEffect } from 'react';
import { useNavigate } from 'react-router-dom';
import { useAuth } from '../contexts/AuthContext';

const ProtectedRoute = ({ children }) => {
  const { user, loading } = useAuth();
  const navigate = useNavigate();

  useEffect(() => {
    if (loading) {
      // Still checking for auth status, do nothing yet
      return;
    }

    if (!user) {
      // If not authenticated, redirect to the login page
      navigate('/login');
    }
  }, [user, loading, navigate]);

  if (loading) {
    // You can render a loading spinner here
    return <div>Loading...</div>;
  }

  // If user is authenticated, render the children
  return user ? children : null; // Return null while redirecting
};

export default ProtectedRoute;
```

---

### Step 4: Create the `NotFoundPage` Component

This is for handling non-existent routes.

```javascript
// src/pages/NotFoundPage.js
import React from 'react';
import { Link } from 'react-router-dom';

const NotFoundPage = () => {
  return (
    <div style={{ textAlign: 'center', marginTop: '50px' }}>
      <h1>404 - Page Not Found</h1>
      <p>Sorry, the page you are looking for does not exist.</p>
      <Link to="/">Go to Homepage</Link>
    </div>
  );
};

export default NotFoundPage;
```

---

### Step 5: Create Example Pages

Let's create a `LoginPage` and a protected `DashboardPage`.

```javascript
// src/pages/LoginPage.js
import React, { useState } from 'react';
import { useNavigate, Link } from 'react-router-dom';
import { useAuth } from '../contexts/AuthContext';

const LoginPage = () => {
  const [username, setUsername] = useState('');
  const { login } = useAuth();
  const navigate = useNavigate();

  const handleLogin = (e) => {
    e.preventDefault();
    // Simulate a successful login
    const fakeUserData = { id: 1, name: username };
    login(fakeUserData);
    navigate('/dashboard'); // Redirect to dashboard after login
  };

  return (
    <div>
      <h2>Login</h2>
      <form onSubmit={handleLogin}>
        <input
          type="text"
          placeholder="Username"
          value={username}
          onChange={(e) => setUsername(e.target.value)}
          required
        />
        <button type="submit">Log In</button>
      </form>
      <p>
        Don't have an account? <Link to="/signup">Sign Up</Link>
      </p>
    </div>
  );
};

export default LoginPage;
```

```javascript
// src/pages/DashboardPage.js
import React from 'react';
import { useAuth } from '../contexts/AuthContext';
import { useNavigate } from 'react-router-dom';

const DashboardPage = () => {
  const { user, logout } = useAuth();
  const navigate = useNavigate();

  const handleLogout = () => {
    logout();
    navigate('/login');
  };

  return (
    <div>
      <h2>Dashboard</h2>
      {user ? (
        <p>Welcome, {user.name}!</p>
      ) : (
        <p>Loading user data...</p>
      )}
      <button onClick={handleLogout}>Log Out</button>
    </div>
  );
};

export default DashboardPage;
```

---

### Step 6: Tie It All Together in `App.js`

This is where we define our routes and wrap the app with our providers.

```javascript
// src/App.js
import React from 'react';
import { BrowserRouter as Router, Routes, Route, Link } from 'react-router-dom';
import { AuthProvider, useAuth } from './contexts/AuthContext';
import ProtectedRoute from './components/ProtectedRoute';
import HomePage from './pages/HomePage';
import LoginPage from './pages/LoginPage';
import DashboardPage from './pages/DashboardPage';
import NotFoundPage from './pages/NotFoundPage';

// A simple navigation component
const Navigation = () => {
  const { user } = useAuth();
  return (
    <nav>
      <Link to="/">Home</Link> | 
      {user ? (
        <>
          <Link to="/dashboard">Dashboard</Link> | 
          <span> (Logged in as {user.name})</span>
        </>
      ) : (
        <Link to="/login">Login</Link>
      )}
    </nav>
  );
};


function App() {
  return (
    <AuthProvider>
      <Router>
        <Navigation />
        <hr />
        <Routes>
          {/* Public Routes */}
          <Route path="/" element={<HomePage />} />
          <Route path="/login" element={<LoginPage />} />
          
          {/* Protected Routes */}
          <Route 
            path="/dashboard" 
            element={
              <ProtectedRoute>
                <DashboardPage />
              </ProtectedRoute>
            } 
          />

          {/* 404 Not Found Route - Must be the last route */}
          <Route path="*" element={<NotFoundPage />} />
        </Routes>
      </Router>
    </AuthProvider>
  );
}

export default App;
```

### Explanation of Key Parts

1.  **`<AuthProvider>`**: Wraps the entire `<Router>`. This ensures that the `user` state and `login`/`logout` functions are available to all components inside, including `Navigation`, `ProtectedRoute`, and all page components.

2.  **`<Routes>` and `<Route>`**: This is the standard `react-router-dom` v6 setup for defining your URL structure.

3.  **`<ProtectedRoute>`**: For any route that should be private (like `/dashboard`), we wrap its `element` in `<ProtectedRoute>`. This is the core of the protection logic.

4.  **`<Route path="*" element={<NotFoundPage />} />`**: This is the crucial part for handling non-existent routes.
    *   The `path="*"` is a wildcard that matches any URL that has not already been matched by a previous route.
    *   **It must be the last `<Route>` in your `<Routes>` list.** If you place it earlier, it will catch all routes and prevent any others from ever rendering.

### Why This Pattern is Good

*   **Separation of Concerns**: Authentication logic is in `AuthContext`, routing logic is in `App.js`, and UI logic is in the page components. Everything has its place.
*   **Centralized State**: No prop drilling. Any component can get auth status with a simple `const { user } = useAuth();`.
*   **Reusable**: The `ProtectedRoute` component can be used to wrap any number of protected routes.
*   **Declarative**: Your routing setup in `App.js` clearly shows which routes are public and which are protected.
*   **Scalable**: This pattern easily extends to more complex scenarios like role-based access control (e.g., checking `user.role === 'admin'` inside `ProtectedRoute`).
