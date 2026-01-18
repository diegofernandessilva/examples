# Agent Instructions for REST API with DynamoDB Backend

## Project Overview

This is a serverless REST API backend built with Node.js and DynamoDB, deployed to AWS Lambda using the Serverless Framework. It demonstrates a complete CRUD (Create, Read, Update, Delete) application for managing todos, showcasing best practices for serverless architecture with NoSQL databases.

## Technology Stack

- **Language**: JavaScript (Node.js)
- **Runtime**: Node.js 10.x on AWS Lambda
- **Database**: AWS DynamoDB (NoSQL)
- **Framework**: Serverless Framework
- **Dependencies**: uuid (for ID generation), AWS SDK (provided by Lambda)

## Project Structure

```
├── todos/                    # Directory for all todo operations
│   ├── create.js            # Create todo handler
│   ├── list.js              # List all todos handler
│   ├── get.js               # Get single todo handler
│   ├── update.js            # Update todo handler
│   └── delete.js            # Delete todo handler
├── package.json
└── serverless.yml           # Serverless configuration with DynamoDB table
```

## Architecture Pattern

### Microservices Pattern
- Each CRUD operation is a separate Lambda function
- Each function in its own file under `todos/` directory
- Direct module exports for Lambda handlers
- Shared DynamoDB table across all functions

### Function Structure
Each handler file follows this pattern:
```javascript
'use strict';
const AWS = require('aws-sdk');
const dynamoDb = new AWS.DynamoDB.DocumentClient();

module.exports.operationName = (event, context, callback) => {
  // Implementation
};
```

## Development Guidelines

### Adding New Endpoints

#### 1. Create Handler File

```javascript
// todos/operation.js
'use strict';

const AWS = require('aws-sdk');
const dynamoDb = new AWS.DynamoDB.DocumentClient();

module.exports.operation = (event, context, callback) => {
  // Parse input
  const data = JSON.parse(event.body);
  
  // Validate input
  if (!data.field) {
    callback(null, {
      statusCode: 400,
      headers: { 'Content-Type': 'text/plain' },
      body: 'Validation error message',
    });
    return;
  }
  
  // DynamoDB operation
  const params = {
    TableName: process.env.DYNAMODB_TABLE,
    // Operation-specific params
  };
  
  dynamoDb.operation(params, (error, result) => {
    if (error) {
      console.error(error);
      callback(null, {
        statusCode: error.statusCode || 501,
        headers: { 'Content-Type': 'text/plain' },
        body: 'Error message',
      });
      return;
    }
    
    callback(null, {
      statusCode: 200,
      body: JSON.stringify(result),
    });
  });
};
```

#### 2. Add Function to serverless.yml

```yaml
functions:
  operation:
    handler: todos/operation.operation
    events:
      - http:
          path: todos
          method: post
          cors: true
```

### DynamoDB Operations

#### Create Item (Put)
```javascript
const params = {
  TableName: process.env.DYNAMODB_TABLE,
  Item: {
    id: uuid.v1(),
    text: data.text,
    checked: false,
    createdAt: timestamp,
    updatedAt: timestamp,
  },
};

dynamoDb.put(params, (error) => {
  // Handle response
});
```

#### Get Item
```javascript
const params = {
  TableName: process.env.DYNAMODB_TABLE,
  Key: {
    id: event.pathParameters.id,
  },
};

dynamoDb.get(params, (error, result) => {
  // Handle response
});
```

#### Scan (List All)
```javascript
const params = {
  TableName: process.env.DYNAMODB_TABLE,
};

dynamoDb.scan(params, (error, result) => {
  // result.Items contains array of items
});
```

