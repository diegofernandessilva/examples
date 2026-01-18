# Agent Instructions for TypeScript REST API with MongoDB Backend

## Project Overview

This is a serverless REST API backend built with TypeScript and MongoDB Atlas, deployed to AWS Lambda using the Serverless Framework. It demonstrates a complete CRUD application with proper layered architecture (Controller-Service-Model pattern).

## Technology Stack

- **Language**: TypeScript 3.x
- **Runtime**: Node.js 12.x on AWS Lambda
- **Database**: MongoDB Atlas (cloud-hosted MongoDB)
- **ODM**: Mongoose 5.x
- **Framework**: Serverless Framework
- **Testing**: Mocha, Chai, lambda-tester
- **Linting**: TSLint with Airbnb config
- **Environment Management**: dotenv

## Project Structure

```
├── app/
│   ├── controller/       # HTTP request handlers
│   │   └── books.ts
│   ├── service/          # Business logic layer
│   ├── model/            # Mongoose models and schemas
│   │   ├── books.ts
│   │   ├── dto/          # Data Transfer Objects
│   │   └── vo/           # Value Objects
│   ├── utils/            # Utility functions
│   └── handler.ts        # Lambda function exports
├── config/
│   ├── .env.dev          # Development environment variables
│   ├── .env.stg          # Staging environment variables
│   └── .env.pro          # Production environment variables
├── tests/                # Test files
├── package.json
├── serverless.yml        # Serverless configuration
└── tsconfig.json         # TypeScript configuration
```

## Architecture Patterns

### Three-Layer Architecture

1. **Controller Layer** (`app/controller/`):
   - Handles HTTP requests and responses
   - Parses event data and path parameters
   - Calls service layer for business logic
   - Formats responses using MessageUtil

2. **Service Layer** (`app/service/`):
   - Contains business logic
   - Interacts with models/database
   - Validates data
   - Reusable across controllers

3. **Model Layer** (`app/model/`):
   - Defines Mongoose schemas and models
   - Database structure definitions
   - DTOs for data validation
   - Value Objects for domain logic

### Handler Pattern

The `handler.ts` file exports separate Lambda functions for each endpoint:
```typescript
export const create: Handler = (event: any, context: Context) => {
  return booksController.create(event, context);
};

export const find: Handler = () => booksController.find();
```

## Development Guidelines

### Adding New Features

#### 1. Create Model (Database Schema)

```typescript
// app/model/resource.ts
import mongoose from 'mongoose';

export type ResourceDocument = mongoose.Document & {
  name: string;
  id: number;
  createdAt: Date;
};

const resourceSchema = new mongoose.Schema({
  name: String,
  id: { type: Number, index: true, unique: true },
  createdAt: { type: Date, default: Date.now },
});

export const resource = (mongoose.models.resource ||
  mongoose.model<ResourceDocument>('resource', resourceSchema, 'resources')
);
```

#### 2. Create DTOs (Data Transfer Objects)

```typescript
// app/model/dto/createResourceDTO.ts
export class CreateResourceDTO {
  name: string;
  id: number;
}
```

#### 3. Create Service (Business Logic)

```typescript
// app/service/resource.ts
import { Model } from 'mongoose';

export class ResourceService {
  private resource: Model<any>;

  constructor(resource: Model<any>) {
    this.resource = resource;
  }

  async createResource(params: any) {
    const resource = new this.resource(params);
    return await resource.save();
  }

  async findResources() {
    return await this.resource.find({}).exec();
  }

  async findOneById(id: number) {
    return await this.resource.findOne({ id }).exec();
  }

  async updateResource(id: number, data: object) {
    return await this.resource.updateOne({ id }, data).exec();
  }

  async deleteOneById(id: number) {
    return await this.resource.deleteOne({ id }).exec();
  }
}
```

#### 4. Create Controller (Request Handler)

```typescript
// app/controller/resource.ts
import { Context } from 'aws-lambda';
import { Model } from 'mongoose';
import { MessageUtil } from '../utils/message';
import { ResourceService } from '../service/resource';

export class ResourceController extends ResourceService {
  constructor(resource: Model<any>) {
    super(resource);
  }

  async create(event: any, context?: Context) {
    const params = JSON.parse(event.body);

    try {
      const result = await this.createResource(params);
      return MessageUtil.success(result);
    } catch (err) {
      console.error(err);
      return MessageUtil.error(err.code, err.message);
    }
  }

  async find() {
    try {
      const result = await this.findResources();
      return MessageUtil.success(result);
    } catch (err) {
      console.error(err);
      return MessageUtil.error(err.code, err.message);
    }
  }

  // Add other methods (findOne, update, deleteOne)
}
```

