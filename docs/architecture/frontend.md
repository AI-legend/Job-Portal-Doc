# Frontend Architecture

## Overview

The frontend is built using React with Next.js framework, providing server-side rendering, static site generation, and excellent performance out of the box.

## Architecture Pattern

### Component-Based Architecture
```
Application
├── Layout Components
│   ├── Header
│   ├── Sidebar
│   ├── Footer
│   └── Navigation
├── Page Components
│   ├── Dashboard
│   ├── Jobs
│   ├── CV Builder
│   └── Profile
├── Feature Components
│   ├── JobSearch
│   ├── CVEditor
│   ├── ApplicationTracker
│   └── AgentInterface
└── UI Components
    ├── Buttons
    ├── Forms
    ├── Modals
    └── Inputs
```

## Folder Structure

```
frontend/
├── public/                 # Static assets
│   ├── images/
│   ├── icons/
│   └── manifest.json
├── src/
│   ├── pages/             # Next.js pages (routing)
│   │   ├── api/           # API routes
│   │   ├── auth/          # Authentication pages
│   │   ├── dashboard/     # Dashboard pages
│   │   ├── jobs/          # Job-related pages
│   │   ├── cv/            # CV builder pages
│   │   └── _app.js        # App wrapper
│   ├── components/        # React components
│   │   ├── common/        # Shared components
│   │   │   ├── Header/
│   │   │   ├── Footer/
│   │   │   ├── Modal/
│   │   │   └── Button/
│   │   ├── forms/         # Form components
│   │   │   ├── AuthForm/
│   │   │   ├── JobForm/
│   │   │   └── CVForm/
│   │   ├── features/      # Feature-specific components
│   │   │   ├── JobSearch/
│   │   │   ├── CVBuilder/
│   │   │   ├── AgentChat/
│   │   │   └── Dashboard/
│   │   └── layout/        # Layout components
│   │       ├── MainLayout/
│   │       ├── AuthLayout/
│   │       └── DashboardLayout/
│   ├── hooks/             # Custom React hooks
│   │   ├── useAuth.js
│   │   ├── useJobs.js
│   │   ├── useCV.js
│   │   └── useAgent.js
│   ├── services/          # API service layer
│   │   ├── api.js         # Base API configuration
│   │   ├── auth.js        # Authentication services
│   │   ├── jobs.js        # Job-related services
│   │   ├── cv.js          # CV services
│   │   └── agent.js       # AI agent services
│   ├── store/             # State management
│   │   ├── index.js       # Store configuration
│   │   ├── slices/        # Redux slices
│   │   │   ├── authSlice.js
│   │   │   ├── jobsSlice.js
│   │   │   ├── cvSlice.js
│   │   │   └── agentSlice.js
│   │   └── providers/     # Context providers
│   ├── utils/             # Utility functions
│   │   ├── helpers.js
│   │   ├── constants.js
│   │   ├── validators.js
│   │   └── formatters.js
│   ├── styles/            # Styling
│   │   ├── globals.css
│   │   ├── components/
│   │   └── pages/
│   └── types/             # TypeScript type definitions
│       ├── auth.ts
│       ├── jobs.ts
│       ├── cv.ts
│       └── common.ts
├── .env.local             # Environment variables
├── next.config.js         # Next.js configuration
├── tailwind.config.js     # Tailwind CSS configuration
├── package.json
└── README.md
```

## State Management Strategy

### Redux Toolkit Setup
```javascript
// store/index.js
import { configureStore } from '@reduxjs/toolkit'
import authSlice from './slices/authSlice'
import jobsSlice from './slices/jobsSlice'
import cvSlice from './slices/cvSlice'
import agentSlice from './slices/agentSlice'

export const store = configureStore({
  reducer: {
    auth: authSlice,
    jobs: jobsSlice,
    cv: cvSlice,
    agent: agentSlice,
  },
})
```

