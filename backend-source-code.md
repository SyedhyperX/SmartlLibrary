# SmartLibrary Backend Source Code

## 📁 File Structure
```
server/
├── models/
│   ├── User.js
│   ├── Book.js
│   └── Transaction.js
├── routes/
│   ├── auth.js
│   ├── books.js
│   ├── users.js
│   └── transactions.js
├── middleware/
│   ├── auth.js
│   └── admin.js
├── config/
│   └── database.js
├── server.js
└── package.json
```

---

## 📦 package.json
```json
{
  "name": "smartlibrary-server",
  "version": "1.0.0",
  "description": "Backend server for SmartLibrary Management System",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [
    "library",
    "management", 
    "nodejs",
    "express",
    "mongodb",
    "jwt"
  ],
  "author": "SyedhyperX",
  "license": "MIT",
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^7.5.0",
    "bcryptjs": "^2.4.3",
    "jsonwebtoken": "^9.0.2",
    "cors": "^2.8.5",
    "dotenv": "^16.3.1",
    "multer": "^1.4.5",
    "express-validator": "^7.0.1",
    "nodemailer": "^6.9.4",
    "moment": "^2.29.4"
  },
  "devDependencies": {
    "nodemon": "^3.0.1"
  }
}
```

---

## 🗄️ models/User.js
```javascript
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');

const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'Please provide a name'],
    trim: true,
    maxlength: [50, 'Name cannot be more than 50 characters']
  },
  email: {
    type: String,
    required: [true, 'Please provide an email'],
    unique: true,
    lowercase: true,
    match: [
      /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
      'Please provide a valid email'
    ]
  },
  password: {
    type: String,
    required: [true, 'Please provide a password'],
    minlength: [6, 'Password must be at least 6 characters'],
    select: false
  },
  role: {
    type: String,
    enum: ['user', 'admin'],
    default: 'user'
  },
  phone: {
    type: String,
    trim: true,
    match: [/^[0-9]{10}$/, 'Please provide a valid 10-digit phone number']
  },
  address: {
    street: String,
    city: String,
    state: String,
    pincode: String
  },
  membershipType: {
    type: String,
    enum: ['basic', 'premium', 'student'],
    default: 'basic'
  },
  membershipDate: {
    type: Date,
    default: Date.now
  },
  status: {
    type: String,
    enum: ['active', 'suspended', 'inactive'],
    default: 'active'
  },
  totalFine: {
    type: Number,
    default: 0
  },
  booksIssued: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Transaction'
  }],
  profileImage: {
    type: String,
    default: ''
  }
}, {
  timestamps: true,
  toJSON: { virtuals: true },
  toObject: { virtuals: true }
});

// Virtual for full name display
userSchema.virtual('fullAddress').get(function() {
  if (this.address) {
    return `${this.address.street || ''}, ${this.address.city || ''}, ${this.address.state || ''} ${this.address.pincode || ''}`.trim();
  }
  return '';
});

// Hash password before saving
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next();
  
  const salt = await bcrypt.genSalt(12);
  this.password = await bcrypt.hash(this.password, salt);
  next();
});

// Compare password method
userSchema.methods.comparePassword = async function(candidatePassword) {
  return await bcrypt.compare(candidatePassword, this.password);
};

// Get active user transactions
userSchema.methods.getActiveTransactions = function() {
  return this.populate({
    path: 'booksIssued',
    match: { status: 'issued' },
    populate: {
      path: 'book',
      select: 'title author isbn'
    }
  });
};

module.exports = mongoose.model('User', userSchema);
```

---

