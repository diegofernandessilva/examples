# Agent Instructions for NestJS Backend Development

## Project Overview

This is a serverless NestJS backend application deployed to AWS Lambda using the Serverless Framework. It demonstrates how to build a modern, scalable backend API with TypeScript and NestJS.

## Technology Stack

- **Framework**: NestJS 5.x
- **Runtime**: Node.js 12.x
- **Language**: TypeScript 3.x
- **Deployment**: AWS Lambda with Serverless Framework
- **Testing**: Jest
- **Linting**: TSLint

## Project Structure

```
src/
├── app.controller.ts      # Application controllers (route handlers)
├── app.controller.spec.ts # Controller unit tests
├── app.service.ts         # Business logic services
├── app.module.ts          # Root application module
└── main.ts               # Lambda handler entry point
```

## Architecture Patterns

### NestJS Module System
- Use `@Module()` decorator to organize application into cohesive blocks
- Modules should group related controllers, services, and providers
- Follow dependency injection pattern using constructor injection

### Controllers
- Use `@Controller()` decorator to define route handlers
- Decorate methods with HTTP verb decorators: `@Get()`, `@Post()`, `@Put()`, `@Delete()`, etc.
- Keep controllers thin - delegate business logic to services

### Services
- Use `@Injectable()` decorator for service classes
- Services contain business logic and data access
- Services are singleton by default within their module scope

### AWS Lambda Integration
- The `main.ts` file bootstraps the NestJS app for Lambda
- Uses `aws-serverless-express` to adapt Express/NestJS to Lambda
- Implements server caching to improve cold start performance

## Development Guidelines

### Adding New Features

1. **Create a new module** (for larger features):
   ```typescript
   @Module({
     imports: [],
     controllers: [FeatureController],
     providers: [FeatureService],
   })
   export class FeatureModule {}
   ```

2. **Create a controller**:
   ```typescript
   @Controller('api/resource')
   export class ResourceController {
     constructor(private readonly resourceService: ResourceService) {}
     
     @Get()
     findAll() {
       return this.resourceService.findAll();
     }
   }
   ```

3. **Create a service**:
   ```typescript
   @Injectable()
   export class ResourceService {
     findAll() {
       // Business logic here
     }
   }
   ```

4. **Write tests**:
   - Create `.spec.ts` files alongside implementation files
   - Test controllers and services independently
   - Use NestJS testing utilities

### TypeScript Best Practices

- Enable strict type checking
- Use interfaces for DTOs (Data Transfer Objects)
- Use type annotations for function parameters and return types
- Avoid using `any` type - use `unknown` if type is truly unknown

### Dependency Management

- Use npm for package management
- Pin major versions in `package.json`
- Test thoroughly after updating dependencies

## Commands

### Development
```bash
npm start                 # Start serverless offline (local development)
npm run build            # Compile TypeScript
npm run format           # Format code with Prettier
```

### Testing
```bash
npm test                 # Run unit tests
npm run test:watch       # Run tests in watch mode
npm run test:cov         # Run tests with coverage
npm run test:e2e         # Run end-to-end tests
```

### Code Quality
```bash
npm run lint             # Run TSLint
```

### Deployment
```bash
sls deploy               # Deploy to AWS
sls logs --function main --tail  # View logs
```

## Serverless Configuration

The `serverless.yml` defines:
- Service name: `serverless-nest-example`
- Runtime: Node.js 12.x
- Functions: Single `main` function handling all HTTP routes via proxy
- Plugins:
  - `@hewmen/serverless-plugin-typescript` - TypeScript compilation
  - `serverless-plugin-optimize` - Code optimization
  - `serverless-offline` - Local development server

### HTTP Routes
All routes are handled by a single Lambda function using catch-all path:
```yaml
events:
  - http:
      method: any
      path: /{proxy+}
```

## Testing Strategy

### Unit Tests
- Test each service method independently
- Mock dependencies using Jest
- Test happy paths and error conditions

### Integration Tests
- Test controller endpoints
- Verify request/response handling
- Test with TestingModule from `@nestjs/testing`

Example test structure:
```typescript
describe('AppController', () => {
  let controller: AppController;
  let service: AppService;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      controllers: [AppController],
      providers: [AppService],
    }).compile();

    controller = module.get<AppController>(AppController);
    service = module.get<AppService>(AppService);
  });

  it('should return "Hello World!"', () => {
    expect(controller.getHello()).toBe('Hello World!');
  });
});
```

## Performance Considerations

### Cold Starts
- Lambda caches the server instance to reduce cold starts
- Consider using serverless-plugin-warmup for production
- Keep dependencies minimal

### Lambda Optimizations
- Package functions individually (`individually: true`)
- Use serverless-plugin-optimize to minimize bundle size
- Implement proper error handling to prevent memory leaks

## Security Best Practices

1. **Input Validation**: Use NestJS pipes for validation
2. **Environment Variables**: Store secrets in environment variables, not in code
3. **CORS**: Configure CORS appropriately for your API
4. **Error Handling**: Don't expose sensitive information in error messages
5. **Dependencies**: Regularly update dependencies to patch security vulnerabilities

## Common Tasks

### Adding a New Endpoint
1. Add method to controller with appropriate decorator
2. Implement business logic in service
3. Add tests for both controller and service
4. Deploy and test

### Adding Database Integration
1. Install appropriate driver (e.g., TypeORM, Mongoose)
2. Create a database module
3. Configure connection in module imports
4. Create entities/models
5. Create repository services
6. Inject repositories into services

### Adding Environment Configuration
1. Install `@nestjs/config`
2. Import ConfigModule in AppModule
3. Create `.env` file for local development
4. Add environment variables to serverless.yml for deployment
5. Use ConfigService to access variables

## Debugging

### Local Debugging
- Use serverless-offline for local testing
- Set `NODE_ENV=development` for verbose logging
- Use `--inspect` flag with nodemon for debugging

### AWS Debugging
- View logs: `sls logs --function main --tail`
- Enable X-Ray tracing for performance analysis
- Use CloudWatch Logs for error investigation

## Conventions

- **File Naming**: Use kebab-case for file names (e.g., `user-service.ts`)
- **Class Naming**: Use PascalCase for classes (e.g., `UserService`)
- **Method Naming**: Use camelCase for methods (e.g., `getUserById`)
- **Constant Naming**: Use UPPER_SNAKE_CASE for constants

## Resources

- [NestJS Documentation](https://docs.nestjs.com/)
- [Serverless Framework Documentation](https://www.serverless.com/framework/docs/)
- [AWS Lambda Best Practices](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
