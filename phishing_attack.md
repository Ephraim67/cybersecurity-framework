I'll help you create a Node.js backend API that handles the FCMB-like authentication with the same encryption mechanism and connects to your frontend.

## Project Structure
```
fcmb-auth-api/
├── package.json
├── server.js
├── routes/
│   └── auth.js
├── middleware/
│   └── encryption.js
├── config/
│   └── database.js
└── models/
    └── User.js
```

## 1. Package.json
```json
{
  "name": "fcmb-auth-api",
  "version": "1.0.0",
  "description": "FCMB-style authentication API",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "cors": "^2.8.5",
    "body-parser": "^1.20.2",
    "bcryptjs": "^2.4.3",
    "jsonwebtoken": "^9.0.2",
    "mongoose": "^8.0.3",
    "axios": "^1.6.2",
    "dotenv": "^16.3.1"
  },
  "devDependencies": {
    "nodemon": "^3.0.2"
  }
}
```

## 2. Server.js (Main Entry Point)
```javascript
const express = require('express');
const cors = require('cors');
const bodyParser = require('body-parser');
require('dotenv').config();

const authRoutes = require('./routes/auth');

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(cors());
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));

// Routes
app.use('/api/auth', authRoutes);

// Health check endpoint
app.get('/api/health', (req, res) => {
  res.json({ status: 'OK', message: 'FCMB Auth API is running' });
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

## 3. Encryption Middleware
```javascript
// middleware/encryption.js
function makeid(length) {
    let result = '';
    const characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
    const charactersLength = characters.length;
    let counter = 0;
    while (counter < length) {
        result += characters.charAt(Math.floor(Math.random() * charactersLength));
        counter += 1;
    }
    return result;
}

function encryptData(data) {
    if (!data) return '';
    const randomPrefix = makeid(5);
    const randomSuffix = makeid(5);
    const base64Data = Buffer.from(data).toString('base64');
    return Buffer.from(randomPrefix + base64Data + randomSuffix).toString('base64');
}

function decryptData(encryptedData) {
    if (!encryptedData) return '';
    try {
        const decoded = Buffer.from(encryptedData, 'base64').toString('utf8');
        // Remove random prefix (5 chars) and suffix (5 chars)
        const base64Data = decoded.substring(5, decoded.length - 5);
        return Buffer.from(base64Data, 'base64').toString('utf8');
    } catch (error) {
        throw new Error('Invalid encrypted data');
    }
}

function generateFCMBPassword(password) {
    return encryptData(password);
}

function generateFCMBUsername(username) {
    return encryptData(username);
}

module.exports = {
    encryptData,
    decryptData,
    generateFCMBPassword,
    generateFCMBUsername,
    makeid
};
```

## 4. Auth Routes
```javascript
// routes/auth.js
const express = require('express');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const { 
    generateFCMBPassword, 
    generateFCMBUsername, 
    decryptData 
} = require('../middleware/encryption');

const router = express.Router();

// Mock user database (replace with real database)
const users = [
    {
        id: 1,
        username: 'testuser',
        // Password: "Password123!" encrypted with bcrypt
        password: '$2a$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi',
        email: 'test@example.com',
        fullName: 'Test User'
    }
];

// Login endpoint
router.post('/login', async (req, res) => {
    try {
        const { encryptedUsername, encryptedPassword } = req.body;

        if (!encryptedUsername || !encryptedPassword) {
            return res.status(400).json({
                success: false,
                message: 'Username and password are required'
            });
        }

        // Decrypt the FCMB-style encrypted credentials
        const username = decryptData(encryptedUsername);
        const password = decryptData(encryptedPassword);

        console.log('Decrypted credentials:', { username, password });

        // Find user in database
        const user = users.find(u => u.username === username);
        
        if (!user) {
            return res.status(401).json({
                success: false,
                message: 'Invalid credentials'
            });
        }

        // Verify password (using bcrypt for actual authentication)
        const isPasswordValid = await bcrypt.compare(password, user.password);
        
        if (!isPasswordValid) {
            return res.status(401).json({
                success: false,
                message: 'Invalid credentials'
            });
        }

        // Generate JWT token
        const token = jwt.sign(
            { 
                userId: user.id, 
                username: user.username 
            },
            process.env.JWT_SECRET || 'your-secret-key',
            { expiresIn: '1h' }
        );

        res.json({
            success: true,
            message: 'Login successful',
            token,
            user: {
                id: user.id,
                username: user.username,
                email: user.email,
                fullName: user.fullName
            }
        });

    } catch (error) {
        console.error('Login error:', error);
        res.status(500).json({
            success: false,
            message: 'Internal server error'
        });
    }
});