## 📚 models/Book.js
```javascript
const mongoose = require('mongoose');

const bookSchema = new mongoose.Schema({
  title: {
    type: String,
    required: [true, 'Please provide book title'],
    trim: true,
    maxlength: [200, 'Title cannot be more than 200 characters']
  },
  author: {
    type: String,
    required: [true, 'Please provide author name'],
    trim: true,
    maxlength: [100, 'Author name cannot be more than 100 characters']
  },
  isbn: {
    type: String,
    required: [true, 'Please provide ISBN'],
    unique: true,
    trim: true,
    match: [/^[0-9-]+$/, 'Please provide a valid ISBN']
  },
  genre: {
    type: String,
    required: [true, 'Please provide book genre'],
    enum: [
      'Fiction', 'Non-Fiction', 'Science Fiction', 'Fantasy', 'Mystery',
      'Romance', 'Thriller', 'Horror', 'Biography', 'History',
      'Science', 'Technology', 'Business', 'Self-Help', 'Health',
      'Travel', 'Religion', 'Philosophy', 'Art', 'Sports'
    ]
  },
  description: {
    type: String,
    maxlength: [1000, 'Description cannot be more than 1000 characters']
  },
  publisher: {
    type: String,
    required: [true, 'Please provide publisher name'],
    trim: true
  },
  publishedDate: {
    type: Date,
    required: [true, 'Please provide published date']
  },
  pages: {
    type: Number,
    required: [true, 'Please provide number of pages'],
    min: [1, 'Pages must be at least 1']
  },
  language: {
    type: String,
    required: [true, 'Please provide book language'],
    default: 'English'
  },
  totalCopies: {
    type: Number,
    required: [true, 'Please provide total copies'],
    min: [1, 'Total copies must be at least 1']
  },
  availableCopies: {
    type: Number,
    required: [true, 'Please provide available copies'],
    min: [0, 'Available copies cannot be negative']
  },
  location: {
    shelf: {
      type: String,
      required: [true, 'Please provide shelf information']
    },
    section: {
      type: String,
      required: [true, 'Please provide section information']
    }
  },
  coverImage: {
    type: String,
    default: ''
  },
  rating: {
    type: Number,
    default: 0,
    min: 0,
    max: 5
  },
  reviews: [{
    user: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User',
      required: true
    },
    rating: {
      type: Number,
      required: true,
      min: 1,
      max: 5
    },
    comment: {
      type: String,
      maxlength: [500, 'Review comment cannot exceed 500 characters']
    },
    date: {
      type: Date,
      default: Date.now
    }
  }],
  status: {
    type: String,
    enum: ['available', 'maintenance', 'lost', 'damaged'],
    default: 'available'
  },
  addedBy: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  }
}, {
  timestamps: true,
  toJSON: { virtuals: true },
  toObject: { virtuals: true }
});

// Virtual for availability status
bookSchema.virtual('isAvailable').get(function() {
  return this.availableCopies > 0 && this.status === 'available';
});

// Virtual for issue count
bookSchema.virtual('issueCount').get(function() {
  return this.totalCopies - this.availableCopies;
});

// Update availability before saving
bookSchema.pre('save', function(next) {
  if (this.availableCopies > this.totalCopies) {
    this.availableCopies = this.totalCopies;
  }
  next();
});

// Calculate average rating
bookSchema.methods.calculateAverageRating = function() {
  if (this.reviews.length === 0) {
    this.rating = 0;
    return;
  }
  
  const sum = this.reviews.reduce((acc, review) => acc + review.rating, 0);
  this.rating = Math.round((sum / this.reviews.length) * 10) / 10;
};

// Static method to find books by availability
bookSchema.statics.findAvailable = function() {
  return this.find({ availableCopies: { $gt: 0 }, status: 'available' });
};

module.exports = mongoose.model('Book', bookSchema);
```

---