#### 5. Export Handlers

```typescript
// app/handler.ts
import { resource } from './model';
import { ResourceController } from './controller/resource';

const resourceController = new ResourceController(resource);

export const createResource: Handler = (event: any, context: Context) => {
  return resourceController.create(event, context);
};

export const findResources: Handler = () => resourceController.find();
```

#### 6. Configure Serverless Functions

```yaml
# serverless.yml
functions:
  createResource:
    handler: app/handler.createResource
    events:
      - http:
          path: resources
          method: post
  findResources:
    handler: app/handler.findResources
    events:
      - http:
          path: resources
          method: get
```

### TypeScript Best Practices

1. **Strong Typing**:
   ```typescript
   // Define types for all function parameters and returns
   async findOneById(id: number): Promise<ResourceDocument | null> {
     return await this.resource.findOne({ id }).exec();
   }
   ```

2. **Interface for DTOs**:
   ```typescript
   interface CreateResourceDTO {
     name: string;
     id: number;
     description?: string; // Optional field
   }
   ```

3. **Type Guards**:
   ```typescript
   function isValidResource(obj: any): obj is CreateResourceDTO {
     return obj && typeof obj.name === 'string' && typeof obj.id === 'number';
   }
   ```

4. **Enums for Constants**:
   ```typescript
   enum ResourceStatus {
     ACTIVE = 'active',
     INACTIVE = 'inactive',
     DELETED = 'deleted'
   }
   ```

## Commands

### Development
```bash
npm install              # Install dependencies
npm run local            # Start serverless offline
npm run lint             # Run TSLint
```

### Testing
```bash
npm test                 # Run unit tests
npm run coverage         # Run tests with coverage report
```

### Deployment
```bash
npm run deploy           # Deploy to AWS (dev environment)
serverless deploy        # Alternative deployment command
serverless invoke local --function find  # Test function locally
```

### Logs
```bash
serverless logs -f create -t    # Tail logs for create function
```

## Environment Configuration

### Environment Files Structure

- `config/.env.dev` - Development settings
- `config/.env.stg` - Staging settings  
- `config/.env.pro` - Production settings

### Environment Variables

```bash
# config/.env.dev
NODE_ENV=dev
MONGODB_URI=mongodb+srv://user:password@cluster.mongodb.net/dbname
DB_BOOKS_COLLECTION=books
```

### Loading Environment Variables

The handler loads environment-specific config:
```typescript
import dotenv from 'dotenv';
import path from 'path';

const dotenvPath = path.join(__dirname, '../', `config/.env.${process.env.NODE_ENV}`);
dotenv.config({ path: dotenvPath });
```

## MongoDB Integration

### Connection Management

Mongoose connects automatically when models are used. Configure connection in environment:

```typescript
// Connection is established when model is first accessed
const result = await this.books.find({}).exec();
```

### Schema Design Best Practices

1. **Indexes**: Add indexes for frequently queried fields
   ```typescript
   id: { type: Number, index: true, unique: true }
   ```

2. **Timestamps**: Use default values for audit fields
   ```typescript
   createdAt: { type: Date, default: Date.now }
   ```

3. **Validation**: Add Mongoose validators
   ```typescript
   name: { type: String, required: true, minlength: 3 }
   ```

### Model Reuse Prevention

Prevent OverwriteModelError:
```typescript
export const books = (mongoose.models.books ||
  mongoose.model<BooksDocument>('books', booksSchema, 'collection_name')
);
```

## Testing Strategy

### Unit Tests with Mocha and Chai

```typescript
// tests/handler.test.ts
import { expect } from 'chai';
import lambdaTester from 'lambda-tester';
import { create, find } from '../app/handler';

describe('Books API', () => {
  it('should create a book', () => {
    return lambdaTester(create)
      .event({
        body: JSON.stringify({ name: 'Test Book', id: 1 })
      })
      .expectResult((result: any) => {
        expect(result.statusCode).to.equal(200);
        const body = JSON.parse(result.body);
        expect(body.code).to.equal(0);
      });
  });

  it('should find all books', () => {
    return lambdaTester(find)
      .expectResult((result: any) => {
        expect(result.statusCode).to.equal(200);
      });
  });
});
```

### Integration Tests

Test with actual MongoDB:
1. Use test database
2. Seed test data
3. Run tests
4. Clean up test data

### Test Coverage