#### Update Item
```javascript
const params = {
  TableName: process.env.DYNAMODB_TABLE,
  Key: {
    id: event.pathParameters.id,
  },
  ExpressionAttributeNames: {
    '#todo_text': 'text',
  },
  ExpressionAttributeValues: {
    ':text': data.text,
    ':checked': data.checked,
    ':updatedAt': timestamp,
  },
  UpdateExpression: 'SET #todo_text = :text, checked = :checked, updatedAt = :updatedAt',
  ReturnValues: 'ALL_NEW',
};

dynamoDb.update(params, (error, result) => {
  // result.Attributes contains updated item
});
```

#### Delete Item
```javascript
const params = {
  TableName: process.env.DYNAMODB_TABLE,
  Key: {
    id: event.pathParameters.id,
  },
};

dynamoDb.delete(params, (error) => {
  // Handle response
});
```

## DynamoDB Best Practices

### Table Design

1. **Primary Key**: Use UUID for unique identifiers
2. **Timestamps**: Store createdAt and updatedAt
3. **Attributes**: Keep data denormalized for read efficiency

### Provisioned Throughput

Configure in `serverless.yml`:
```yaml
ProvisionedThroughput:
  ReadCapacityUnits: 1
  WriteCapacityUnits: 1
```

For production, consider:
- Auto-scaling for variable traffic
- On-demand pricing for unpredictable workloads
- DynamoDB Accelerator (DAX) for caching

### Query Optimization

1. **Scan vs Query**:
   - Use Query with partition key for better performance
   - Scan reads entire table (expensive)
   - Consider Global Secondary Indexes (GSI) for complex queries

2. **Pagination**:
   ```javascript
   const params = {
     TableName: process.env.DYNAMODB_TABLE,
     Limit: 10,
     ExclusiveStartKey: lastEvaluatedKey, // From previous response
   };
   ```

3. **Filtering**:
   ```javascript
   const params = {
     TableName: process.env.DYNAMODB_TABLE,
     FilterExpression: 'checked = :checked',
     ExpressionAttributeValues: {
       ':checked': true,
     },
   };
   ```

## Commands

### Development
```bash
npm install              # Install dependencies
```

### Deployment
```bash
serverless deploy        # Deploy to AWS
serverless remove        # Remove deployed service
```

### Testing
```bash
# Create a todo
curl -X POST https://XXXXXXX.execute-api.us-east-1.amazonaws.com/dev/todos \
  --data '{ "text": "Learn Serverless" }'

# List all todos
curl https://XXXXXXX.execute-api.us-east-1.amazonaws.com/dev/todos

# Get one todo
curl https://XXXXXXX.execute-api.us-east-1.amazonaws.com/dev/todos/{id}

# Update a todo
curl -X PUT https://XXXXXXX.execute-api.us-east-1.amazonaws.com/dev/todos/{id} \
  --data '{ "text": "Learn Serverless", "checked": true }'

# Delete a todo
curl -X DELETE https://XXXXXXX.execute-api.us-east-1.amazonaws.com/dev/todos/{id}
```

### Logs
```bash
serverless logs -f create -t    # Tail create function logs
serverless logs -f list         # View list function logs
```

## Input Validation

### Basic Validation Pattern
```javascript
const data = JSON.parse(event.body);

// Validate required fields
if (typeof data.text !== 'string') {
  console.error('Validation Failed');
  callback(null, {
    statusCode: 400,
    headers: { 'Content-Type': 'text/plain' },
    body: 'Couldn\'t create the todo item.',
  });
  return;
}
```

### Advanced Validation
```javascript
// Validate multiple fields
const errors = [];

if (!data.text || typeof data.text !== 'string') {
  errors.push('text is required and must be a string');
}

if (data.text && data.text.length > 200) {
  errors.push('text must be less than 200 characters');
}

if (errors.length > 0) {
  callback(null, {
    statusCode: 400,
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ errors }),
  });
  return;
}
```

## Error Handling

### Standard Error Response Pattern
```javascript
// DynamoDB error handling
dynamoDb.operation(params, (error, result) => {
  if (error) {
    console.error(error);
    callback(null, {
      statusCode: error.statusCode || 501,
      headers: { 'Content-Type': 'text/plain' },
      body: 'Operation failed.',
    });
    return;
  }
  
  // Success handling
  callback(null, {
    statusCode: 200,
    body: JSON.stringify(result),
  });
});
```

