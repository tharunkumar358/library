# Athena — Library Management System (MERN Stack)

A modern, full-stack web application designed for librarians and academic institutions to manage book inventories, patron memberships, and circulation records with real-time stock tracking and an attractive, aesthetic UI.

Built strictly according to the specifications in the **MERN Stack Project Documentation**.

---

## 📸 Overview & Key Features

- **📊 Comprehensive Dashboard**: Real-time KPI statistics including Total Books, Physical Copies, Active Members, Currently Issued Books, and Completed Returns, alongside recent circulation activity feeds and category distributions.
- **📚 Book Management (Full CRUD)**:
  - Add, view, edit, and delete books.
  - Search books in real time by title, author, or ISBN.
  - Category filtering (Computer Science, Fiction, Science, Philosophy, Literature, etc.).
  - Automatic inventory tracking: displays total copies vs. available copies with visual progress indicators.
  - Data integrity protection: prevents deleting books with active loans.
- **👥 Member Directory (Full CRUD)**:
  - Add, view, edit, and delete patrons.
  - Email format validation and duplicate prevention.
  - Status management (`active` / `inactive`).
  - Active loan protection: prevents removing members who haven't returned books.
- **🔄 Book Issue & Circulation Desk**:
  - Intuitive issue workflow with live preview.
  - Prevents issuing books when `availableQuantity === 0`.
  - Restricts issuing to active members.
  - Automatically calculates 14-day return due dates.
  - Atomically decrements available book stock.
- **↩️ Transactions & Return Management**:
  - Full history ledger of all current and past loans.
  - Filter by Active Loans or Returned Books.
  - Overdue loan badge indicators.
  - Single-click "Mark as Returned" action that atomically restores available book stock (+1).
  - Prevents duplicate returns.
- **🎨 Attractive & Aesthetic UI**:
  - Styled with Tailwind CSS, soft gradients, clean typography (Plus Jakarta Sans), Lucide icons, glassmorphism headers, responsive sidebar navigation, and floating toast notifications.

---

## 🛠️ Technology Stack

| Layer | Technology | Purpose |
| :--- | :--- | :--- |
| **Frontend** | React 18, Vite | Single-page reactive user interface |
| **Styling** | Tailwind CSS, Lucide React | Modern aesthetic styling, icons, and micro-interactions |
| **HTTP Client** | Axios | Frontend-to-backend REST API communication |
| **Backend** | Node.js, Express.js | REST API routing, business logic, and error handling |
| **Database** | MongoDB, Mongoose | Schema modeling, indexes, and document persistence |
| **Testing** | Node Fetch Test Runner | End-to-end automated verification suite |

---

## 📁 Project Structure

```
library-management-system/
├── client/                      # React Frontend (Vite)
│   ├── src/
│   │   ├── components/          # Reusable UI components
│   │   │   ├── Badge.jsx        # Status & stock badges
│   │   │   ├── Modal.jsx        # Animated modal dialogs
│   │   │   ├── Navbar.jsx       # Header & quick actions
│   │   │   ├── Notification.jsx # Floating toast alerts
│   │   │   ├── Sidebar.jsx      # Navigation sidebar
│   │   │   └── StatCard.jsx     # Dashboard KPI card
│   │   ├── pages/               # Main application views
│   │   │   ├── AddEditBookModal.jsx
│   │   │   ├── AddEditMemberModal.jsx
│   │   │   ├── BooksList.jsx
│   │   │   ├── Dashboard.jsx
│   │   │   ├── IssueBook.jsx
│   │   │   ├── MembersList.jsx
│   │   │   └── Transactions.jsx
│   │   ├── services/
│   │   │   └── api.js           # Central Axios service
│   │   ├── App.jsx              # Main React state & layout
│   │   ├── index.css            # Tailwind CSS directives
│   │   └── main.jsx             # React DOM root
│   ├── index.html
│   ├── tailwind.config.js
│   ├── vite.config.js
│   └── package.json
├── server/                      # Node.js / Express Backend
│   ├── config/
│   │   └── db.js                # MongoDB Mongoose connection
│   ├── controllers/
│   │   ├── bookController.js    # Books CRUD & stock validation
│   │   ├── memberController.js  # Members CRUD & loan checks
│   │   ├── transactionController.js # Issue & return operations
│   │   └── dashboardController.js   # KPI & metric aggregations
│   ├── models/
│   │   ├── Book.js              # Book schema & pre-save hooks
│   │   ├── Member.js            # Member schema & email regex
│   │   └── Transaction.js       # Transaction schema & refs
│   ├── routes/
│   │   ├── bookRoutes.js
│   │   ├── memberRoutes.js
│   │   ├── transactionRoutes.js
│   │   └── dashboardRoutes.js
│   ├── .env                     # Server environment variables
│   ├── seed.js                  # Realistic database seeder
│   ├── server.js                # Express app entrypoint
│   ├── test-all.js              # 23-point verification suite
│   └── package.json
├── package.json                 # Root script orchestration
└── README.md
```