## 💳 models/Transaction.js
```javascript
const mongoose = require('mongoose');
const moment = require('moment');

const transactionSchema = new mongoose.Schema({
  user: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: [true, 'Please provide user']
  },
  book: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Book',
    required: [true, 'Please provide book']
  },
  issueDate: {
    type: Date,
    default: Date.now,
    required: true
  },
  dueDate: {
    type: Date,
    required: true
  },
  returnDate: {
    type: Date
  },
  status: {
    type: String,
    enum: ['issued', 'returned', 'overdue', 'lost'],
    default: 'issued'
  },
  fine: {
    amount: {
      type: Number,
      default: 0,
      min: 0
    },
    paid: {
      type: Boolean,
      default: false
    },
    paidDate: {
      type: Date
    }
  },
  renewalCount: {
    type: Number,
    default: 0,
    max: [3, 'Maximum 3 renewals allowed']
  },
  renewalHistory: [{
    date: {
      type: Date,
      default: Date.now
    },
    previousDueDate: Date,
    newDueDate: Date,
    reason: String
  }],
  notes: {
    type: String,
    maxlength: [500, 'Notes cannot exceed 500 characters']
  },
  issuedBy: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  returnedBy: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  }
}, {
  timestamps: true,
  toJSON: { virtuals: true },
  toObject: { virtuals: true }
});

// Virtual for days overdue
transactionSchema.virtual('daysOverdue').get(function() {
  if (this.status !== 'overdue' && this.status !== 'returned') return 0;
  
  const compareDate = this.returnDate || new Date();
  const overdueDays = moment(compareDate).diff(moment(this.dueDate), 'days');
  return Math.max(0, overdueDays);
});

// Virtual for fine calculation
transactionSchema.virtual('calculatedFine').get(function() {
  const daysOverdue = this.daysOverdue;
  if (daysOverdue <= 0) return 0;
  
  // Fine rate: ₹5 per day for first 7 days, ₹10 per day thereafter
  const basicDays = Math.min(daysOverdue, 7);
  const extraDays = Math.max(0, daysOverdue - 7);
  
  return (basicDays * 5) + (extraDays * 10);
});

// Pre-save middleware to set due date
transactionSchema.pre('save', function(next) {
  // Set due date to 14 days from issue date if not set
  if (!this.dueDate && this.issueDate) {
    this.dueDate = moment(this.issueDate).add(14, 'days').toDate();
  }
  
  // Auto-calculate fine if overdue
  if (this.status === 'overdue' || (this.returnDate && moment(this.returnDate).isAfter(this.dueDate))) {
    this.fine.amount = this.calculatedFine;
    if (this.returnDate && moment(this.returnDate).isAfter(this.dueDate)) {
      this.status = 'returned'; // But with fine
    }
  }
  
  next();
});

// Method to renew book
transactionSchema.methods.renewBook = function(adminId, reason = '') {
  if (this.renewalCount >= 3) {
    throw new Error('Maximum renewal limit reached');
  }
  
  if (this.status !== 'issued') {
    throw new Error('Can only renew issued books');
  }
  
  const previousDueDate = this.dueDate;
  const newDueDate = moment(this.dueDate).add(14, 'days').toDate();
  
  this.renewalHistory.push({
    previousDueDate,
    newDueDate,
    reason
  });
  
  this.dueDate = newDueDate;
  this.renewalCount += 1;
  
  return this;
};

// Method to return book
transactionSchema.methods.returnBook = function(adminId) {
  if (this.status === 'returned') {
    throw new Error('Book already returned');
  }
  
  this.returnDate = new Date();
  this.returnedBy = adminId;
  
  // Calculate fine if overdue
  if (moment(this.returnDate).isAfter(this.dueDate)) {
    this.fine.amount = this.calculatedFine;
    this.status = 'returned';
  } else {
    this.status = 'returned';
  }
  
  return this;
};

// Static method to get overdue transactions
transactionSchema.statics.getOverdue = function() {
  return this.find({
    status: 'issued',
    dueDate: { $lt: new Date() }
  }).populate('user book');
};

module.exports = mongoose.model('Transaction', transactionSchema);
```

---

## 🔧 config/database.js
```javascript
const mongoose = require('mongoose');

const connectDB = async () => {
  try {
    const conn = await mongoose.connect(process.env.MONGODB_URI, {
      useNewUrlParser: true,
      useUnifiedTopology: true
    });

    console.log(`MongoDB Connected: ${conn.connection.host}`.cyan.underline.bold);

    // Handle connection events
    mongoose.connection.on('error', (err) => {
      console.error('MongoDB connection error:', err);
    });

    mongoose.connection.on('disconnected', () => {
      console.log('MongoDB disconnected');
    });

    // Handle app termination
    process.on('SIGINT', async () => {
      await mongoose.connection.close();
      console.log('MongoDB connection closed.');
      process.exit(0);
    });

  } catch (error) {
    console.error('Database connection error:', error);
    process.exit(1);
  }
};

module.exports = connectDB;
```

---