### Item Not Found
```javascript
dynamoDb.get(params, (error, result) => {
  if (error) {
    // Handle error
  }
  
  if (!result.Item) {
    callback(null, {
      statusCode: 404,
      headers: { 'Content-Type': 'text/plain' },
      body: 'Todo not found.',
    });
    return;
  }
  
  // Return item
  callback(null, {
    statusCode: 200,
    body: JSON.stringify(result.Item),
  });
});
```

## Environment Variables

### Accessing Environment Variables
```javascript
const tableName = process.env.DYNAMODB_TABLE;
```

### Configuring in serverless.yml
```yaml
provider:
  environment:
    DYNAMODB_TABLE: ${self:service}-${opt:stage, self:provider.stage}
    API_VERSION: v1
```

## IAM Permissions

The `serverless.yml` configures necessary DynamoDB permissions:
```yaml
iamRoleStatements:
  - Effect: Allow
    Action:
      - dynamodb:Query
      - dynamodb:Scan
      - dynamodb:GetItem
      - dynamodb:PutItem
      - dynamodb:UpdateItem
      - dynamodb:DeleteItem
    Resource: "arn:aws:dynamodb:${opt:region, self:provider.region}:*:table/${self:provider.environment.DYNAMODB_TABLE}"
```

### Principle of Least Privilege
Only grant permissions needed:
- Read operations: Query, Scan, GetItem
- Write operations: PutItem, UpdateItem, DeleteItem

## CORS Configuration

Enable CORS for frontend integration:
```yaml
events:
  - http:
      path: todos
      method: post
      cors: true
```

Custom CORS configuration:
```yaml
cors:
  origin: 'https://yourdomain.com'
  headers:
    - Content-Type
    - X-Api-Key
  allowCredentials: true
```

## Response Format Best Practices

### Success Response
```javascript
callback(null, {
  statusCode: 200,
  headers: {
    'Content-Type': 'application/json',
    'Access-Control-Allow-Origin': '*',
  },
  body: JSON.stringify({
    message: 'Success',
    data: result,
  }),
});
```

### Error Response
```javascript
callback(null, {
  statusCode: 400,
  headers: {
    'Content-Type': 'application/json',
    'Access-Control-Allow-Origin': '*',
  },
  body: JSON.stringify({
    error: 'ValidationError',
    message: 'Invalid input',
  }),
});
```

## Scaling Considerations

### AWS Lambda
- Default: 100 concurrent executions per region
- Request limit increase for production workloads
- Monitor throttling in CloudWatch

### DynamoDB
- **Provisioned Mode**: Set Read/Write Capacity Units
- **On-Demand Mode**: Pay per request (good for unpredictable traffic)
- **Auto-scaling**: Automatically adjust capacity based on load

Example auto-scaling configuration:
```yaml
resources:
  Resources:
    TodosDynamoDbTable:
      Type: 'AWS::DynamoDB::Table'
      Properties:
        BillingMode: PAY_PER_REQUEST  # On-demand
        # OR
        BillingMode: PROVISIONED
        ProvisionedThroughput:
          ReadCapacityUnits: 5
          WriteCapacityUnits: 5
```

## Testing Strategy

### Local Testing
Use `serverless-offline` and `serverless-dynamodb-local`:
```bash
# Install plugins
npm install --save-dev serverless-offline serverless-dynamodb-local

# Add to serverless.yml
plugins:
  - serverless-dynamodb-local
  - serverless-offline

# Start local DynamoDB
serverless dynamodb install
serverless dynamodb start

# Start offline
serverless offline
```

### Integration Testing
```javascript
const AWS = require('aws-sdk');
const { expect } = require('chai');

describe('Todos API', () => {
  before(() => {
    AWS.config.update({ region: 'us-east-1' });
  });

  it('should create a todo', async () => {
    // Test implementation
  });
});
```