---

## 🗄️ Database Design

### 1. Books Collection (`books`)
| Field | Type | Description |
| :--- | :--- | :--- |
| `_id` | ObjectId | Unique identifier |
| `title` | String | Title of the book (Required, Trimmed) |
| `author` | String | Author name (Required, Trimmed) |
| `category` | String | Genre / category classification (Required) |
| `isbn` | String | Unique ISBN code (Unique, Indexed) |
| `quantity` | Number | Total physical copies owned (Min: 1) |
| `availableQuantity` | Number | Copies currently on shelf (0 <= available <= quantity) |
| `createdAt` / `updatedAt`| Date | Timestamps |

### 2. Members Collection (`members`)
| Field | Type | Description |
| :--- | :--- | :--- |
| `_id` | ObjectId | Unique identifier |
| `name` | String | Full name (Required) |
| `email` | String | Email address (Unique, Validated) |
| `phone` | String | Contact phone number |
| `membershipDate` | Date | Date joined (Default: Date.now) |
| `status` | String | `active` or `inactive` |
| `createdAt` / `updatedAt`| Date | Timestamps |

### 3. Transactions Collection (`transactions`)
| Field | Type | Description |
| :--- | :--- | :--- |
| `_id` | ObjectId | Unique identifier |
| `bookId` | ObjectId | Reference to `Book` model |
| `memberId` | ObjectId | Reference to `Member` model |
| `issueDate` | Date | Date issued |
| `dueDate` | Date | Expected return date (Default: 14 days) |
| `returnDate` | Date | Date returned (null while issued) |
| `status` | String | `issued` or `returned` |
| `remarks` | String | Optional notes |

---

## 🔌 REST API Endpoints

### Book APIs
- `GET /api/books` — Retrieve all books (supports `?search=` and `?category=`)
- `GET /api/books/:id` — Retrieve single book details
- `POST /api/books` — Create a new book record
- `PUT /api/books/:id` — Update book record
- `DELETE /api/books/:id` — Delete a book record (guarded against active loans)

### Member APIs
- `GET /api/members` — Retrieve all members (supports `?search=` and `?status=`)
- `GET /api/members/:id` — Retrieve single member details
- `POST /api/members` — Register a new member
- `PUT /api/members/:id` — Update member information
- `DELETE /api/members/:id` — Remove a member (guarded against active loans)

### Transaction & Circulation APIs
- `GET /api/transactions` — Retrieve transaction ledger with populated books and members (supports `?status=` and `?search=`)
- `POST /api/transactions/issue` — Issue an available book to a member (`{ bookId, memberId, dueDate, remarks }`)
- `PUT /api/transactions/:id/return` — Mark an issued book as returned and restore inventory

### Dashboard API
- `GET /api/dashboard/stats` — Retrieve aggregated library metrics and recent activity

---

## 🚀 Installation & Running

### Prerequisites
- **Node.js** v18+ (tested with v24)
- **MongoDB** running on `localhost:27017`

### 1. Install Dependencies
From the project root:
```bash
npm run install:all
```
*(Or install inside `server/` and `client/` individually via `npm install`)*

### 2. Seed Sample Library Data
Populate the database with popular books, diverse patrons, and sample circulation records:
```bash
npm run seed
```

### 3. Start the Application
Run both backend and frontend servers:

**Backend Server** (runs on port 5000):
```bash
npm run server
```

**Frontend Client** (runs on port 5173):
```bash
npm run client
```

Open your browser at **`http://localhost:5173`**.

---

## 🧪 Automated Testing

An automated verification test suite is provided to validate all functional and validation requirements:
```bash
npm --prefix server test
```

### Test Suite Validations:
1. Backend connectivity and health check
2. Dashboard analytics and KPI aggregation
3. Book creation, update, listing, and deletion
4. Member registration, validation, and listing
5. Book checkout with stock decrement
6. Zero-stock issue rejection (preventing checkout when stock = 0)
7. Deletion block on books with active loans
8. Book return process with stock restoration (+1)
9. Duplicate return prevention
10. Form validation rejections (missing fields, invalid email format)
