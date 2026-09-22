

Đây là roadmap theo dạng tree.

```text
JavaScript for Pentest
│
├── 1. Cơ bản (Bắt buộc)
│   │
│   ├── Variable
│   │   ├── let
│   │   ├── const
│   │   └── var
│   │
│   ├── Data Types
│   │   ├── Number
│   │   ├── String
│   │   ├── Boolean
│   │   ├── null
│   │   ├── undefined
│   │   ├── Object
│   │   └── Array
│   │
│   ├── Operators
│   ├── if / else
│   ├── switch
│   ├── for
│   ├── while
│   ├── Function
│   ├── Arrow Function
│   └── Scope
│
├── 2. Object & Array
│   │
│   ├── Object
│   ├── Property
│   ├── Method
│   ├── Array
│   ├── push()
│   ├── pop()
│   ├── map()
│   ├── filter()
│   └── forEach()
│
├── 3. String (Rất quan trọng)
│   │
│   ├── split()
│   ├── join()
│   ├── replace()
│   ├── slice()
│   ├── substring()
│   ├── indexOf()
│   ├── includes()
│   └── template literal
│
├── 4. DOM (Quan trọng nhất)
│   │
│   ├── document
│   ├── window
│   ├── querySelector()
│   ├── getElementById()
│   ├── innerHTML
│   ├── innerText
│   ├── textContent
│   ├── value
│   ├── setAttribute()
│   ├── createElement()
│   ├── appendChild()
│   └── remove()
│
├── 5. Event
│   │
│   ├── click
│   ├── submit
│   ├── input
│   ├── addEventListener()
│   ├── preventDefault()
│   └── Event Object
│
├── 6. Browser APIs
│   │
│   ├── location
│   ├── history
│   ├── navigator
│   ├── document.cookie
│   ├── localStorage
│   ├── sessionStorage
│   ├── alert()
│   ├── prompt()
│   └── console.log()
│
├── 7. HTTP
│   │
│   ├── fetch()
│   ├── XMLHttpRequest
│   ├── Headers
│   ├── GET
│   ├── POST
│   ├── JSON
│   ├── Response
│   └── Promise
│
├── 8. Async
│   │
│   ├── Promise
│   ├── async
│   ├── await
│   ├── then()
│   └── catch()
│
├── 9. JSON
│   │
│   ├── JSON.parse()
│   └── JSON.stringify()
│
├── 10. Encoding
│   │
│   ├── encodeURIComponent()
│   ├── decodeURIComponent()
│   ├── atob()
│   ├── btoa()
│   └── URLSearchParams
│
├── 11. DevTools
│   │
│   ├── Console
│   ├── Network
│   ├── Sources
│   ├── Breakpoint
│   └── Debugger
│
├── 12. ES6 (Đủ dùng)
│   │
│   ├── let
│   ├── const
│   ├── Arrow Function
│   ├── Destructuring
│   ├── Spread (...)
│   ├── Rest (...)
│   ├── Default Parameter
│   └── Module (import/export)
│
└── 13. JavaScript phục vụ Pentest
    │
    ├── DOM XSS
    ├── Reflected XSS
    ├── Stored XSS
    ├── Sink
    ├── Source
    ├── document.write()
    ├── innerHTML
    ├── eval()
    ├── setTimeout()
    ├── setInterval()
    ├── postMessage()
    ├── iframe
    ├── window.open()
    ├── Same Origin Policy
    ├── CORS
    ├── CSRF PoC
    ├── Cookie
    ├── localStorage Abuse
    ├── Prototype Pollution (đọc hiểu)
    └── Payload Writing
```