## Monitoring and Debugging

### CloudWatch Logs
- Each function logs to CloudWatch Logs
- Use `console.log()` and `console.error()` for logging
- View logs: `serverless logs -f functionName -t`

### CloudWatch Metrics
Monitor:
- Invocation count
- Error rate
- Duration
- Throttles

### DynamoDB Metrics
Monitor:
- Read/Write capacity usage
- Throttled requests
- System errors

### X-Ray Tracing
Enable distributed tracing:
```yaml
provider:
  tracing:
    lambda: true
```

## Security Best Practices

### 1. Input Validation
Always validate and sanitize input:
```javascript
// Sanitize text input
const sanitizeText = (text) => {
  return text.trim().substring(0, 200);
};

const data = JSON.parse(event.body);
const sanitizedText = sanitizeText(data.text);
```

### 2. IAM Policies
- Use least privilege principle
- Scope permissions to specific tables
- Avoid wildcard (*) in production

### 3. Environment Variables
- Store sensitive data in AWS Secrets Manager
- Use parameter store for configuration
- Never hardcode credentials

### 4. API Gateway Security
```yaml
events:
  - http:
      path: todos
      method: post
      cors: true
      authorizer: aws_iam  # Require AWS signature
```

### 5. DynamoDB Encryption
Enable encryption at rest:
```yaml
TodosDynamoDbTable:
  Properties:
    SSESpecification:
      SSEEnabled: true
```

## Common Patterns

### UUID Generation
```javascript
const uuid = require('uuid');
const id = uuid.v1();  // Time-based
// or
const id = uuid.v4();  // Random
```

### Timestamp Management
```javascript
const timestamp = new Date().getTime();
// or
const timestamp = Date.now();
```

### Path Parameters
```javascript
const id = event.pathParameters.id;
```

### Query String Parameters
```javascript
const limit = event.queryStringParameters?.limit || 10;
```

## Advanced Features

### Batch Operations
```javascript
// Batch write
const params = {
  RequestItems: {
    [process.env.DYNAMODB_TABLE]: [
      { PutRequest: { Item: item1 } },
      { PutRequest: { Item: item2 } },
    ],
  },
};

dynamoDb.batchWrite(params, callback);
```

### Transactions
```javascript
const params = {
  TransactItems: [
    {
      Put: {
        TableName: process.env.DYNAMODB_TABLE,
        Item: item,
      },
    },
  ],
};

dynamoDb.transactWrite(params, callback);
```

### Conditional Updates
```javascript
const params = {
  TableName: process.env.DYNAMODB_TABLE,
  Key: { id },
  UpdateExpression: 'SET #text = :text',
  ConditionExpression: 'attribute_exists(id)',
  ExpressionAttributeNames: { '#text': 'text' },
  ExpressionAttributeValues: { ':text': newText },
};
```

## Multi-Resource Services

To add more resources (e.g., users, notes):
```
├── todos/
│   ├── create.js
│   └── ...
├── users/
│   ├── create.js
│   └── ...
└── notes/
    ├── create.js
    └── ...
```

Each resource gets its own DynamoDB table in `serverless.yml`.

## Conventions

- **File Naming**: Use kebab-case or lowercase (e.g., `create.js`)
- **Function Naming**: Use descriptive names (e.g., `createTodo`)
- **HTTP Status Codes**:
  - 200: Success
  - 201: Created
  - 400: Bad Request
  - 404: Not Found
  - 500/501: Server Error

## Resources

- [AWS DynamoDB Documentation](https://docs.aws.amazon.com/dynamodb/)
- [DynamoDB Best Practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html)
- [Serverless Framework Documentation](https://www.serverless.com/framework/docs/)
- [AWS Lambda Best Practices](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html)
- [DynamoDB JavaScript SDK](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB/DocumentClient.html)
