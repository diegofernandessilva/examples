# Agent Instructions for Express.js Backend on AWS Lambda

## Project Overview

This is a serverless Express.js backend application deployed to AWS Lambda using the Serverless Framework. It demonstrates how to develop and deploy a simple Node.js Express API using the serverless-http wrapper.

## Technology Stack

- **Framework**: Express.js 4.x
- **Runtime**: Node.js (AWS Lambda compatible)
- **Language**: JavaScript (ES6+)
- **Deployment**: AWS Lambda with Serverless Framework
- **Adapter**: serverless-http (wraps Express for Lambda)

## Project Structure

```
├── handler.js         # Express app and Lambda handler
├── package.json       # Dependencies and scripts
└── serverless.yml     # Serverless Framework configuration
```

## Architecture Pattern

### Express Application
- Traditional Express.js application structure
- Uses `serverless-http` to adapt Express for AWS Lambda
- Single Lambda function handles all HTTP routes

### Lambda Integration
The `handler.js` exports a Lambda handler that wraps the Express app:
```javascript
const serverless = require("serverless-http");
const express = require("express");
const app = express();

// Define routes
app.get("/", (req, res) => { ... });
app.get("/hello", (req, res) => { ... });

// Export Lambda handler
module.exports.handler = serverless(app);
```

## Development Guidelines

### Adding New Routes

1. **Define routes in handler.js**:
   ```javascript
   app.get("/path", (req, res, next) => {
     return res.status(200).json({
       message: "Response data"
     });
   });
   ```

2. **HTTP Methods**:
   - `app.get()` - GET requests
   - `app.post()` - POST requests
   - `app.put()` - PUT requests
   - `app.delete()` - DELETE requests
   - `app.patch()` - PATCH requests

3. **Route Parameters**:
   ```javascript
   app.get("/users/:id", (req, res) => {
     const userId = req.params.id;
     // Handle request
   });
   ```

4. **Query Parameters**:
   ```javascript
   app.get("/search", (req, res) => {
     const query = req.query.q;
     // Handle search
   });
   ```

5. **Request Body** (for POST/PUT):
   ```javascript
   // First, add body parser middleware
   app.use(express.json());
   
   app.post("/data", (req, res) => {
     const data = req.body;
     // Process data
   });
   ```

### Error Handling

1. **404 Handler** (already implemented):
   ```javascript
   app.use((req, res, next) => {
     return res.status(404).json({
       error: "Not Found"
     });
   });
   ```

2. **General Error Handler**:
   ```javascript
   app.use((err, req, res, next) => {
     console.error(err.stack);
     return res.status(500).json({
       error: "Internal Server Error"
     });
   });
   ```

### Middleware

Add middleware for common tasks:

```javascript
// Body parsing
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// CORS
const cors = require('cors');
app.use(cors());

// Logging
const morgan = require('morgan');
app.use(morgan('combined'));

// Custom middleware
app.use((req, res, next) => {
  req.requestTime = Date.now();
  next();
});
```

## Code Organization Best Practices

### For Larger Applications

Consider organizing code into separate modules:

```javascript
// routes/users.js
const express = require('express');
const router = express.Router();

router.get('/', (req, res) => { ... });
router.post('/', (req, res) => { ... });

module.exports = router;

// handler.js
const userRoutes = require('./routes/users');
app.use('/users', userRoutes);
```

### Controller Pattern

Separate business logic from routes:

```javascript
// controllers/userController.js
exports.getUsers = (req, res) => {
  // Business logic
};

// routes/users.js
const { getUsers } = require('../controllers/userController');
router.get('/', getUsers);
```

## Commands

### Development
```bash
npm install              # Install dependencies
serverless offline       # Start local development server (requires plugin)
```

### Deployment
```bash
serverless deploy        # Deploy to AWS
serverless remove        # Remove deployed service
```

### Testing
```bash
# Local testing with curl
curl http://localhost:3000/
curl http://localhost:3000/hello
```

## Serverless Configuration

The `serverless.yml` defines:
- Service name: `aws-node-express-api`
- Single function: `api`
- Event configuration: `httpApi` with catch-all routing
- All HTTP methods and paths are handled by Express routing

### HTTP API Configuration
```yaml
functions:
  api:
    handler: handler.handler
    events:
      - httpApi: '*'
```

This configuration passes all requests to Express, which handles routing internally.

## Local Development

### Setting up serverless-offline

1. Install the plugin:
   ```bash
   serverless plugin install -n serverless-offline
   ```

2. Start local server:
   ```bash
   serverless offline
   ```

3. Test endpoints:
   ```bash
   curl http://localhost:3000/
   ```

## Testing Strategy

### Manual Testing
- Use curl, Postman, or similar tools
- Test all endpoints and HTTP methods
- Verify response status codes and bodies
- Test error cases (404, 500, etc.)