```bash
npm run coverage  # Generates coverage report with nyc
```

## Error Handling

### Standard Error Response

```typescript
// app/utils/message.ts
export class MessageUtil {
  static success(data: any) {
    return {
      statusCode: 200,
      body: JSON.stringify({
        code: 0,
        message: 'success',
        data
      })
    };
  }

  static error(code: number, message: string) {
    return {
      statusCode: code >= 1000 ? 200 : code,
      body: JSON.stringify({
        code,
        message,
        data: null
      })
    };
  }
}
```

### Error Codes Convention

- `0`: Success
- `1xxx`: Business logic errors (e.g., 1010 - Not found)
- `4xx`: Client errors
- `5xx`: Server errors

## RESTful API Design

### CRUD Endpoints

```
POST   /books          - Create a book
GET    /books          - Get all books
GET    /books/{id}     - Get one book by id
PUT    /books/{id}     - Update a book by id
DELETE /books/{id}     - Delete a book by id
```

### Path Parameters

Access in handler:
```typescript
const id: number = Number(event.pathParameters.id);
```

### Request Body

Parse JSON body:
```typescript
const params = JSON.parse(event.body);
```

### Response Format

Consistent JSON structure:
```json
{
  "code": 0,
  "message": "success",
  "data": { ... }
}
```

## Performance Optimization

### MongoDB Query Optimization

1. Use indexes for frequently queried fields
2. Limit returned fields: `.select('name id')`
3. Use lean queries for read-only: `.lean()`
4. Implement pagination for large datasets

### Lambda Optimization

1. Reuse MongoDB connections (handled by Mongoose)
2. Minimize dependencies in package.json
3. Use `package.exclude` in serverless.yml
4. Individual packaging per function

## Security Best Practices

### 1. Environment Variables
- Never commit `.env` files
- Use AWS Secrets Manager for sensitive data
- Rotate MongoDB credentials regularly

### 2. Input Validation
```typescript
// Validate input before processing
if (!params.name || !params.id) {
  return MessageUtil.error(400, 'Missing required fields');
}

if (typeof params.id !== 'number') {
  return MessageUtil.error(400, 'ID must be a number');
}
```

### 3. MongoDB Security
- Use MongoDB Atlas IP whitelisting
- Enable authentication
- Use connection string encryption
- Implement rate limiting

### 4. API Security
- Add API Gateway authentication
- Implement CORS properly
- Use API keys for public APIs
- Enable AWS WAF for DDoS protection

## Debugging

### Local Debugging

1. **Serverless Offline**:
   ```bash
   npm run local
   curl -X POST http://localhost:3000/books -d '{"name":"Test","id":1}'
   ```

2. **Lambda Local Invocation**:
   ```bash
   serverless invoke local --function find
   ```

3. **TypeScript Debugging**:
   - Use source maps
   - Add breakpoints in IDE
   - Use `console.log` for quick debugging

### Production Debugging

- CloudWatch Logs: `serverless logs -f functionName -t`
- X-Ray for distributed tracing
- Monitor Lambda metrics (invocations, errors, duration)

## Common Tasks

### Updating Dependencies
```bash
npm update                    # Update all packages
npm outdated                  # Check for outdated packages
npm audit                     # Check for vulnerabilities
npm audit fix                 # Fix vulnerabilities
```

### Adding New Database Collection
1. Create model in `app/model/`
2. Add collection name to environment config
3. Create service and controller
4. Export handlers
5. Add serverless functions

### Multi-Environment Deployment
```bash
# Deploy to staging
NODE_ENV=stg serverless deploy --stage stg

# Deploy to production
NODE_ENV=pro serverless deploy --stage pro
```

## Conventions

- **File Naming**: Use camelCase for TypeScript files (e.g., `booksController.ts`)
- **Class Naming**: Use PascalCase (e.g., `BooksController`)
- **Method Naming**: Use camelCase (e.g., `findOneById`)
- **Constants**: Use UPPER_SNAKE_CASE
- **DTOs**: Suffix with `DTO` (e.g., `CreateBookDTO`)

## Linting Configuration

TSLint with Airbnb config is configured. Run:
```bash
npm run lint
```

Pre-commit hook runs linting automatically.

## Resources

- [TypeScript Documentation](https://www.typescriptlang.org/docs/)
- [Mongoose Documentation](https://mongoosejs.com/docs/)
- [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)
- [Serverless Framework](https://www.serverless.com/framework/docs/)
- [AWS Lambda Best Practices](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html)
- [Mocha Testing Framework](https://mochajs.org/)