## 🔒 middleware/auth.js
```javascript
const jwt = require('jsonwebtoken');
const User = require('../models/User');

// Protect routes - verify JWT token
const protect = async (req, res, next) => {
  let token;

  // Check for token in headers
  if (
    req.headers.authorization &&
    req.headers.authorization.startsWith('Bearer')
  ) {
    try {
      // Get token from header
      token = req.headers.authorization.split(' ')[1];

      // Verify token
      const decoded = jwt.verify(token, process.env.JWT_SECRET);

      // Get user from token and attach to request
      req.user = await User.findById(decoded.id).select('-password');

      if (!req.user) {
        return res.status(401).json({
          success: false,
          message: 'Not authorized - user not found'
        });
      }

      // Check if user is active
      if (req.user.status !== 'active') {
        return res.status(401).json({
          success: false,
          message: 'Account is suspended or inactive'
        });
      }

      next();
    } catch (error) {
      console.error('Token verification error:', error);
      return res.status(401).json({
        success: false,
        message: 'Not authorized - token failed'
      });
    }
  }

  if (!token) {
    return res.status(401).json({
      success: false,
      message: 'Not authorized - no token provided'
    });
  }
};

// Generate JWT Token
const generateToken = (id) => {
  return jwt.sign({ id }, process.env.JWT_SECRET, {
    expiresIn: process.env.JWT_EXPIRE || '7d'
  });
};

module.exports = {
  protect,
  generateToken
};
```

---

## 👮 middleware/admin.js
```javascript
const adminOnly = (req, res, next) => {
  // Check if user is authenticated
  if (!req.user) {
    return res.status(401).json({
      success: false,
      message: 'Not authenticated'
    });
  }

  // Check if user is admin
  if (req.user.role !== 'admin') {
    return res.status(403).json({
      success: false,
      message: 'Access denied. Admin privileges required.'
    });
  }

  next();
};

module.exports = {
  adminOnly
};
```

---

## 🚀 server.js
```javascript
const express = require('express');
const cors = require('cors');
const dotenv = require('dotenv');
const colors = require('colors');
const connectDB = require('./config/database');

// Load environment variables
dotenv.config();

// Connect to database
connectDB();

const app = express();

// Enable CORS
app.use(cors({
  origin: process.env.CLIENT_URL || 'http://localhost:3000',
  credentials: true
}));

// Body parser middleware
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true, limit: '10mb' }));

// Routes
app.use('/api/auth', require('./routes/auth'));
app.use('/api/books', require('./routes/books'));
app.use('/api/users', require('./routes/users'));
app.use('/api/transactions', require('./routes/transactions'));

// Health check route
app.get('/api/health', (req, res) => {
  res.json({
    success: true,
    message: 'SmartLibrary API is running',
    timestamp: new Date().toISOString(),
    version: '1.0.0'
  });
});

// Welcome route
app.get('/', (req, res) => {
  res.json({
    success: true,
    message: 'Welcome to SmartLibrary API',
    documentation: '/api/docs',
    health: '/api/health'
  });
});

// Global error handler
app.use((err, req, res, next) => {
  console.error('Global error handler:', err);

  // Default error
  res.status(err.statusCode || 500).json({
    success: false,
    message: err.message || 'Server Error',
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
  });
});

// 404 handler
app.use((req, res) => {
  res.status(404).json({
    success: false,
    message: 'Route not found',
    path: req.originalUrl
  });
});

const PORT = process.env.PORT || 5000;

const server = app.listen(PORT, () => {
  console.log(
    `Server running in ${process.env.NODE_ENV} mode on port ${PORT}`.yellow.bold
  );
});

module.exports = app;
```

---

