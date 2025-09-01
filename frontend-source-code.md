# SmartLibrary Frontend Source Code

## 📁 File Structure
```
client/
├── public/
│   └── index.html
├── src/
│   ├── components/
│   │   ├── Navbar.js
│   │   └── ProtectedRoute.js
│   ├── pages/
│   │   ├── Home.js
│   │   ├── Login.js
│   │   └── Books.js
│   ├── context/
│   │   └── AuthContext.js
│   ├── utils/
│   │   ├── api.js
│   │   └── auth.js
│   ├── styles/
│   │   ├── App.css
│   │   └── index.css
│   ├── App.js
│   └── index.js
└── package.json
```

---

## 📦 package.json
```json
{
  "name": "smartlibrary-client",
  "version": "1.0.0",
  "description": "Frontend client for SmartLibrary Management System",
  "private": true,
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-router-dom": "^6.15.0",
    "axios": "^1.5.0",
    "react-icons": "^4.11.0",
    "react-toastify": "^9.1.3",
    "moment": "^2.29.4",
    "chart.js": "^4.4.0",
    "react-chartjs-2": "^5.2.0",
    "web-vitals": "^3.4.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
  "eslintConfig": {
    "extends": [
      "react-app",
      "react-app/jest"
    ]
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  },
  "devDependencies": {
    "react-scripts": "^5.0.1"
  },
  "proxy": "http://localhost:5000"
}
```

---

## 📄 public/index.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <link rel="icon" href="%PUBLIC_URL%/favicon.ico" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <meta name="theme-color" content="#000000" />
  <meta name="description" content="SmartLibrary - Digital Library Management System" />
  
  <!-- Tailwind CSS CDN -->
  <script src="https://cdn.tailwindcss.com"></script>
  
  <!-- Google Fonts -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
  
  <title>SmartLibrary - Digital Library Management</title>
  
  <style>
    * {
      font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
    }
  </style>
</head>
<body>
  <noscript>You need to enable JavaScript to run this app.</noscript>
  <div id="root"></div>
</body>
</html>
```

---

## ⚛️ src/App.js
```javascript
import React, { useEffect } from 'react';
import { BrowserRouter as Router, Routes, Route, Navigate } from 'react-router-dom';
import { ToastContainer } from 'react-toastify';
import 'react-toastify/dist/ReactToastify.css';

// Components
import Navbar from './components/Navbar';
import ProtectedRoute from './components/ProtectedRoute';

// Pages
import Home from './pages/Home';
import Login from './pages/Login';
import Register from './pages/Register';
import Books from './pages/Books';
import BookDetails from './pages/BookDetails';
import Profile from './pages/Profile';
import Admin from './pages/Admin';
import Dashboard from './pages/Dashboard';
import UserTransactions from './pages/UserTransactions';
import NotFound from './pages/NotFound';

// Utils
import { getCurrentUser } from './utils/auth';
import { useAuthContext } from './context/AuthContext';

// Styles
import './styles/App.css';

function App() {
  const { user, setUser, loading, setLoading } = useAuthContext();

  useEffect(() => {
    // Initialize user from localStorage
    const initUser = async () => {
      try {
        const currentUser = getCurrentUser();
        if (currentUser) {
          setUser(currentUser);
        }
      } catch (error) {
        console.error('Error initializing user:', error);
      } finally {
        setLoading(false);
      }
    };

    initUser();
  }, [setUser, setLoading]);

  if (loading) {
    return (
      <div className="min-h-screen flex items-center justify-center bg-gradient-to-br from-blue-50 to-indigo-100">
        <div className="text-center">
          <div className="animate-spin rounded-full h-16 w-16 border-b-2 border-indigo-600 mx-auto mb-4"></div>
          <h2 className="text-xl font-semibold text-gray-700">Loading SmartLibrary...</h2>
        </div>
      </div>
    );
  }

  return (
    <Router>
      <div className="App min-h-screen bg-gray-50">
        <Navbar />
        
        <main className="main-content">
          <Routes>
            {/* Public Routes */}
            <Route path="/" element={<Home />} />
            <Route path="/books" element={<Books />} />
            <Route path="/books/:id" element={<BookDetails />} />
            
            {/* Auth Routes - redirect if already logged in */}
            <Route 
              path="/login" 
              element={user ? <Navigate to="/dashboard" replace /> : <Login />} 
            />
            <Route 
              path="/register" 
              element={user ? <Navigate to="/dashboard" replace /> : <Register />} 
            />

            {/* Protected Routes */}
            <Route 
              path="/dashboard" 
              element={
                <ProtectedRoute>
                  <Dashboard />
                </ProtectedRoute>
              } 
            />
            <Route 
              path="/profile" 
              element={
                <ProtectedRoute>
                  <Profile />
                </ProtectedRoute>
              } 
            />
            <Route 
              path="/my-books" 
              element={
                <ProtectedRoute>
                  <UserTransactions />
                </ProtectedRoute>
              } 
            />

            {/* Admin Routes */}
            <Route 
              path="/admin/*" 
              element={
                <ProtectedRoute requireAdmin>
                  <Admin />
                </ProtectedRoute>
              } 
            />

            {/* 404 Route */}
            <Route path="*" element={<NotFound />} />
          </Routes>
        </main>

        {/* Toast Notifications */}
        <ToastContainer
          position="top-right"
          autoClose={5000}
          hideProgressBar={false}
          newestOnTop={false}
          closeOnClick
          rtl={false}
          pauseOnFocusLoss
          draggable
          pauseOnHover
          theme="light"
          className="toast-container"
        />
      </div>
    </Router>
  );
}

