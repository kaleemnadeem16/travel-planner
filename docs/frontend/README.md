# Frontend Development Guide (React)

## Technology Stack

- **Framework**: React (Create React App, Vite, or Next.js in SPA mode)
- **State Management**: React Context or Redux
- **HTTP Client**: Axios or Fetch API
- **Routing**: React Router
- **UI Framework**: Bootstrap, Material UI, or Tailwind CSS
- **PWA**: Optional Progressive Web App configuration

## Application Structure

### Core Pages/Components

```
src/
├── components/
│   ├── common/
│   │   ├── Header.jsx
│   │   ├── Footer.jsx
│   │   └── LoadingSpinner.jsx
│   ├── auth/
│   │   ├── LoginForm.jsx
│   │   ├── SignupForm.jsx
│   │   └── SocialLoginButton.jsx
│   └── plans/
│       ├── PlanList.jsx
│       ├── PlanCard.jsx
│       ├── PlanForm.jsx
│       └── PlanDetails.jsx
├── pages/
│   ├── HomePage.jsx
│   ├── LoginPage.jsx
│   ├── SignupPage.jsx
│   ├── DashboardPage.jsx
│   └── PlanEditorPage.jsx
├── services/
│   ├── api.js
│   ├── auth.js
│   └── plans.js
├── context/
│   ├── AuthContext.js
│   └── PlansContext.js
└── utils/
    ├── constants.js
    ├── helpers.js
    └── validators.js
```

### Key Pages

1. **Home/Landing Page**
   - App explanation and value proposition
   - Login/Signup call-to-action buttons
   - Features overview

2. **Login/Signup Screens**
   - Email/password forms with validation
   - "Login with Google" OAuth button
   - Password strength indicators
   - Error handling and user feedback

3. **Dashboard**
   - List of user's saved travel plans
   - Quick actions (create new plan, edit existing)
   - User profile information

4. **Plan Create/Edit Screen**
   - Form for plan details (destination, dates, preferences)
   - Rich text editor or structured inputs
   - Save/update functionality

## State Management

### React Context Approach

```javascript
// AuthContext.js
const AuthContext = createContext();

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [token, setToken] = useState(localStorage.getItem('token'));
  const [loading, setLoading] = useState(false);

  const login = async (credentials) => {
    // API call implementation
  };

  const logout = () => {
    // Clear token and user data
  };

  return (
    <AuthContext.Provider value={{ user, token, login, logout, loading }}>
      {children}
    </AuthContext.Provider>
  );
};
```

### Redux Alternative
- Actions: `LOGIN_REQUEST`, `LOGIN_SUCCESS`, `LOGIN_FAILURE`
- Reducers: `authReducer`, `plansReducer`
- Middleware: Redux Thunk for async operations

## API Communication

### Axios Configuration

```javascript
// api.js
import axios from 'axios';

const API_BASE_URL = process.env.REACT_APP_API_URL || 'http://localhost:8000/api';

const apiClient = axios.create({
  baseURL: API_BASE_URL,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor to add auth token
apiClient.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Response interceptor for error handling
apiClient.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      // Handle unauthorized access
      localStorage.removeItem('token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

export default apiClient;
```

### API Service Examples

```javascript
// auth.js
export const authService = {
  login: (credentials) => apiClient.post('/login/', credentials),
  signup: (userData) => apiClient.post('/signup/', userData),
  getProfile: () => apiClient.get('/user/'),
  refreshToken: (refreshToken) => apiClient.post('/token/refresh/', { refresh: refreshToken }),
};

// plans.js
export const plansService = {
  getPlans: () => apiClient.get('/plans/'),
  createPlan: (planData) => apiClient.post('/plans/', planData),
  updatePlan: (id, planData) => apiClient.put(`/plans/${id}/`, planData),
  deletePlan: (id) => apiClient.delete(`/plans/${id}/`),
};
```

## Form Validation

### Client-Side Validation

```javascript
// validators.js
export const validateEmail = (email) => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
};

export const validatePassword = (password) => {
  return {
    minLength: password.length >= 8,
    hasUpperCase: /[A-Z]/.test(password),
    hasLowerCase: /[a-z]/.test(password),
    hasNumber: /\d/.test(password),
    hasSpecialChar: /[!@#$%^&*(),.?":{}|<>]/.test(password),
  };
};
```

## Progressive Web App (PWA)

### Service Worker Setup
- Cache static assets
- Offline functionality
- Push notifications (future feature)

### Manifest Configuration
```json
{
  "name": "Travel Planner",
  "short_name": "TravelPlan",
  "description": "Plan your perfect trip",
  "start_url": "/",
  "display": "standalone",
  "theme_color": "#2196F3",
  "background_color": "#ffffff",
  "icons": [...]
}
```

## Security Considerations

1. **Token Storage**: Use secure HTTP-only cookies or secure localStorage
2. **Input Sanitization**: Validate and sanitize all user inputs
3. **XSS Prevention**: Use React's built-in protection, avoid dangerouslySetInnerHTML
4. **CSRF Protection**: Include CSRF tokens in forms
5. **HTTPS**: Ensure all API calls use HTTPS in production

## Performance Optimization

1. **Code Splitting**: Use React.lazy() for route-based splitting
2. **Image Optimization**: Lazy loading and proper sizing
3. **Bundle Analysis**: Use webpack-bundle-analyzer
4. **Memoization**: React.memo() for expensive components
5. **Virtual Scrolling**: For large lists of plans

## Development Workflow

1. **Setup**: `npx create-react-app travel-planner-frontend`
2. **Dependencies**: Install Axios, React Router, UI framework
3. **Environment Variables**: Configure API endpoints
4. **Testing**: Jest and React Testing Library
5. **Building**: Optimize for production deployment

## TODO: Implementation Tasks

- [ ] Set up React project structure
- [ ] Implement authentication components
- [ ] Create plan management interface
- [ ] Add form validation
- [ ] Integrate with backend API
- [ ] Implement error handling
- [ ] Add loading states
- [ ] Configure PWA features
- [ ] Set up routing
- [ ] Add responsive design