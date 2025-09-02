I'll guide you through starting the FCMB authentication project step by step.

## Step 1: Create Project Directory and Files

### Create the project folder:
```bash
mkdir fcmb-auth-api
cd fcmb-auth-api
```

### Create all necessary files:
```bash
# Create directory structure
mkdir routes middleware config models

# Create package.json
touch package.json

# Create main server file
touch server.js

# Create route files
touch routes/auth.js

# Create middleware files
touch middleware/encryption.js

# Create environment file
touch .env

# Create .gitignore
touch .gitignore
```

## Step 2: Set Up Package.json

Copy this into `package.json`:
```json
{
  "name": "fcmb-auth-api",
  "version": "1.0.0",
  "description": "FCMB-style authentication API",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "dependencies": {
    "express": "^4.18.2",
    "cors": "^2.8.5",
    "body-parser": "^1.20.2",
    "bcryptjs": "^2.4.3",
    "jsonwebtoken": "^9.0.2",
    "axios": "^1.6.2",
    "dotenv": "^16.3.1"
  },
  "devDependencies": {
    "nodemon": "^3.0.2"
  },
  "keywords": ["authentication", "fcmb", "api"],
  "author": "Your Name",
  "license": "MIT"
}
```

## Step 3: Set Up Environment Variables

Copy this into `.env`:
```env
PORT=3000
JWT_SECRET=your-super-secret-jwt-key-change-this-in-production
NODE_ENV=development
```

## Step 4: Create the Encryption Middleware

Copy this into `middleware/encryption.js`:
```javascript
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

## Step 5: Create Auth Routes

Copy this into `routes/auth.js`:
```javascript
const express = require('express');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const { 
    generateFCMBPassword, 
    generateFCMBUsername, 
    decryptData 
} = require('../middleware/encryption');

const router = express.Router();

// Mock user database
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

        // Verify password
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

module.exports = router;
```

## Step 6: Create Main Server File

Copy this into `server.js`:
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
  res.json({ 
    status: 'OK', 
    message: 'FCMB Auth API is running',
    timestamp: new Date().toISOString()
  });
});

// Root endpoint
app.get('/', (req, res) => {
  res.json({
    message: 'FCMB Authentication API',
    endpoints: {
      health: '/api/health',
      login: '/api/auth/login',
      encrypt: '/api/auth/encrypt-credentials',
      verify: '/api/auth/verify-token'
    }
  });
});

// Error handling middleware
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({
    success: false,
    message: 'Something went wrong!'
  });
});

// 404 handler
app.use('*', (req, res) => {
  res.status(404).json({
    success: false,
    message: 'Endpoint not found'
  });
});

app.listen(PORT, () => {
  console.log(`🚀 Server running on port ${PORT}`);
  console.log(`📍 Health check: http://localhost:${PORT}/api/health`);
  console.log(`🔗 API base URL: http://localhost:${PORT}/api`);
});
```

## Step 7: Create .gitignore

Copy this into `.gitignore`:
```gitignore
# Dependencies
node_modules/
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# Environment variables
.env
.env.local
.env.development.local
.env.test.local
.env.production.local

# Logs
logs
*.log

# Runtime data
pids
*.pid
*.seed
*.pid.lock

# Coverage directory used by tools like istanbul
coverage/

# nyc test coverage
.nyc_output

# OS generated files
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
ehthumbs.db
Thumbs.db

# IDE files
.vscode/
.idea/
*.swp
*.swo
```

## Step 8: Install Dependencies

```bash
npm install
```

## Step 9: Start the Server

### For development (with auto-restart):
```bash
npm run dev
```

### For production:
```bash
npm start
```

## Step 10: Test the API

### Test 1: Health Check
```bash
curl http://localhost:3000/api/health
```

### Test 2: Encrypt Credentials
```bash
curl -X POST http://localhost:3000/api/auth/encrypt-credentials \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "password": "Password123!"
  }'
```

### Test 3: Login with Encrypted Credentials
```bash
# Use the encrypted values from the previous response
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "encryptedUsername": "VkVFRUdNVWpCWFhBQUFBV1RnPT09PT09",
    "encryptedPassword": "V1VaTVdGakFYQUFBQUFBRGtBPT09PT09"
  }'
```

## Step 11: Create a Simple Test Frontend

Create `test.html` in your project root:
```html
<!DOCTYPE html>
<html>
<head>
    <title>FCMB Login Test</title>
    <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
    <style>
        body { font-family: Arial, sans-serif; max-width: 400px; margin: 50px auto; }
        .form-group { margin: 15px 0; }
        input { width: 100%; padding: 8px; margin: 5px 0; }
        button { background: #007bff; color: white; padding: 10px 15px; border: none; cursor: pointer; }
        .result { margin-top: 20px; padding: 10px; background: #f4f4f4; }
    </style>
</head>
<body>
    <h2>FCMB Login Test</h2>
    
    <div class="form-group">
        <label>Username:</label>
        <input type="text" id="username" value="testuser">
    </div>
    
    <div class="form-group">
        <label>Password:</label>
        <input type="password" id="password" value="Password123!">
    </div>
    
    <button onclick="testLogin()">Test Login</button>
    
    <div class="result" id="result"></div>

    <script>
        function encryptData(data) {
            if (!data) return '';
            const randomPrefix = makeid(5);
            const randomSuffix = makeid(5);
            const base64Data = btoa(data);
            return btoa(randomPrefix + base64Data + randomSuffix);
        }

        function makeid(length) {
            let result = '';
            const characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
            for (let i = 0; i < length; i++) {
                result += characters.charAt(Math.floor(Math.random() * characters.length));
            }
            return result;
        }

        async function testLogin() {
            const username = document.getElementById('username').value;
            const password = document.getElementById('password').value;
            
            const encryptedUsername = encryptData(username);
            const encryptedPassword = encryptData(password);
            
            try {
                const response = await axios.post('http://localhost:3000/api/auth/login', {
                    encryptedUsername,
                    encryptedPassword
                });
                
                document.getElementById('result').innerHTML = 
                    `<strong>Success:</strong> ${JSON.stringify(response.data, null, 2)}`;
            } catch (error) {
                document.getElementById('result').innerHTML = 
                    `<strong>Error:</strong> ${error.response?.data?.message || error.message}`;
            }
        }
    </script>
</body>
</html>
```

## Step 12: Open the Test Page

Open `test.html` in your browser or serve it with:
```bash
# If you have Python installed
python -m http.server 8000

# Or with Node.js http-server
npx http-server
```

Then visit `http://localhost:8000/test.html`

Your FCMB authentication API is now running! The server will start on port 3000 and you can test the endpoints using the provided examples.