export default App;
```

---

## 🚀 src/index.js
```javascript
import React from 'react';
import ReactDOM from 'react-dom/client';
import './styles/index.css';
import App from './App';
import { AuthProvider } from './context/AuthContext';
import reportWebVitals from './reportWebVitals';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <AuthProvider>
      <App />
    </AuthProvider>
  </React.StrictMode>
);

// Performance monitoring
reportWebVitals();
```

---

## 🔐 src/context/AuthContext.js
```javascript
import React, { createContext, useContext, useState, useCallback } from 'react';
import { toast } from 'react-toastify';
import * as authUtils from '../utils/auth';

const AuthContext = createContext();

export const useAuthContext = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuthContext must be used within an AuthProvider');
  }
  return context;
};

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  // Login function
  const login = useCallback(async (credentials) => {
    try {
      const response = await authUtils.login(credentials);
      if (response.success) {
        setUser(response.data.user);
        toast.success('Login successful!');
        return { success: true };
      } else {
        toast.error(response.message || 'Login failed');
        return { success: false, message: response.message };
      }
    } catch (error) {
      const message = error.response?.data?.message || 'Login failed';
      toast.error(message);
      return { success: false, message };
    }
  }, []);

  // Register function
  const register = useCallback(async (userData) => {
    try {
      const response = await authUtils.register(userData);
      if (response.success) {
        setUser(response.data.user);
        toast.success('Registration successful!');
        return { success: true };
      } else {
        toast.error(response.message || 'Registration failed');
        return { success: false, message: response.message };
      }
    } catch (error) {
      const message = error.response?.data?.message || 'Registration failed';
      toast.error(message);
      return { success: false, message };
    }
  }, []);

  // Logout function
  const logout = useCallback(() => {
    authUtils.logout();
    setUser(null);
    toast.success('Logged out successfully');
  }, []);

  // Update user profile
  const updateProfile = useCallback(async (profileData) => {
    try {
      const response = await authUtils.updateProfile(profileData);
      if (response.success) {
        setUser(response.data.user);
        toast.success('Profile updated successfully!');
        return { success: true };
      } else {
        toast.error(response.message || 'Update failed');
        return { success: false, message: response.message };
      }
    } catch (error) {
      const message = error.response?.data?.message || 'Update failed';
      toast.error(message);
      return { success: false, message };
    }
  }, []);

  const value = {
    user,
    setUser,
    loading,
    setLoading,
    login,
    register,
    logout,
    updateProfile,
    isAuthenticated: !!user,
    isAdmin: user?.role === 'admin'
  };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
};
```

---

## 🌐 src/utils/api.js
```javascript
import axios from 'axios';
import { getToken, logout } from './auth';

// Create axios instance
const api = axios.create({
  baseURL: process.env.REACT_APP_API_URL || 'http://localhost:5000/api',
  timeout: 30000,
});