```text
FULLSTACK DEV — JavaScript → React → Node.js → Next.js
│
├── 0. JS Nền tảng — BẠN ĐÃ CÓ
│   │
│   ├── Variable / Data Types
│   ├── Condition / Loop
│   ├── Function / Arrow Function
│   ├── Object / Array
│   ├── String
│   ├── DOM / Event
│   ├── Browser API
│   ├── HTTP / Fetch
│   ├── Promise / async / await
│   ├── JSON
│   ├── ES6
│   └── DevTools
│
├── 1. JS NÂNG CAO — HỌC TIẾP
│   │
│   ├── Execution Model
│   │   ├── Call Stack
│   │   ├── Heap
│   │   ├── Event Loop
│   │   ├── Microtask Queue
│   │   └── Macrotask Queue
│   │
│   ├── Scope & Closure
│   │   ├── Lexical Scope
│   │   ├── Closure
│   │   ├── IIFE
│   │   └── Scope Chain
│   │
│   ├── this
│   │   ├── Global
│   │   ├── Object Method
│   │   ├── call()
│   │   ├── apply()
│   │   ├── bind()
│   │   └── Arrow Function vs normal function
│   │
│   ├── Object nâng cao
│   │   ├── Destructuring
│   │   ├── Spread / Rest
│   │   ├── Shallow Copy
│   │   ├── Deep Copy
│   │   ├── Object.keys()
│   │   ├── Object.values()
│   │   ├── Object.entries()
│   │   ├── Optional Chaining ?.
│   │   └── Nullish Coalescing ??
│   │
│   ├── Array nâng cao
│   │   ├── map()
│   │   ├── filter()
│   │   ├── reduce()
│   │   ├── find()
│   │   ├── some()
│   │   ├── every()
│   │   ├── sort()
│   │   └── flat()
│   │
│   ├── Function nâng cao
│   │   ├── Higher-Order Function
│   │   ├── Callback
│   │   ├── Pure Function
│   │   ├── Recursion
│   │   └── Function Composition
│   │
│   ├── Async sâu
│   │   ├── Promise.all()
│   │   ├── Promise.allSettled()
│   │   ├── Promise.race()
│   │   ├── Promise.any()
│   │   ├── Error Handling
│   │   └── Event Loop
│   │
│   ├── Module
│   │   ├── import
│   │   ├── export
│   │   ├── default export
│   │   ├── named export
│   │   └── CommonJS vs ES Module
│   │
│   ├── Error
│   │   ├── throw
│   │   ├── try/catch
│   │   ├── finally
│   │   └── Custom Error
│   │
│   ├── Class / OOP
│   │   ├── class
│   │   ├── constructor
│   │   ├── extends
│   │   ├── super
│   │   └── private field
│   │
│   ├── Web APIs
│   │   ├── URL
│   │   ├── FormData
│   │   ├── AbortController
│   │   ├── File API
│   │   └── WebSocket
│   │
│   └── Package Management
│       ├── npm
│       ├── package.json
│       ├── package-lock.json
│       ├── npm install
│       ├── npm scripts
│       └── dependencies / devDependencies
│
├── 2. HTML + CSS — ĐỦ ĐỂ BUILD UI
│   │
│   ├── HTML5
│   │   ├── Semantic HTML
│   │   ├── Form
│   │   ├── Input
│   │   ├── Button
│   │   └── Accessibility cơ bản
│   │
│   ├── CSS
│   │   ├── Box Model
│   │   ├── Display
│   │   ├── Position
│   │   ├── Flexbox
│   │   ├── Grid
│   │   ├── Responsive
│   │   ├── Media Query
│   │   └── CSS Variables
│   │
│   └── UI
│       ├── Component thinking
│       ├── Layout
│       ├── Form
│       ├── Modal
│       └── Loading / Error state
│
├── 3. REACT — FRONTEND CORE
│   │
│   ├── Fundamental
│   │   ├── Component
│   │   ├── JSX
│   │   ├── Props
│   │   ├── State
│   │   ├── Event
│   │   └── Conditional Rendering
│   │
│   ├── Hooks
│   │   ├── useState
│   │   ├── useEffect
│   │   ├── useRef
│   │   ├── useMemo
│   │   ├── useCallback
│   │   ├── useContext
│   │   └── Custom Hook
│   │
│   ├── Data Flow
│   │   ├── Parent → Child
│   │   ├── Child → Parent
│   │   ├── Lifting State Up
│   │   └── Context
│   │
│   ├── Forms
│   │   ├── Controlled Component
│   │   ├── Form Validation
│   │   ├── Submit
│   │   └── Error State
│   │
│   ├── Rendering
│   │   ├── Render
│   │   ├── Re-render
│   │   ├── Reconciliation
│   │   ├── key
│   │   └── Virtual DOM concept
│   │
│   ├── Routing
│   │   ├── React Router
│   │   ├── Route
│   │   ├── Params
│   │   ├── Nested Route
│   │   ├── Protected Route
│   │   └── Navigation
│   │
│   └── State Management
│       ├── Context
│       ├── Redux Toolkit
│       ├── Zustand
│       └── Server State
│
├── 4. FRONTEND API — REACT ↔ BACKEND
│   │
│   ├── HTTP
│   │   ├── GET
│   │   ├── POST
│   │   ├── PUT
│   │   ├── PATCH
│   │   └── DELETE
│   │
│   ├── Request
│   │   ├── Headers
│   │   ├── Body
│   │   ├── Query Parameters
│   │   ├── Path Parameters
│   │   └── Cookies
│   │
│   ├── Response
│   │   ├── Status Code
│   │   ├── JSON
│   │   ├── Error Response
│   │   └── Pagination
│   │
│   ├── API Client
│   │   ├── fetch()
│   │   ├── axios
│   │   ├── Interceptor
│   │   └── Request wrapper
│   │
│   └── Auth Frontend
│       ├── Login
│       ├── Logout
│       ├── Access Token
│       ├── Refresh Token
│       ├── Cookie
│       ├── Protected Page
│       └── Role-based UI
│
├── 5. NODE.JS — BACKEND
│   │
│   ├── Node Core
│   │   ├── Runtime
│   │   ├── Event Loop
│   │   ├── Modules
│   │   ├── process
│   │   ├── Buffer
│   │   ├── Stream
│   │   └── EventEmitter
│   │
│   ├── Built-in Modules
│   │   ├── fs
│   │   ├── path
│   │   ├── http
│   │   ├── crypto
│   │   ├── os
│   │   └── events
│   │
│   ├── Express.js
│   │   ├── Server
│   │   ├── Routing
│   │   ├── Request
│   │   ├── Response
│   │   ├── Middleware
│   │   ├── Router
│   │   └── Error Middleware
│   │
│   ├── REST API
│   │   ├── Resource
│   │   ├── CRUD
│   │   ├── Controller
│   │   ├── Service
│   │   ├── Repository
│   │   └── API Response format
│   │
│   ├── Validation
│   │   ├── Input Validation
│   │   ├── Schema Validation
│   │   ├── Zod
│   │   └── Sanitization
│   │
│   ├── Authentication
│   │   ├── Session
│   │   ├── JWT
│   │   ├── Access Token
│   │   ├── Refresh Token
│   │   ├── Cookie
│   │   ├── httpOnly
│   │   └── SameSite
│   │
│   ├── Authorization
│   │   ├── Role
│   │   ├── Permission
│   │   ├── RBAC
│   │   └── Middleware authorization
│   │
│   └── Security
│       ├── CORS
│       ├── CSRF
│       ├── XSS
│       ├── SQL Injection
│       ├── NoSQL Injection
│       ├── SSRF
│       ├── Rate Limiting
│       ├── Helmet
│       ├── Password Hashing
│       ├── bcrypt / Argon2
│       └── Secrets / Environment Variables
│
├── 6. DATABASE
│   │
│   ├── SQL — BẮT BUỘC
│   │   ├── PostgreSQL
│   │   ├── Table
│   │   ├── Row
│   │   ├── Primary Key
│   │   ├── Foreign Key
│   │   ├── JOIN
│   │   ├── Index
│   │   ├── Transaction
│   │   └── Normalization
│   │
│   ├── ORM
│   │   ├── Prisma
│   │   ├── Schema
│   │   ├── Migration
│   │   ├── Query
│   │   └── Relation
│   │
│   └── NoSQL — BIẾT
│       ├── MongoDB
│       ├── Document
│       ├── Collection
│       └── Mongoose
│
├── 7. TYPESCRIPT — NÊN HỌC SAU JS
│   │
│   ├── Type
│   ├── Interface
│   ├── Union
│   ├── Intersection
│   ├── Generic
│   ├── Enum
│   ├── Type Narrowing
│   ├── Utility Types
│   │   ├── Partial
│   │   ├── Pick
│   │   ├── Omit
│   │   └── Record
│   └── React + TypeScript
│
├── 8. NEXT.JS — FULLSTACK FRAMEWORK
│   │
│   ├── App Router
│   │   ├── app/
│   │   ├── page.tsx
│   │   ├── layout.tsx
│   │   ├── loading.tsx
│   │   ├── error.tsx
│   │   └── not-found.tsx
│   │
│   ├── Routing
│   │   ├── Static Route
│   │   ├── Dynamic Route
│   │   ├── Nested Route
│   │   ├── Route Groups
│   │   └── Catch-all Route
│   │
│   ├── React Server Components
│   │   ├── Server Component
│   │   ├── Client Component
│   │   ├── "use client"
│   │   └── Server ↔ Client boundary
│   │
│   ├── Data Fetching
│   │   ├── Server Fetch
│   │   ├── Client Fetch
│   │   ├── Caching
│   │   ├── Revalidation
│   │   └── Streaming
│   │
│   ├── Backend trong Next
│   │   ├── Route Handlers
│   │   ├── GET / POST
│   │   ├── Request / Response
│   │   ├── Cookies
│   │   ├── Headers
│   │   └── Middleware
│   │
│   ├── Server Actions
│   │   ├── "use server"
│   │   ├── Form Action
│   │   ├── Mutation
│   │   └── Validation
│   │
│   ├── Rendering
│   │   ├── SSR
│   │   ├── SSG
│   │   ├── ISR
│   │   ├── CSR
│   │   └── Streaming
│   │
│   ├── SEO
│   │   ├── Metadata
│   │   ├── generateMetadata()
│   │   ├── sitemap
│   │   └── robots
│   │
│   └── Optimization
│       ├── Image
│       ├── Font
│       ├── Code Splitting
│       ├── Lazy Loading
│       └── Caching
│
├── 9. FULLSTACK ARCHITECTURE
│   │
│   ├── Frontend
│   │   ├── UI
│   │   ├── State
│   │   └── API Client
│   │
│   ├── Backend
│   │   ├── Route
│   │   ├── Controller
│   │   ├── Service
│   │   └── Database
│   │
│   ├── Authentication Flow
│   │   ├── Register
│   │   ├── Login
│   │   ├── Session
│   │   ├── Refresh
│   │   └── Logout
│   │
│   ├── Architecture
│   │   ├── MVC
│   │   ├── Layered Architecture
│   │   ├── Clean Architecture
│   │   └── Repository Pattern
│   │
│   └── Error Handling
│       ├── Validation Error
│       ├── Auth Error
│       ├── Not Found
│       ├── Server Error
│       └── Global Error Handler
│
├── 10. TESTING
│   │
│   ├── Unit Test
│   │   ├── Jest
│   │   └── Vitest
│   │
│   ├── Integration Test
│   │
│   ├── E2E
│   │   └── Playwright
│   │
│   └── API Test
│       └── Supertest
│
├── 11. GIT + TOOLING
│   │
│   ├── Git
│   │   ├── clone
│   │   ├── branch
│   │   ├── merge
│   │   ├── rebase
│   │   └── pull request
│   │
│   ├── Environment
│   │   ├── .env
│   │   ├── .env.local
│   │   └── secrets
│   │
│   ├── Linting
│   │   ├── ESLint
│   │   └── Prettier
│   │
│   └── Debug
│       ├── Browser DevTools
│       ├── Node Inspector
│       └── Logs
│
├── 12. DEPLOYMENT
│   │
│   ├── Linux
│   │   ├── SSH
│   │   ├── Process
│   │   ├── Port
│   │   └── Permission
│   │
│   ├── Docker
│   │   ├── Image
│   │   ├── Container
│   │   ├── Dockerfile
│   │   └── Docker Compose
│   │
│   ├── Reverse Proxy
│   │   └── Nginx
│   │
│   ├── CI/CD
│   │   └── GitHub Actions
│   │
│   └── Cloud
│       ├── Vercel
│       ├── VPS
│       └── PostgreSQL Cloud
│
└── 13. PROJECT — HỌC THEO THỨ TỰ
    │
    ├── Project 1
    │   └── Todo App
    │       ├── React
    │       ├── State
    │       └── LocalStorage
    │
    ├── Project 2
    │   └── Blog
    │       ├── React
    │       ├── Node.js
    │       ├── Express
    │       ├── PostgreSQL
    │       └── REST API
    │
    ├── Project 3
    │   └── Authentication App
    │       ├── Register
    │       ├── Login
    │       ├── JWT / Session
    │       ├── Role
    │       └── Protected Route
    │
    ├── Project 4
    │   └── E-commerce
    │       ├── Product
    │       ├── Cart
    │       ├── Order
    │       ├── User
    │       ├── Payment mock
    │       └── Admin
    │
    └── Project 5
        └── Next.js Fullstack
            ├── App Router
            ├── Server Component
            ├── Client Component
            ├── Server Action
            ├── Route Handler
            ├── Prisma
            ├── PostgreSQL
            ├── Auth
            ├── Validation
            └── Deploy
```