### Slice Structure Example
```javascript
// store/slices/authSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit'
import { authService } from '../../services/auth'

export const loginUser = createAsyncThunk(
  'auth/loginUser',
  async (credentials, { rejectWithValue }) => {
    try {
      const response = await authService.login(credentials)
      return response.data
    } catch (error) {
      return rejectWithValue(error.response.data)
    }
  }
)

const authSlice = createSlice({
  name: 'auth',
  initialState: {
    user: null,
    token: null,
    isLoading: false,
    error: null,
  },
  reducers: {
    logout: (state) => {
      state.user = null
      state.token = null
    },
    clearError: (state) => {
      state.error = null
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(loginUser.pending, (state) => {
        state.isLoading = true
        state.error = null
      })
      .addCase(loginUser.fulfilled, (state, action) => {
        state.isLoading = false
        state.user = action.payload.user
        state.token = action.payload.token
      })
      .addCase(loginUser.rejected, (state, action) => {
        state.isLoading = false
        state.error = action.payload
      })
  },
})

export const { logout, clearError } = authSlice.actions
export default authSlice.reducer
```

## Component Architecture

### Higher-Order Components (HOCs)
```javascript
// components/hoc/withAuth.js
import { useEffect } from 'react'
import { useRouter } from 'next/router'
import { useSelector } from 'react-redux'

const withAuth = (WrappedComponent) => {
  return (props) => {
    const router = useRouter()
    const { user, isLoading } = useSelector((state) => state.auth)

    useEffect(() => {
      if (!isLoading && !user) {
        router.push('/auth/login')
      }
    }, [user, isLoading, router])

    if (isLoading) {
      return <div>Loading...</div>
    }

    if (!user) {
      return null
    }

    return <WrappedComponent {...props} />
  }
}

export default withAuth
```

### Custom Hooks
```javascript
// hooks/useAuth.js
import { useSelector, useDispatch } from 'react-redux'
import { loginUser, logout } from '../store/slices/authSlice'

export const useAuth = () => {
  const dispatch = useDispatch()
  const { user, token, isLoading, error } = useSelector((state) => state.auth)

  const login = async (credentials) => {
    return dispatch(loginUser(credentials))
  }

  const handleLogout = () => {
    dispatch(logout())
    localStorage.removeItem('token')
  }

  return {
    user,
    token,
    isLoading,
    error,
    login,
    logout: handleLogout,
    isAuthenticated: !!user,
  }
}
```

## Routing Strategy

### Next.js File-based Routing
```
pages/
├── index.js                 # Home page (/)
├── auth/
│   ├── login.js            # Login page (/auth/login)
│   ├── register.js         # Register page (/auth/register)
│   └── forgot-password.js  # Password reset (/auth/forgot-password)
├── dashboard/
│   ├── index.js            # Dashboard home (/dashboard)
│   ├── seeker.js           # Job seeker dashboard (/dashboard/seeker)
│   └── recruiter.js        # Recruiter dashboard (/dashboard/recruiter)
├── jobs/
│   ├── index.js            # Job listings (/jobs)
│   ├── [id].js             # Job details (/jobs/[id])
│   ├── search.js           # Job search (/jobs/search)
│   └── apply/
│       └── [id].js         # Job application (/jobs/apply/[id])
├── cv/
│   ├── index.js            # CV overview (/cv)
│   ├── builder.js          # CV builder (/cv/builder)
│   ├── templates.js        # CV templates (/cv/templates)
│   └── [id]/
│       ├── edit.js         # Edit CV (/cv/[id]/edit)
│       └── view.js         # View CV (/cv/[id]/view)
├── agent/
│   ├── index.js            # Agent interface (/agent)
│   ├── chat.js             # Chat with agent (/agent/chat)
│   └── applications.js     # Auto-applied jobs (/agent/applications)
└── api/                    # API routes
    ├── auth/
    ├── jobs/
    ├── cv/
    └── agent/
```

## Performance Optimization

### Code Splitting
```javascript
// Dynamic imports for large components
import dynamic from 'next/dynamic'

const CVBuilder = dynamic(() => import('../components/features/CVBuilder'), {
  loading: () => <p>Loading CV Builder...</p>,
  ssr: false
})

const Dashboard = dynamic(() => import('../components/features/Dashboard'), {
  loading: () => <p>Loading Dashboard...</p>
})
```

### Image Optimization
```javascript
// Using Next.js Image component
import Image from 'next/image'

const ProfileImage = ({ src, alt }) => (
  <Image
    src={src}
    alt={alt}
    width={200}
    height={200}
    priority={true}
    placeholder="blur"
    blurDataURL="data:image/jpeg;base64,..."
  />
)
```