// Request interceptor to add auth token
api.interceptors.request.use(
  (config) => {
    const token = getToken();
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

// Response interceptor to handle errors
api.interceptors.response.use(
  (response) => {
    return response.data;
  },
  (error) => {
    // Handle specific error cases
    if (error.response?.status === 401) {
      // Token expired or invalid
      logout();
      window.location.href = '/login';
    }
    
    // Return error in consistent format
    return Promise.reject(error);
  }
);

// API endpoints
export const endpoints = {
  // Authentication
  login: '/auth/login',
  register: '/auth/register',
  profile: '/auth/profile',
  
  // Books
  books: '/books',
  bookDetails: (id) => `/books/${id}`,
  
  // Users
  users: '/users',
  userDetails: (id) => `/users/${id}`,
  
  // Transactions
  transactions: '/transactions',
  userTransactions: '/transactions/user',
  adminTransactions: '/transactions/admin',
  issueBook: '/transactions/issue',
  returnBook: (id) => `/transactions/return/${id}`,
};

// Specific API calls
export const authAPI = {
  login: (credentials) => api.post(endpoints.login, credentials),
  register: (userData) => api.post(endpoints.register, userData),
  getProfile: () => api.get(endpoints.profile),
  updateProfile: (data) => api.put(endpoints.profile, data),
};

export const booksAPI = {
  getBooks: (params = {}) => api.get(endpoints.books, { params }),
  getBook: (id) => api.get(endpoints.bookDetails(id)),
  createBook: (data) => api.post(endpoints.books, data),
  updateBook: (id, data) => api.put(endpoints.bookDetails(id), data),
  deleteBook: (id) => api.delete(endpoints.bookDetails(id)),
};

export const usersAPI = {
  getUsers: (params = {}) => api.get(endpoints.users, { params }),
  getUser: (id) => api.get(endpoints.userDetails(id)),
  updateUser: (id, data) => api.put(endpoints.userDetails(id), data),
  deleteUser: (id) => api.delete(endpoints.userDetails(id)),
};

export const transactionsAPI = {
  getUserTransactions: (params = {}) => api.get(endpoints.userTransactions, { params }),
  getAdminTransactions: (params = {}) => api.get(endpoints.adminTransactions, { params }),
  issueBook: (data) => api.post(endpoints.issueBook, data),
  returnBook: (id) => api.put(endpoints.returnBook(id)),
};

export default api;
```

---

## 🔑 src/utils/auth.js
```javascript
import { authAPI } from './api';

const TOKEN_KEY = 'smartlibrary_token';
const USER_KEY = 'smartlibrary_user';

// Token management
export const getToken = () => {
  return localStorage.getItem(TOKEN_KEY);
};

export const setToken = (token) => {
  localStorage.setItem(TOKEN_KEY, token);
};

export const removeToken = () => {
  localStorage.removeItem(TOKEN_KEY);
};

// User management
export const getCurrentUser = () => {
  try {
    const user = localStorage.getItem(USER_KEY);
    return user ? JSON.parse(user) : null;
  } catch (error) {
    console.error('Error getting current user:', error);
    return null;
  }
};

export const setCurrentUser = (user) => {
  localStorage.setItem(USER_KEY, JSON.stringify(user));
};

export const removeCurrentUser = () => {
  localStorage.removeItem(USER_KEY);
};

// Authentication methods
export const login = async (credentials) => {
  try {
    const response = await authAPI.login(credentials);
    if (response.success && response.data) {
      setToken(response.data.token);
      setCurrentUser(response.data.user);
    }
    return response;
  } catch (error) {
    throw error;
  }
};

export const register = async (userData) => {
  try {
    const response = await authAPI.register(userData);
    if (response.success && response.data) {
      setToken(response.data.token);
      setCurrentUser(response.data.user);
    }
    return response;
  } catch (error) {
    throw error;
  }
};

export const logout = () => {
  removeToken();
  removeCurrentUser();
  localStorage.removeItem('smartlibrary_cache');
};

export const updateProfile = async (profileData) => {
  try {
    const response = await authAPI.updateProfile(profileData);
    if (response.success && response.data) {
      setCurrentUser(response.data.user);
    }
    return response;
  } catch (error) {
    throw error;
  }
};

// Utility functions
export const isAuthenticated = () => {
  const token = getToken();
  const user = getCurrentUser();
  return !!(token && user);
};

export const isAdmin = () => {
  const user = getCurrentUser();
  return user?.role === 'admin';
};
```

---

## 🧭 src/components/Navbar.js
```javascript
import React, { useState } from 'react';
import { Link, useNavigate, useLocation } from 'react-router-dom';
import { FiHome, FiBook, FiUser, FiLogIn, FiLogOut, FiMenu, FiX, FiSettings } from 'react-icons/fi';
import { useAuthContext } from '../context/AuthContext';

const Navbar = () => {
  const { user, logout, isAuthenticated, isAdmin } = useAuthContext();
  const navigate = useNavigate();
  const location = useLocation();
  const [isMobileMenuOpen, setIsMobileMenuOpen] = useState(false);

  const handleLogout = () => {
    logout();
    navigate('/');
    setIsMobileMenuOpen(false);
  };

  const isActiveRoute = (path) => {
    return location.pathname === path;
  };

  const navLinks = [
    { path: '/', label: 'Home', icon: FiHome },
    { path: '/books', label: 'Books', icon: FiBook },
  ];

  const authenticatedLinks = [
    { path: '/dashboard', label: 'Dashboard', icon: FiUser },
    { path: '/my-books', label: 'My Books', icon: FiBook },
    { path: '/profile', label: 'Profile', icon: FiUser },
  ];

  const adminLinks = [
    { path: '/admin', label: 'Admin', icon: FiSettings },
  ];

  return (
    <nav className="bg-white shadow-lg border-b">
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div className="flex justify-between h-16">
          {/* Logo */}
          <div className="flex items-center">
            <Link
              to="/"
              className="flex items-center space-x-2 text-2xl font-bold text-indigo-600 hover:text-indigo-800"
            >
              <FiBook className="text-3xl" />
              <span>SmartLibrary</span>
            </Link>
          </div>

          {/* Desktop Navigation */}
          <div className="hidden md:flex items-center space-x-8">
            {/* Public Links */}
            {navLinks.map((link) => (
              <Link
                key={link.path}
                to={link.path}
                className={`flex items-center space-x-1 px-3 py-2 rounded-md text-sm font-medium transition-colors duration-200 ${
                  isActiveRoute(link.path)
                    ? 'text-indigo-600 bg-indigo-50'
                    : 'text-gray-700 hover:text-indigo-600 hover:bg-gray-50'
                }`}
              >
                <link.icon />
                <span>{link.label}</span>
              </Link>
            ))}

            {/* Authenticated Links */}
            {isAuthenticated && (
              <>
                {authenticatedLinks.map((link) => (
                  <Link
                    key={link.path}
                    to={link.path}
                    className={`flex items-center space-x-1 px-3 py-2 rounded-md text-sm font-medium transition-colors duration-200 ${
                      isActiveRoute(link.path)
                        ? 'text-indigo-600 bg-indigo-50'
                        : 'text-gray-700 hover:text-indigo-600 hover:bg-gray-50'
                    }`}
                  >
                    <link.icon />
                    <span>{link.label}</span>
                  </Link>
                ))}

                {/* Admin Links */}
                {isAdmin &&
                  adminLinks.map((link) => (
                    <Link
                      key={link.path}
                      to={link.path}
                      className={`flex items-center space-x-1 px-3 py-2 rounded-md text-sm font-medium transition-colors duration-200 ${
                        location.pathname.startsWith(link.path)
                          ? 'text-indigo-600 bg-indigo-50'
                          : 'text-gray-700 hover:text-indigo-600 hover:bg-gray-50'
                      }`}
                    >
                      <link.icon />
                      <span>{link.label}</span>
                    </Link>
                  ))}
              </>
            )}

            {/* Auth Buttons */}
            {isAuthenticated ? (
              <div className="flex items-center space-x-4">
                <span className="text-sm text-gray-600">
                  Welcome, {user?.name}
                  {user?.role === 'admin' && (
                    <span className="ml-1 px-2 py-1 text-xs bg-indigo-100 text-indigo-800 rounded-full">
                      Admin
                    </span>
                  )}
                </span>
                <button
                  onClick={handleLogout}
                  className="flex items-center space-x-1 px-4 py-2 bg-red-600 text-white rounded-md hover:bg-red-700 transition-colors duration-200"
                >
                  <FiLogOut />
                  <span>Logout</span>
                </button>
              </div>
            ) : (
              <div className="flex items-center space-x-4">
                <Link
                  to="/login"
                  className="flex items-center space-x-1 px-4 py-2 text-indigo-600 border border-indigo-600 rounded-md hover:bg-indigo-50 transition-colors duration-200"
                >
                  <FiLogIn />
                  <span>Login</span>
                </Link>
                <Link
                  to="/register"
                  className="flex items-center space-x-1 px-4 py-2 bg-indigo-600 text-white rounded-md hover:bg-indigo-700 transition-colors duration-200"
                >
                  <FiUser />
                  <span>Register</span>
                </Link>
              </div>
            )}
          </div>

          {/* Mobile menu button */}
          <div className="md:hidden flex items-center">
            <button
              onClick={() => setIsMobileMenuOpen(!isMobileMenuOpen)}
              className="text-gray-700 hover:text-indigo-600 focus:outline-none focus:text-indigo-600"
            >
              {isMobileMenuOpen ? <FiX size={24} /> : <FiMenu size={24} />}
            </button>
          </div>
        </div>
      </div>

      {/* Mobile Navigation */}
      {isMobileMenuOpen && (
        <div className="md:hidden bg-white border-t border-gray-200">
          <div className="px-2 pt-2 pb-3 space-y-1 sm:px-3">
            {/* Public Links */}
            {navLinks.map((link) => (
              <Link
                key={link.path}
                to={link.path}
                onClick={() => setIsMobileMenuOpen(false)}
                className={`flex items-center space-x-2 px-3 py-2 rounded-md text-base font-medium ${
                  isActiveRoute(link.path)
                    ? 'text-indigo-600 bg-indigo-50'
                    : 'text-gray-700 hover:text-indigo-600 hover:bg-gray-50'
                }`}
              >
                <link.icon />
                <span>{link.label}</span>
              </Link>
            ))}

            {/* Auth Section */}
            <div className="border-t border-gray-200 pt-4">
              {isAuthenticated ? (
                <>
                  <div className="px-3 py-2">
                    <span className="text-sm text-gray-600">
                      Welcome, {user?.name}
                    </span>
                  </div>
                  <button
                    onClick={handleLogout}
                    className="flex items-center space-x-2 w-full px-3 py-2 text-left text-red-600 hover:bg-red-50 rounded-md"
                  >
                    <FiLogOut />
                    <span>Logout</span>
                  </button>
                </>
              ) : (
                <>
                  <Link
                    to="/login"
                    onClick={() => setIsMobileMenuOpen(false)}
                    className="flex items-center space-x-2 px-3 py-2 text-indigo-600 hover:bg-indigo-50 rounded-md"
                  >
                    <FiLogIn />
                    <span>Login</span>
                  </Link>
                  <Link
                    to="/register"
                    onClick={() => setIsMobileMenuOpen(false)}
                    className="flex items-center space-x-2 px-3 py-2 text-indigo-600 hover:bg-indigo-50 rounded-md"
                  >
                    <FiUser />
                    <span>Register</span>
                  </Link>
                </>
              )}
            </div>
          </div>
        </div>
      )}
    </nav>
  );
};

export default Navbar;
```

---

## 🛡️ src/components/ProtectedRoute.js
```javascript
import React from 'react';
import { Navigate, useLocation } from 'react-router-dom';
import { useAuthContext } from '../context/AuthContext';

const ProtectedRoute = ({ children, requireAdmin = false }) => {
  const { isAuthenticated, isAdmin, loading } = useAuthContext();
  const location = useLocation();

  // Show loading spinner while checking authentication
  if (loading) {
    return (
      <div className="min-h-screen flex items-center justify-center bg-gray-50">
        <div className="text-center">
          <div className="animate-spin rounded-full h-16 w-16 border-b-2 border-indigo-600 mx-auto mb-4"></div>
          <h2 className="text-xl font-semibold text-gray-700">Checking authentication...</h2>
        </div>
      </div>
    );
  }

  // Redirect to login if not authenticated
  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  // Redirect to dashboard if admin access required but user is not admin
  if (requireAdmin && !isAdmin) {
    return (
      <div className="min-h-screen flex items-center justify-center bg-gray-50">
        <div className="text-center p-8">
          <div className="text-6xl text-red-500 mb-4">🚫</div>
          <h2 className="text-2xl font-bold text-gray-900 mb-2">Access Denied</h2>
          <p className="text-gray-600 mb-4">You don't have permission to access this page.</p>
          <Navigate to="/dashboard" replace />
        </div>
      </div>
    );
  }

  // Render children if all checks pass
  return children;
};

export default ProtectedRoute;
```

---

## 🏠 src/pages/Home.js
```javascript
import React, { useState, useEffect } from 'react';
import { Link } from 'react-router-dom';
import { FiBook, FiUsers, FiTrendingUp, FiSearch, FiArrowRight } from 'react-icons/fi';
import { booksAPI } from '../utils/api';

const Home = () => {
  const [stats, setStats] = useState({
    totalBooks: 0,
    availableBooks: 0,
    totalUsers: 0,
  });
  const [featuredBooks, setFeaturedBooks] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const fetchHomeData = async () => {
      try {
        // Fetch featured books
        const booksResponse = await booksAPI.getBooks({ 
          limit: 6, 
          sortBy: 'rating' 
        });
        
        if (booksResponse.success) {
          setFeaturedBooks(booksResponse.data.books);
          setStats({
            totalBooks: booksResponse.data.pagination.totalBooks,
            availableBooks: booksResponse.data.books.filter(book => book.isAvailable).length,
            totalUsers: 1250, // Mock data
          });
        }
      } catch (error) {
        console.error('Error fetching home data:', error);
      } finally {
        setLoading(false);
      }
    };

    fetchHomeData();
  }, []);

  const features = [
    {
      icon: FiBook,
      title: 'Extensive Collection',
      description: 'Browse through thousands of books across multiple genres and categories.',
    },
    {
      icon: FiSearch,
      title: 'Smart Search',
      description: 'Find books quickly with our advanced search and filtering system.',
    },
    {
      icon: FiUsers,
      title: 'Community Reviews',
      description: 'Read reviews and ratings from fellow readers to discover great books.',
    },
    {
      icon: FiTrendingUp,
      title: 'Reading Analytics',
      description: 'Track your reading progress and discover new reading patterns.',
    },
  ];

  if (loading) {
    return (
      <div className="min-h-screen flex items-center justify-center bg-gray-50">
        <div className="text-center">
          <div className="animate-spin rounded-full h-16 w-16 border-b-2 border-indigo-600 mx-auto mb-4"></div>
          <h2 className="text-xl font-semibold text-gray-700">Loading...</h2>
        </div>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-gray-50">
      {/* Hero Section */}
      <section className="bg-gradient-to-r from-indigo-600 to-purple-700 text-white">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-24">
          <div className="text-center">
            <h1 className="text-4xl md:text-6xl font-bold mb-6">
              Welcome to <span className="text-yellow-300">SmartLibrary</span>
            </h1>
            <p className="text-xl md:text-2xl mb-8 opacity-90">
              Your digital gateway to knowledge and discovery
            </p>
            <div className="flex flex-col sm:flex-row justify-center gap-4">
              <Link
                to="/books"
                className="inline-flex items-center px-8 py-3 bg-white text-indigo-600 rounded-lg font-semibold hover:bg-gray-100 transition-colors duration-200"
              >
                <FiBook className="mr-2" />
                Browse Books
              </Link>
              <Link
                to="/register"
                className="inline-flex items-center px-8 py-3 bg-transparent border-2 border-white text-white rounded-lg font-semibold hover:bg-white hover:text-indigo-600 transition-colors duration-200"
              >
                Get Started
                <FiArrowRight className="ml-2" />
              </Link>
            </div>
          </div>
        </div>
      </section>

      {/* Stats Section */}
      <section className="py-16 bg-white">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
          <div className="grid grid-cols-1 md:grid-cols-3 gap-8 text-center">
            <div className="p-6">
              <div className="text-4xl font-bold text-indigo-600 mb-2">
                {stats.totalBooks.toLocaleString()}
              </div>
              <div className="text-gray-600">Books Available</div>
            </div>
            <div className="p-6">
              <div className="text-4xl font-bold text-green-600 mb-2">
                {stats.availableBooks}
              </div>
              <div className="text-gray-600">Currently Available</div>
            </div>
            <div className="p-6">
              <div className="text-4xl font-bold text-purple-600 mb-2">
                {stats.totalUsers.toLocaleString()}
              </div>
              <div className="text-gray-600">Active Members</div>
            </div>
          </div>
        </div>
      </section>

      {/* Features Section */}
      <section className="py-16 bg-gray-50">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
          <div className="text-center mb-12">
            <h2 className="text-3xl md:text-4xl font-bold text-gray-900 mb-4">
              Why Choose SmartLibrary?
            </h2>
            <p className="text-xl text-gray-600">
              Experience the future of library management
            </p>
          </div>

          <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-8">
            {features.map((feature, index) => (
              <div
                key={index}
                className="bg-white p-6 rounded-lg shadow-md hover:shadow-lg transition-shadow duration-200"
              >
                <feature.icon className="text-4xl text-indigo-600 mb-4" />
                <h3 className="text-xl font-semibold mb-2">{feature.title}</h3>
                <p className="text-gray-600">{feature.description}</p>
              </div>
            ))}
          </div>
        </div>
      </section>

      {/* CTA Section */}
      <section className="py-16 bg-indigo-600 text-white">
        <div className="max-w-4xl mx-auto text-center px-4 sm:px-6 lg:px-8">
          <h2 className="text-3xl md:text-4xl font-bold mb-4">
            Ready to Start Your Reading Journey?
          </h2>
          <p className="text-xl mb-8 opacity-90">
            Join thousands of readers and discover your next favorite book today.
          </p>
          <Link
            to="/register"
            className="inline-flex items-center px-8 py-3 bg-white text-indigo-600 rounded-lg font-semibold text-lg hover:bg-gray-100 transition-colors duration-200"
          >
            Get Started Now
            <FiArrowRight className="ml-2" />
          </Link>
        </div>
      </section>
    </div>
  );
};

export default Home;
```

---

## 🔐 src/pages/Login.js
```javascript
import React, { useState } from 'react';
import { Link, useNavigate, useLocation } from 'react-router-dom';
import { FiMail, FiLock, FiEye, FiEyeOff, FiLogIn } from 'react-icons/fi';
import { useAuthContext } from '../context/AuthContext';

const Login = () => {
  const [formData, setFormData] = useState({
    email: '',
    password: '',
  });
  const [showPassword, setShowPassword] = useState(false);
  const [loading, setLoading] = useState(false);
  const [errors, setErrors] = useState({});

  const { login } = useAuthContext();
  const navigate = useNavigate();
  const location = useLocation();
  const from = location.state?.from?.pathname || '/dashboard';

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));
    
    // Clear error when user starts typing
    if (errors[name]) {
      setErrors(prev => ({
        ...prev,
        [name]: ''
      }));
    }
  };

  const validateForm = () => {
    const newErrors = {};

    if (!formData.email) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = 'Please enter a valid email';
    }

    if (!formData.password) {
      newErrors.password = 'Password is required';
    }

    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    
    if (!validateForm()) return;

    setLoading(true);
    
    try {
      const result = await login(formData);
      if (result.success) {
        navigate(from, { replace: true });
      }
    } catch (error) {
      console.error('Login error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-indigo-50 to-blue-100 flex items-center justify-center py-12 px-4 sm:px-6 lg:px-8">
      <div className="max-w-md w-full space-y-8">
        <div>
          <div className="text-center">
            <FiLogIn className="mx-auto h-12 w-12 text-indigo-600" />
            <h2 className="mt-6 text-3xl font-extrabold text-gray-900">
              Sign in to your account
            </h2>
            <p className="mt-2 text-sm text-gray-600">
              Or{' '}
              <Link
                to="/register"
                className="font-medium text-indigo-600 hover:text-indigo-500"
              >
                create a new account
              </Link>
            </p>
          </div>
        </div>

        <div className="bg-white py-8 px-6 shadow rounded-lg">
          <form className="space-y-6" onSubmit={handleSubmit}>
            {/* Email Field */}
            <div>
              <label htmlFor="email" className="block text-sm font-medium text-gray-700">
                Email address
              </label>
              <div className="mt-1 relative">
                <div className="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
                  <FiMail className="h-5 w-5 text-gray-400" />
                </div>
                <input
                  id="email"
                  name="email"
                  type="email"
                  autoComplete="email"
                  required
                  value={formData.email}
                  onChange={handleChange}
                  className={`block w-full pl-10 pr-3 py-2 border rounded-md shadow-sm placeholder-gray-400 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm ${
                    errors.email ? 'border-red-300' : 'border-gray-300'
                  }`}
                  placeholder="Enter your email"
                />
              </div>
              {errors.email && (
                <p className="mt-1 text-sm text-red-600">{errors.email}</p>
              )}
            </div>

            {/* Password Field */}
            <div>
              <label htmlFor="password" className="block text-sm font-medium text-gray-700">
                Password
              </label>
              <div className="mt-1 relative">
                <div className="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
                  <FiLock className="h-5 w-5 text-gray-400" />
                </div>
                <input
                  id="password"
                  name="password"
                  type={showPassword ? 'text' : 'password'}
                  autoComplete="current-password"
                  required
                  value={formData.password}
                  onChange={handleChange}
                  className={`block w-full pl-10 pr-10 py-2 border rounded-md shadow-sm placeholder-gray-400 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm ${
                    errors.password ? 'border-red-300' : 'border-gray-300'
                  }`}
                  placeholder="Enter your password"
                />
                <button
                  type="button"
                  className="absolute inset-y-0 right-0 pr-3 flex items-center"
                  onClick={() => setShowPassword(!showPassword)}
                >
                  {showPassword ? (
                    <FiEyeOff className="h-5 w-5 text-gray-400 hover:text-gray-500" />
                  ) : (
                    <FiEye className="h-5 w-5 text-gray-400 hover:text-gray-500" />
                  )}
                </button>
              </div>
              {errors.password && (
                <p className="mt-1 text-sm text-red-600">{errors.password}</p>
              )}
            </div>

            {/* Demo Account Info */}
            <div className="bg-blue-50 border border-blue-200 rounded-md p-4">
              <h3 className="text-sm font-medium text-blue-800 mb-2">Demo Accounts</h3>
              <div className="text-sm text-blue-700 space-y-1">
                <div><strong>Admin:</strong> admin@smartlibrary.com / admin123</div>
                <div><strong>User:</strong> user@smartlibrary.com / user123</div>
              </div>
            </div>

            {/* Submit Button */}
            <div>
              <button
                type="submit"
                disabled={loading}
                className="group relative w-full flex justify-center py-2 px-4 border border-transparent text-sm font-medium rounded-md text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 disabled:opacity-50 disabled:cursor-not-allowed"
              >
                {loading ? (
                  <div className="flex items-center">
                    <div className="animate-spin rounded-full h-4 w-4 border-b-2 border-white mr-2"></div>
                    Signing in...
                  </div>
                ) : (
                  <>
                    <FiLogIn className="mr-2" />
                    Sign in
                  </>
                )}
              </button>
            </div>
          </form>
        </div>
      </div>
    </div>
  );
};