```text
THỨ TỰ HỌC TỐI ƯU CHO BẠN
│
├── JS cơ bản ✅
│
├── JS nâng cao
│   └── Closure → this → Event Loop → Promise → Module
│
├── HTML/CSS
│   └── Flex → Grid → Responsive
│
├── React
│   └── JSX → Component → Props → State → Hooks → Router
│
├── API
│   └── HTTP → REST → JSON → Auth
│
├── Node.js
│   └── Node → Express → Middleware → REST API
│
├── Database
│   └── SQL → PostgreSQL → Prisma
│
├── TypeScript
│   └── Type → Interface → Generic → React TS
│
├── Next.js
│   └── App Router → RSC → Server Action → Route Handler
│
└── Fullstack Project
    └── React/Next + Node + PostgreSQL + Auth + Deploy
```

```text
VÌ BẠN HỌC JS CHO PENTEST TRƯỚC
│
├── Có thể BỎ QUA / học lướt
│   ├── DOM cơ bản
│   ├── Browser API cơ bản
│   ├── fetch cơ bản
│   ├── JSON
│   └── DevTools cơ bản
│
├── CẦN HỌC SÂU
│   ├── Closure
│   ├── this
│   ├── Event Loop
│   ├── Promise
│   ├── Module
│   ├── Array methods
│   ├── Error Handling
│   └── npm
│
└── CẦN ĐẶC BIỆT SÂU NẾU MUỐN VỪA DEV VỪA SECURITY
    ├── HTTP
    ├── Cookie
    ├── Session
    ├── JWT
    ├── CORS
    ├── CSRF
    ├── XSS
    ├── SSRF
    ├── SQL Injection
    ├── NoSQL Injection
    └── Authentication / Authorization
```

```text
MỤC TIÊU CUỐI
│
├── Frontend
│   └── React / Next.js
│
├── Backend
│   └── Node.js / Express / Next.js
│
├── Database
│   └── PostgreSQL / Prisma
│
├── Language
│   └── JavaScript → TypeScript
│
├── DevOps cơ bản
│   └── Git → Docker → Linux → CI/CD
│
└── Security advantage
    └── Hiểu cả code + HTTP + backend
        → rất thuận lợi cho Web Pentest / AppSec
```

Đối với nền hiện tại của bạn, điểm bắt đầu hợp lý nhất là **JS nâng cao → React → Node/Express → PostgreSQL/Prisma → TypeScript → Next.js**, không cần học lại phần JS cơ bản.