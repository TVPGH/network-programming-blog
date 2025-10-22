---
title: "Xây Dựng REST API với Node.js và Express"
date: 2024-02-20
tags: ["JavaScript", "Node.js", "Express", "REST API", "Backend"]
draft: false
---

Node.js và Express.js là một bộ đôi mạnh mẽ để xây dựng REST API. Trong bài viết này, chúng ta sẽ tìm hiểu cách tạo một REST API hoàn chỉnh.

## Cài đặt và Setup

```bash
# Tạo project mới
mkdir my-api
cd my-api
npm init -y

# Cài đặt dependencies
npm install express cors helmet morgan dotenv
npm install -D nodemon
```

## Cấu trúc project

```
my-api/
├── src/
│   ├── controllers/
│   ├── models/
│   ├── routes/
│   ├── middleware/
│   └── utils/
├── package.json
└── server.js
```

## Server cơ bản

```javascript
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const morgan = require('morgan');
require('dotenv').config();

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(helmet());
app.use(cors());
app.use(morgan('combined'));
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Routes
app.use('/api/users', require('./routes/users'));
app.use('/api/posts', require('./routes/posts'));

// Error handling middleware
app.use((err, req, res, next) => {
    console.error(err.stack);
    res.status(500).json({
        error: 'Something went wrong!',
        message: err.message
    });
});

// 404 handler
app.use('*', (req, res) => {
    res.status(404).json({
        error: 'Route not found',
        message: `Cannot ${req.method} ${req.originalUrl}`
    });
});

app.listen(PORT, () => {
    console.log(`Server is running on port ${PORT}`);
});
```

## Model và Database

```javascript
// models/User.js
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
    name: {
        type: String,
        required: true,
        trim: true
    },
    email: {
        type: String,
        required: true,
        unique: true,
        lowercase: true
    },
    age: {
        type: Number,
        min: 0,
        max: 120
    },
    isActive: {
        type: Boolean,
        default: true
    }
}, {
    timestamps: true
});

module.exports = mongoose.model('User', userSchema);
```

## Controller

```javascript
// controllers/userController.js
const User = require('../models/User');

class UserController {
    // GET /api/users
    async getAllUsers(req, res) {
        try {
            const { page = 1, limit = 10, search } = req.query;
            const query = search ? { name: { $regex: search, $options: 'i' } } : {};
            
            const users = await User.find(query)
                .limit(limit * 1)
                .skip((page - 1) * limit)
                .sort({ createdAt: -1 });
                
            const total = await User.countDocuments(query);
            
            res.json({
                users,
                totalPages: Math.ceil(total / limit),
                currentPage: page,
                total
            });
        } catch (error) {
            res.status(500).json({ error: error.message });
        }
    }
    
    // GET /api/users/:id
    async getUserById(req, res) {
        try {
            const user = await User.findById(req.params.id);
            if (!user) {
                return res.status(404).json({ error: 'User not found' });
            }
            res.json(user);
        } catch (error) {
            res.status(500).json({ error: error.message });
        }
    }
    
    // POST /api/users
    async createUser(req, res) {
        try {
            const user = new User(req.body);
            await user.save();
            res.status(201).json(user);
        } catch (error) {
            if (error.code === 11000) {
                res.status(400).json({ error: 'Email already exists' });
            } else {
                res.status(400).json({ error: error.message });
            }
        }
    }
    
    // PUT /api/users/:id
    async updateUser(req, res) {
        try {
            const user = await User.findByIdAndUpdate(
                req.params.id,
                req.body,
                { new: true, runValidators: true }
            );
            
            if (!user) {
                return res.status(404).json({ error: 'User not found' });
            }
            
            res.json(user);
        } catch (error) {
            res.status(400).json({ error: error.message });
        }
    }
    
    // DELETE /api/users/:id
    async deleteUser(req, res) {
        try {
            const user = await User.findByIdAndDelete(req.params.id);
            if (!user) {
                return res.status(404).json({ error: 'User not found' });
            }
            res.json({ message: 'User deleted successfully' });
        } catch (error) {
            res.status(500).json({ error: error.message });
        }
    }
}

module.exports = new UserController();
```

## Routes