export default Login;
```

---

## 🎨 src/styles/App.css
```css
/* SmartLibrary App Styles */

/* Global Styles */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  line-height: 1.6;
  color: #374151;
}

/* App Layout */
.App {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.main-content {
  flex: 1;
  padding-top: 0;
}

/* Utility Classes */
.line-clamp-2 {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.line-clamp-3 {
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

/* Button Styles */
.btn-primary {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border: none;
  color: white;
  padding: 12px 24px;
  border-radius: 8px;
  font-weight: 600;
  transition: all 0.3s ease;
  cursor: pointer;
}

.btn-primary:hover {
  transform: translateY(-1px);
  box-shadow: 0 5px 15px rgba(102, 126, 234, 0.3);
}

/* Loading States */
.loading-spinner {
  display: inline-block;
  width: 20px;
  height: 20px;
  border: 2px solid #f3f3f3;
  border-top: 2px solid #6366f1;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

/* Card Hover Effects */
.card-hover {
  transition: all 0.3s ease;
}

.card-hover:hover {
  transform: translateY(-2px);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
}

/* Toast Notifications */
.toast-container {
  font-family: inherit;
}

/* Animation Classes */
.fade-in {
  animation: fadeIn 0.6s ease-in-out;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* Mobile Responsive */
@media (max-width: 768px) {
  .main-content {
    padding: 1rem;
  }
}
```

---

## 🎯 src/styles/index.css
```css
/* SmartLibrary Index Styles - Tailwind CSS Base */

@import 'tailwindcss/base';
@import 'tailwindcss/components'; 
@import 'tailwindcss/utilities';

/* Custom CSS Variables */
:root {
  --primary-50: #eff6ff;
  --primary-100: #dbeafe;
  --primary-500: #3b82f6;
  --primary-600: #2563eb;
  --primary-700: #1d4ed8;
}

/* Base Styles */
html {
  scroll-behavior: smooth;
}

body {
  margin: 0;
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  background-color: #f9fafb;
}

/* Custom Utilities */
.text-gradient {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

/* Component Overrides */
.btn {
  @apply px-4 py-2 rounded-lg font-medium transition-all duration-200 focus:outline-none focus:ring-2 focus:ring-offset-2;
}

.btn-primary {
  @apply bg-indigo-600 text-white hover:bg-indigo-700 focus:ring-indigo-500;
}

.card {
  @apply bg-white rounded-lg shadow-md border border-gray-200;
}

.input {
  @apply w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm placeholder-gray-400 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500;
}

/* Responsive Design */
@media (max-width: 640px) {
  .container {
    @apply px-4;
  }
}
```

---

## 🚀 Getting Started

### 1. Install Dependencies
```bash
cd client
npm install
```

### 2. Environment Setup
Create `.env` file in client root:
```env
REACT_APP_API_URL=http://localhost:5000/api
```

### 3. Start Development Server
```bash
npm start
```

The application will be running on http://localhost:3000

---

## 📱 Features Summary

### Core Components
- **Navbar**: Responsive navigation with mobile menu
- **ProtectedRoute**: Route protection with role-based access
- **AuthContext**: Global authentication state management

### Pages
- **Home**: Landing page with stats and featured books
- **Login**: Authentication with demo accounts
- **Books**: Book catalog with search and filtering
- **Dashboard**: User/Admin dashboard (additional implementation needed)
- **Profile**: User profile management (additional implementation needed)

### Utilities
- **API**: Axios configuration with interceptors
- **Auth**: Token and user management utilities
- **Responsive**: Mobile-first design approach

---

This frontend provides a modern, responsive user interface for the SmartLibrary system with comprehensive authentication, routing, and state management capabilities.