## 🔐 routes/auth.js
```javascript
const express = require('express');
const { body, validationResult } = require('express-validator');
const User = require('../models/User');
const { protect, generateToken } = require('../middleware/auth');

const router = express.Router();

// @desc    Register user
// @route   POST /api/auth/register
// @access  Public
router.post('/register', [
  body('name')
    .trim()
    .notEmpty()
    .withMessage('Name is required')
    .isLength({ min: 2, max: 50 })
    .withMessage('Name must be between 2-50 characters'),
  body('email')
    .isEmail()
    .normalizeEmail()
    .withMessage('Please provide a valid email'),
  body('password')
    .isLength({ min: 6 })
    .withMessage('Password must be at least 6 characters')
], async (req, res) => {
  try {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({
        success: false,
        message: 'Validation failed',
        errors: errors.array()
      });
    }

    const { name, email, password, phone, address } = req.body;

    // Check if user already exists
    const existingUser = await User.findOne({ email });
    if (existingUser) {
      return res.status(400).json({
        success: false,
        message: 'User already exists with this email'
      });
    }

    // Create user
    const user = await User.create({
      name,
      email,
      password,
      phone,
      address
    });

    // Generate token
    const token = generateToken(user._id);

    res.status(201).json({
      success: true,
      message: 'User registered successfully',
      data: {
        user: {
          _id: user._id,
          name: user.name,
          email: user.email,
          role: user.role
        },
        token
      }
    });

  } catch (error) {
    console.error('Registration error:', error);
    res.status(500).json({
      success: false,
      message: 'Server error during registration'
    });
  }
});

// @desc    Login user
// @route   POST /api/auth/login
// @access  Public
router.post('/login', [
  body('email')
    .isEmail()
    .normalizeEmail()
    .withMessage('Please provide a valid email'),
  body('password')
    .notEmpty()
    .withMessage('Password is required')
], async (req, res) => {
  try {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({
        success: false,
        message: 'Validation failed',
        errors: errors.array()
      });
    }

    const { email, password } = req.body;

    // Find user and include password
    const user = await User.findOne({ email }).select('+password');
    if (!user) {
      return res.status(401).json({
        success: false,
        message: 'Invalid credentials'
      });
    }

    // Check password
    const isPasswordValid = await user.comparePassword(password);
    if (!isPasswordValid) {
      return res.status(401).json({
        success: false,
        message: 'Invalid credentials'
      });
    }

    // Generate token
    const token = generateToken(user._id);

    res.json({
      success: true,
      message: 'Login successful',
      data: {
        user: {
          _id: user._id,
          name: user.name,
          email: user.email,
          role: user.role
        },
        token
      }
    });

  } catch (error) {
    console.error('Login error:', error);
    res.status(500).json({
      success: false,
      message: 'Server error during login'
    });
  }
});

// @desc    Get current user profile
// @route   GET /api/auth/profile
// @access  Private
router.get('/profile', protect, async (req, res) => {
  try {
    const user = await User.findById(req.user._id);
    res.json({
      success: true,
      data: { user }
    });
  } catch (error) {
    console.error('Profile fetch error:', error);
    res.status(500).json({
      success: false,
      message: 'Server error fetching profile'
    });
  }
});

module.exports = router;
```

---

## ⚙️ Environment Configuration (.env)
```env
# Database Configuration
MONGODB_URI=mongodb://localhost:27017/smartlibrary

# JWT Configuration
JWT_SECRET=your_super_secret_jwt_key_here_make_it_long_and_random
JWT_EXPIRE=7d

# Server Configuration
PORT=5000
NODE_ENV=development

# Client URL (for CORS)
CLIENT_URL=http://localhost:3000

# Email Configuration (Optional)
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USER=your_email@gmail.com
EMAIL_PASS=your_app_password
```

---

## 🚀 Getting Started

### 1. Install Dependencies
```bash
cd server
npm install
```

### 2. Setup Environment
Copy `.env.example` to `.env` and update the values

### 3. Start MongoDB
Make sure MongoDB is running on your system

### 4. Run Development Server
```bash
npm run dev
```

The server will be running on http://localhost:5000

---

## 📋 API Endpoints Summary

### Authentication
- `POST /api/auth/register` - Register new user
- `POST /api/auth/login` - User login
- `GET /api/auth/profile` - Get user profile

### Books
- `GET /api/books` - Get all books
- `POST /api/books` - Add book (Admin only)
- `GET /api/books/:id` - Get single book
- `PUT /api/books/:id` - Update book (Admin only)
- `DELETE /api/books/:id` - Delete book (Admin only)

### Transactions
- `POST /api/transactions/issue` - Issue book (Admin only)
- `PUT /api/transactions/return/:id` - Return book (Admin only)
- `GET /api/transactions/user` - Get user transactions
- `GET /api/transactions/admin` - Get all transactions (Admin only)

---

This backend provides a robust foundation for the SmartLibrary management system with comprehensive user management, book catalog, and transaction handling capabilities.