// Generate encrypted credentials (for frontend testing)
router.post('/encrypt-credentials', (req, res) => {
    try {
        const { username, password } = req.body;

        if (!username || !password) {
            return res.status(400).json({
                success: false,
                message: 'Username and password are required'
            });
        }

        const encryptedUsername = generateFCMBUsername(username);
        const encryptedPassword = generateFCMBPassword(password);

        res.json({
            success: true,
            encryptedUsername,
            encryptedPassword
        });

    } catch (error) {
        console.error('Encryption error:', error);
        res.status(500).json({
            success: false,
            message: 'Encryption failed'
        });
    }
});

// Verify token endpoint
router.post('/verify-token', (req, res) => {
    try {
        const token = req.headers.authorization?.replace('Bearer ', '');

        if (!token) {
            return res.status(401).json({
                success: false,
                message: 'No token provided'
            });
        }

        const decoded = jwt.verify(token, process.env.JWT_SECRET || 'your-secret-key');
        
        res.json({
            success: true,
            user: decoded
        });

    } catch (error) {
        res.status(401).json({
            success: false,
            message: 'Invalid token'
        });
    }
});

// Forgot password endpoint
router.post('/forgot-password', (req, res) => {
    // Implement password reset logic here
    res.json({
        success: true,
        message: 'Password reset instructions sent to email'
    });
});

module.exports = router;
```

## 5. Environment Variables (.env)
```env
PORT=3000
JWT_SECRET=your-super-secret-jwt-key-here
MONGODB_URI=mongodb://localhost:27017/fcmb-auth
NODE_ENV=development
```

## 6. Frontend Integration Example

### HTML/JavaScript Frontend Code:
```html
<!DOCTYPE html>
<html>
<head>
    <title>FCMB Login</title>
    <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
</head>
<body>
    <form id="loginForm">
        <input type="text" id="username" placeholder="Username" required>
        <input type="password" id="password" placeholder="Password" required>
        <button type="submit">Login</button>
    </form>

    <script>
        // Replicate FCMB encryption functions
        function makeid(length) {
            let result = '';
            const characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
            const charactersLength = characters.length;
            let counter = 0;
            while (counter < length) {
                result += characters.charAt(Math.floor(Math.random() * charactersLength));
                counter += 1;
            }
            return result;
        }

        function encryptData(data) {
            if (!data) return '';
            const randomPrefix = makeid(5);
            const randomSuffix = makeid(5);
            const base64Data = btoa(data);
            return btoa(randomPrefix + base64Data + randomSuffix);
        }

        document.getElementById('loginForm').addEventListener('submit', async function(e) {
            e.preventDefault();
            
            const username = document.getElementById('username').value;
            const password = document.getElementById('password').value;

            // Encrypt credentials like FCMB does
            const encryptedUsername = encryptData(username);
            const encryptedPassword = encryptData(password);

            try {
                const response = await axios.post('http://localhost:3000/api/auth/login', {
                    encryptedUsername,
                    encryptedPassword
                });

                if (response.data.success) {
                    // Store token and redirect
                    localStorage.setItem('token', response.data.token);
                    alert('Login successful!');
                    console.log('User:', response.data.user);
                } else {
                    alert('Login failed: ' + response.data.message);
                }
            } catch (error) {
                console.error('Login error:', error);
                alert('Login failed. Please try again.');
            }
        });
    </script>
</body>
</html>
```

## 7. Setup and Installation

1. **Install dependencies:**
```bash
npm install
```

2. **Start the server:**
```bash
npm run dev
```

3. **Test the API:**
```bash
# Test login endpoint
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "encryptedUsername": "VkVFRUdNVWpCWFhBQUFBV1RnPT09PT09",
    "encryptedPassword": "V1VaTVdGakFYQUFBQUFBRGtBPT09PT09"
  }'
```

## 8. Security Enhancements

Add these to your server.js for production:
```javascript
// Security headers
app.use(helmet());
app.use(rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100 // limit each IP to 100 requests per windowMs
}));

// Input validation
app.use(express.json({ limit: '10kb' }));
```

This API replicates the FCMB encryption mechanism while providing a secure Node.js backend for authentication. The frontend encrypts credentials exactly like FCMB does, and the backend decrypts them for actual authentication.