```javascript
// routes/users.js
const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');
const { validateUser, validateUpdateUser } = require('../middleware/validation');

// GET /api/users
router.get('/', userController.getAllUsers);

// GET /api/users/:id
router.get('/:id', userController.getUserById);

// POST /api/users
router.post('/', validateUser, userController.createUser);

// PUT /api/users/:id
router.put('/:id', validateUpdateUser, userController.updateUser);

// DELETE /api/users/:id
router.delete('/:id', userController.deleteUser);

module.exports = router;
```

## Middleware

```javascript
// middleware/validation.js
const { body, validationResult } = require('express-validator');

const validateUser = [
    body('name')
        .notEmpty()
        .withMessage('Name is required')
        .isLength({ min: 2, max: 50 })
        .withMessage('Name must be between 2 and 50 characters'),
    
    body('email')
        .isEmail()
        .withMessage('Please provide a valid email')
        .normalizeEmail(),
    
    body('age')
        .optional()
        .isInt({ min: 0, max: 120 })
        .withMessage('Age must be between 0 and 120'),
    
    (req, res, next) => {
        const errors = validationResult(req);
        if (!errors.isEmpty()) {
            return res.status(400).json({
                error: 'Validation failed',
                details: errors.array()
            });
        }
        next();
    }
];

const validateUpdateUser = [
    body('name')
        .optional()
        .isLength({ min: 2, max: 50 })
        .withMessage('Name must be between 2 and 50 characters'),
    
    body('email')
        .optional()
        .isEmail()
        .withMessage('Please provide a valid email')
        .normalizeEmail(),
    
    body('age')
        .optional()
        .isInt({ min: 0, max: 120 })
        .withMessage('Age must be between 0 and 120'),
    
    (req, res, next) => {
        const errors = validationResult(req);
        if (!errors.isEmpty()) {
            return res.status(400).json({
                error: 'Validation failed',
                details: errors.array()
            });
        }
        next();
    }
];

module.exports = { validateUser, validateUpdateUser };
```

## Authentication Middleware

```javascript
// middleware/auth.js
const jwt = require('jsonwebtoken');

const authenticateToken = (req, res, next) => {
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1];
    
    if (!token) {
        return res.status(401).json({ error: 'Access token required' });
    }
    
    jwt.verify(token, process.env.JWT_SECRET, (err, user) => {
        if (err) {
            return res.status(403).json({ error: 'Invalid token' });
        }
        req.user = user;
        next();
    });
};

module.exports = { authenticateToken };
```

## Error Handling

```javascript
// middleware/errorHandler.js
const errorHandler = (err, req, res, next) => {
    let error = { ...err };
    error.message = err.message;
    
    // Mongoose bad ObjectId
    if (err.name === 'CastError') {
        const message = 'Resource not found';
        error = { message, statusCode: 404 };
    }
    
    // Mongoose duplicate key
    if (err.code === 11000) {
        const message = 'Duplicate field value entered';
        error = { message, statusCode: 400 };
    }
    
    // Mongoose validation error
    if (err.name === 'ValidationError') {
        const message = Object.values(err.errors).map(val => val.message);
        error = { message, statusCode: 400 };
    }
    
    res.status(error.statusCode || 500).json({
        success: false,
        error: error.message || 'Server Error'
    });
};

module.exports = errorHandler;
```

## Testing

```javascript
// tests/user.test.js
const request = require('supertest');
const app = require('../server');

describe('User API', () => {
    test('GET /api/users should return all users', async () => {
        const response = await request(app)
            .get('/api/users')
            .expect(200);
            
        expect(response.body).toHaveProperty('users');
        expect(Array.isArray(response.body.users)).toBe(true);
    });
    
    test('POST /api/users should create a new user', async () => {
        const userData = {
            name: 'John Doe',
            email: 'john@example.com',
            age: 30
        };
        
        const response = await request(app)
            .post('/api/users')
            .send(userData)
            .expect(201);
            
        expect(response.body).toHaveProperty('_id');
        expect(response.body.name).toBe(userData.name);
    });
});
```

## Kết luận

Node.js và Express.js cung cấp một nền tảng mạnh mẽ để xây dựng REST API. Với cấu trúc rõ ràng, middleware linh hoạt và khả năng mở rộng cao, đây là lựa chọn tốt cho việc phát triển backend API.