### Caching Strategy
```javascript
// API caching with SWR
import useSWR from 'swr'

const useJobs = (filters) => {
  const { data, error } = useSWR(
    filters ? ['jobs', filters] : null,
    () => jobsService.getJobs(filters),
    {
      revalidateOnFocus: false,
      dedupingInterval: 60000, // 1 minute
    }
  )

  return {
    jobs: data,
    isLoading: !error && !data,
    isError: error
  }
}
```

## Styling Architecture

### Tailwind CSS Configuration
```javascript
// tailwind.config.js
module.exports = {
  content: [
    './src/pages/**/*.{js,ts,jsx,tsx}',
    './src/components/**/*.{js,ts,jsx,tsx}',
  ],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#eff6ff',
          500: '#3b82f6',
          900: '#1e3a8a',
        },
        secondary: {
          50: '#f9fafb',
          500: '#6b7280',
          900: '#111827',
        }
      },
      fontFamily: {
        sans: ['Inter', 'sans-serif'],
      },
    },
  },
  plugins: [
    require('@tailwindcss/forms'),
    require('@tailwindcss/typography'),
  ],
}
```

### Component Styling Pattern
```javascript
// Using CSS Modules with Tailwind
import styles from './Button.module.css'
import clsx from 'clsx'

const Button = ({ variant = 'primary', size = 'md', children, ...props }) => {
  const baseClasses = 'font-medium rounded-lg focus:outline-none focus:ring-2'
  
  const variantClasses = {
    primary: 'bg-blue-600 text-white hover:bg-blue-700 focus:ring-blue-500',
    secondary: 'bg-gray-200 text-gray-900 hover:bg-gray-300 focus:ring-gray-500',
  }
  
  const sizeClasses = {
    sm: 'px-3 py-2 text-sm',
    md: 'px-4 py-2 text-base',
    lg: 'px-6 py-3 text-lg',
  }

  return (
    <button
      className={clsx(
        baseClasses,
        variantClasses[variant],
        sizeClasses[size]
      )}
      {...props}
    >
      {children}
    </button>
  )
}
```

## Testing Strategy

### Unit Testing Setup
```javascript
// __tests__/components/Button.test.js
import { render, screen, fireEvent } from '@testing-library/react'
import { Button } from '../src/components/common/Button'

describe('Button Component', () => {
  test('renders button with correct text', () => {
    render(<Button>Click me</Button>)
    expect(screen.getByRole('button')).toHaveTextContent('Click me')
  })

  test('calls onClick handler when clicked', () => {
    const handleClick = jest.fn()
    render(<Button onClick={handleClick}>Click me</Button>)
    
    fireEvent.click(screen.getByRole('button'))
    expect(handleClick).toHaveBeenCalledTimes(1)
  })
})
```

### Integration Testing
```javascript
// __tests__/pages/login.test.js
import { render, screen, fireEvent, waitFor } from '@testing-library/react'
import { Provider } from 'react-redux'
import { store } from '../src/store'
import LoginPage from '../src/pages/auth/login'

const renderWithRedux = (component) => {
  return render(
    <Provider store={store}>
      {component}
    </Provider>
  )
}

describe('Login Page', () => {
  test('submits form with correct credentials', async () => {
    renderWithRedux(<LoginPage />)
    
    fireEvent.change(screen.getByLabelText(/email/i), {
      target: { value: 'test@example.com' }
    })
    
    fireEvent.change(screen.getByLabelText(/password/i), {
      target: { value: 'password123' }
    })
    
    fireEvent.click(screen.getByRole('button', { name: /login/i }))
    
    await waitFor(() => {
      expect(screen.getByText(/dashboard/i)).toBeInTheDocument()
    })
  })
})
```

## Deployment Configuration

### Next.js Configuration
```javascript
// next.config.js
const nextConfig = {
  reactStrictMode: true,
  swcMinify: true,
  images: {
    domains: ['localhost', 'api.example.com', 's3.amazonaws.com'],
  },
  env: {
    CUSTOM_KEY: process.env.CUSTOM_KEY,
  },
  async rewrites() {
    return [
      {
        source: '/api/:path*',
        destination: `${process.env.API_BASE_URL}/:path*`,
      },
    ]
  },
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff',
          },
          {
            key: 'X-Frame-Options',
            value: 'DENY',
          },
        ],
      },
    ]
  },
}

module.exports = nextConfig
```

### Build Optimization
```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "test": "jest --watch",
    "test:ci": "jest --ci --coverage",
    "analyze": "cross-env ANALYZE=true next build"
  }
}
```