### Automated Testing
Consider adding:

```javascript
// tests/handler.test.js
const request = require('supertest');
const app = require('../handler');

describe('GET /', () => {
  it('should return hello message', async () => {
    const response = await request(app)
      .get('/')
      .expect(200);
    expect(response.body.message).toBe('Hello from root!');
  });
});
```

## Performance Considerations

### Cold Starts
- Keep dependencies minimal
- Consider Lambda provisioned concurrency for production
- Use Lambda layers for large dependencies

### Optimization Tips
1. Minimize npm package size
2. Use `package.json` to exclude dev dependencies from deployment
3. Enable compression for responses
4. Implement caching strategies

## Security Best Practices

1. **Input Validation**:
   ```javascript
   const validator = require('express-validator');
   
   app.post('/user', [
     validator.body('email').isEmail(),
     validator.body('age').isInt()
   ], (req, res) => {
     const errors = validator.validationResult(req);
     if (!errors.isEmpty()) {
       return res.status(400).json({ errors: errors.array() });
     }
     // Process valid data
   });
   ```

2. **Environment Variables**:
   - Never hardcode secrets
   - Use AWS Secrets Manager or Parameter Store
   - Access via `process.env.VARIABLE_NAME`

3. **CORS Configuration**:
   ```javascript
   const cors = require('cors');
   app.use(cors({
     origin: 'https://yourdomain.com',
     methods: ['GET', 'POST'],
     credentials: true
   }));
   ```

4. **Rate Limiting**:
   ```javascript
   const rateLimit = require('express-rate-limit');
   
   const limiter = rateLimit({
     windowMs: 15 * 60 * 1000, // 15 minutes
     max: 100 // limit each IP to 100 requests per windowMs
   });
   
   app.use(limiter);
   ```

5. **Helmet for Security Headers**:
   ```javascript
   const helmet = require('helmet');
   app.use(helmet());
   ```

## Common Patterns

### RESTful API Structure
```javascript
// CRUD operations for a resource
app.get('/items', getAllItems);           // List all
app.get('/items/:id', getItemById);       // Get one
app.post('/items', createItem);           // Create
app.put('/items/:id', updateItem);        // Update
app.delete('/items/:id', deleteItem);     // Delete
```

### Async/Await Pattern
```javascript
app.get('/data', async (req, res, next) => {
  try {
    const data = await fetchDataFromDatabase();
    return res.status(200).json({ data });
  } catch (error) {
    next(error); // Pass to error handler
  }
});
```

### Response Formatting
```javascript
// Success response
res.status(200).json({
  success: true,
  data: result
});

// Error response
res.status(400).json({
  success: false,
  error: "Invalid request",
  message: "Detailed error message"
});
```

## Debugging

### Local Debugging
- Use `console.log()` for simple debugging
- Use Node.js debugger with `serverless-offline`
- Check Lambda logs with `serverless logs -f api -t`

### Production Debugging
- View CloudWatch Logs
- Enable X-Ray for tracing
- Use structured logging (JSON format)

## Monitoring

### CloudWatch Metrics
- Monitor Lambda invocations
- Track error rates
- Monitor duration and memory usage

### Logging Best Practices
```javascript
// Structured logging
const log = (level, message, data) => {
  console.log(JSON.stringify({
    timestamp: new Date().toISOString(),
    level,
    message,
    data
  }));
};

app.use((req, res, next) => {
  log('info', 'Request received', {
    method: req.method,
    path: req.path
  });
  next();
});
```

## API Documentation

Consider adding API documentation:
- Use Swagger/OpenAPI specification
- Add JSDoc comments to routes
- Create a README with endpoint documentation

## Common Tasks

### Adding Database Integration
```javascript
// Example with MongoDB
const mongoose = require('mongoose');

// Connect to database
mongoose.connect(process.env.MONGODB_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true
});

// Use in routes
app.get('/users', async (req, res) => {
  const users = await User.find();
  res.json({ users });
});
```

### Adding Authentication
```javascript
const jwt = require('jsonwebtoken');

// Middleware to verify JWT
const authenticate = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' });
  }
};

// Protected route
app.get('/protected', authenticate, (req, res) => {
  res.json({ message: 'Access granted', user: req.user });
});
```

## Conventions

- **HTTP Status Codes**:
  - 200: Success
  - 201: Created
  - 400: Bad Request
  - 401: Unauthorized
  - 404: Not Found
  - 500: Internal Server Error

- **Response Format**: Consistent JSON structure
- **Error Messages**: Clear and helpful, but don't expose sensitive info
- **Route Naming**: Use kebab-case for URLs (`/user-profiles`)

## Resources

- [Express.js Documentation](https://expressjs.com/)
- [serverless-http GitHub](https://github.com/dougmoscrop/serverless-http)
- [Serverless Framework Documentation](https://www.serverless.com/framework/docs/)
- [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/)
