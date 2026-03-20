# comp4537AssignmentFrontEnd
Front End of Term Assignment for COMP 4537 - AI Powered Classroom Web Application


Project Structure
src/
├── models/           ← Pure OOP data classes
│   ├── User.js           class User
│   ├── AuthToken.js      class AuthToken
│   ├── ApiUsage.js       class ApiUsageRecord, ApiUsageSummary
│   └── Classroom.js      class Question, StudentAnswer
│
├── services/         ← Service layer (inheritance + encapsulation)
│   ├── BaseApiService.js     abstract base class + class ApiError
│   ├── AuthService.js        extends BaseApiService
│   ├── ClassroomService.js   extends BaseApiService
│   └── AdminService.js       extends BaseApiService
│
├── hooks/            ← React hooks wrapping the service/context layer
│   ├── useAuth.jsx       AuthProvider + useAuth()
│   └── useApi.js         generic async call hook
│
├── components/common/
│   ├── UI.jsx            Button, Input, Textarea, Alert, Card, UsageMeter, Badge, Spinner
│   ├── Navbar.jsx        shared nav (renders differently per role)
│   └── ProtectedRoute.jsx  auth/role guard
│
├── pages/
│   ├── LandingPage.jsx
│   ├── AuthPages.jsx     LoginPage + RegisterPage + class FormValidator
│   ├── StudentDashboard.jsx
│   └── AdminDashboard.jsx
│
└── styles/global.css     full design system
