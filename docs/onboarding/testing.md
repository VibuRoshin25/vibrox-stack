# Testing Guide

## Overview

This guide covers testing strategies, best practices, and tools for the Vibrox Stack. Comprehensive testing ensures code quality, reliability, and maintainability.

## Testing Strategy

### Testing Pyramid

```
    /\
   /  \     E2E Tests (Few)
  /____\    Integration Tests (Some)
 /______\   Unit Tests (Many)
```

### Test Types

1. **Unit Tests**: Test individual functions and components
2. **Integration Tests**: Test service interactions
3. **End-to-End Tests**: Test complete user workflows
4. **Performance Tests**: Test system performance under load

## Go Testing

### Unit Tests

```go
// user_service_test.go
package services

import (
    "testing"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/mock"
)

func TestUserService_CreateUser(t *testing.T) {
    // Arrange
    mockDB := &MockDatabase{}
    service := NewUserService(mockDB)
    
    user := &models.User{
        Name:  "John Doe",
        Email: "john@example.com",
    }
    
    mockDB.On("Create", user).Return(nil)
    
    // Act
    result, err := service.CreateUser(user)
    
    // Assert
    assert.NoError(t, err)
    assert.NotNil(t, result)
    assert.Equal(t, user.Name, result.Name)
    mockDB.AssertExpectations(t)
}
```

### Integration Tests

```go
// integration_test.go
package tests

import (
    "testing"
    "github.com/stretchr/testify/suite"
)

type IntegrationTestSuite struct {
    suite.Suite
    db *sql.DB
}

func (suite *IntegrationTestSuite) SetupSuite() {
    suite.db = setupTestDatabase()
}

func (suite *IntegrationTestSuite) TearDownSuite() {
    suite.db.Close()
}

func (suite *IntegrationTestSuite) TestUserWorkflow() {
    // Test complete user workflow
    user := createTestUser(suite.db)
    token := authenticateUser(suite.T(), user)
    
    // Use token to access protected endpoint
    response := makeAuthenticatedRequest(token, "/api/users")
    suite.Equal(200, response.StatusCode)
}

func TestIntegrationSuite(t *testing.T) {
    suite.Run(t, new(IntegrationTestSuite))
}
```

### Running Go Tests

```bash
# Run all tests
go test ./...

# Run tests with coverage
go test -cover ./...

# Run tests with verbose output
go test -v ./...

# Run specific test
go test -run TestUserService_CreateUser

# Run tests with race detection
go test -race ./...

# Generate coverage report
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
```

## Node.js Testing

### Unit Tests

```javascript
// auth.service.test.js
const AuthService = require('../src/services/auth.service');
const { expect } = require('chai');
const sinon = require('sinon');

describe('AuthService', () => {
    let authService;
    let mockDB;

    beforeEach(() => {
        mockDB = {
            query: sinon.stub()
        };
        authService = new AuthService(mockDB);
    });

    describe('authenticate', () => {
        it('should authenticate valid user', async () => {
            // Arrange
            const username = 'testuser';
            const password = 'password123';
            const mockUser = { 
                id: 1, 
                username, 
                password_hash: 'hashed_password' 
            };
            
            mockDB.query.resolves({ rows: [mockUser] });
            
            // Act
            const result = await authService.authenticate(username, password);
            
            // Assert
            expect(result.success).to.be.true;
            expect(result.token).to.not.be.empty;
        });

        it('should reject invalid credentials', async () => {
            // Arrange
            mockDB.query.resolves({ rows: [] });
            
            // Act & Assert
            await expect(
                authService.authenticate('invalid', 'wrong')
            ).to.be.rejectedWith('Invalid credentials');
        });
    });
});
```

### Integration Tests

```javascript
// integration.test.js
const request = require('supertest');
const app = require('../src/app');
const { expect } = require('chai');

describe('Authentication API', () => {
    describe('POST /auth/login', () => {
        it('should authenticate user and return token', async () => {
            const response = await request(app)
                .post('/auth/login')
                .send({
                    username: 'testuser',
                    password: 'password123'
                })
                .expect(200);

            expect(response.body).to.have.property('token');
            expect(response.body).to.have.property('user');
        });

        it('should reject invalid credentials', async () => {
            await request(app)
                .post('/auth/login')
                .send({
                    username: 'testuser',
                    password: 'wrongpassword'
                })
                .expect(401);
        });
    });
});
```

### Running Node.js Tests

```bash
# Run all tests
npm test

# Run tests with coverage
npm run test:coverage

# Run tests in watch mode
npm run test:watch

# Run specific test file
npm test -- --grep "AuthService"

# Run integration tests
npm run test:integration
```

## Database Testing

### Test Database Setup

```go
// test_helpers.go
package tests

import (
    "database/sql"
    "testing"
    _ "github.com/lib/pq"
)

func setupTestDatabase(t *testing.T) *sql.DB {
    // Use test database
    db, err := sql.Open("postgres", "postgres://test:test@localhost:5432/vibrox_test?sslmode=disable")
    if err != nil {
        t.Fatal(err)
    }
    
    // Run migrations
    runMigrations(db)
    
    return db
}

func cleanupTestDatabase(t *testing.T, db *sql.DB) {
    // Clean up test data
    tables := []string{"users", "user_auth", "audit_logs"}
    for _, table := range tables {
        db.Exec("TRUNCATE TABLE " + table + " CASCADE")
    }
}
```

### Database Test Examples

```go
func TestUserRepository_Create(t *testing.T) {
    db := setupTestDatabase(t)
    defer cleanupTestDatabase(t, db)
    
    repo := NewUserRepository(db)
    
    user := &models.User{
        Name:  "Test User",
        Email: "test@example.com",
    }
    
    result, err := repo.Create(user)
    
    assert.NoError(t, err)
    assert.NotZero(t, result.ID)
    assert.Equal(t, user.Name, result.Name)
}
```

## API Testing

### REST API Tests

```go
// api_test.go
package tests

import (
    "net/http"
    "net/http/httptest"
    "testing"
    "encoding/json"
    "github.com/gin-gonic/gin"
)

func TestUserAPI_CreateUser(t *testing.T) {
    // Setup
    gin.SetMode(gin.TestMode)
    router := setupTestRouter()
    
    // Test data
    userData := map[string]interface{}{
        "name":  "John Doe",
        "email": "john@example.com",
    }
    
    // Create request
    body, _ := json.Marshal(userData)
    req := httptest.NewRequest("POST", "/api/users", bytes.NewBuffer(body))
    req.Header.Set("Content-Type", "application/json")
    
    // Record response
    w := httptest.NewRecorder()
    router.ServeHTTP(w, req)
    
    // Assertions
    assert.Equal(t, http.StatusCreated, w.Code)
    
    var response map[string]interface{}
    json.Unmarshal(w.Body.Bytes(), &response)
    assert.Equal(t, userData["name"], response["name"])
}
```

### gRPC API Tests

```go
// grpc_test.go
package tests

import (
    "context"
    "testing"
    pb "path/to/proto"
    "google.golang.org/grpc"
    "google.golang.org/grpc/test/bufconn"
)

func TestAuthService_Authenticate(t *testing.T) {
    // Setup gRPC server
    server := grpc.NewServer()
    pb.RegisterAuthServiceServer(server, &AuthService{})
    
    // Create test connection
    listener := bufconn.Listen(1024 * 1024)
    go server.Serve(listener)
    defer server.Stop()
    
    // Create client
    ctx := context.Background()
    conn, err := grpc.DialContext(ctx, "bufnet", 
        grpc.WithContextDialer(func(context.Context, string) (net.Conn, error) {
            return listener.Dial()
        }),
        grpc.WithInsecure(),
    )
    defer conn.Close()
    
    client := pb.NewAuthServiceClient(conn)
    
    // Test authentication
    resp, err := client.Authenticate(ctx, &pb.AuthRequest{
        Username: "testuser",
        Password: "password123",
    })
    
    assert.NoError(t, err)
    assert.True(t, resp.Success)
    assert.NotEmpty(t, resp.Token)
}
```

## Performance Testing

### Load Testing

```javascript
// load_test.js
const autocannon = require('autocannon');

async function runLoadTest() {
    const result = await autocannon({
        url: 'http://localhost:8080/api/users',
        connections: 10,
        duration: 10,
        headers: {
            'Authorization': 'Bearer ' + testToken
        }
    });
    
    console.log(result);
}

runLoadTest();
```

### Benchmark Tests

```go
// benchmark_test.go
package tests

import (
    "testing"
    "github.com/stretchr/testify/assert"
)

func BenchmarkUserService_CreateUser(b *testing.B) {
    service := NewUserService(testDB)
    
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        user := &models.User{
            Name:  "Benchmark User",
            Email: fmt.Sprintf("user%d@example.com", i),
        }
        
        _, err := service.CreateUser(user)
        assert.NoError(b, err)
    }
}
```

## Test Configuration

### Test Environment

```yaml
# docker-compose.test.yml
version: '3.8'
services:
  test-db:
    image: postgres:15
    environment:
      POSTGRES_DB: vibrox_test
      POSTGRES_USER: test
      POSTGRES_PASSWORD: test
    ports:
      - "5433:5432"
  
  test-redis:
    image: redis:7-alpine
    ports:
      - "6380:6379"
```

### Test Configuration Files

```go
// config/test.go
package config

func LoadTestConfig() *Config {
    return &Config{
        Database: DatabaseConfig{
            Host:     "localhost",
            Port:     5433,
            User:     "test",
            Password: "test",
            Name:     "vibrox_test",
        },
        Logging: LoggingConfig{
            Level:  "debug",
            Format: "json",
        },
    }
}
```

## Continuous Integration

### GitHub Actions

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_DB: vibrox_test
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Go
      uses: actions/setup-go@v3
      with:
        go-version: '1.21'
    
    - name: Set up Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
    
    - name: Run Go tests
      run: |
        go test -v -race -coverprofile=coverage.out ./...
        go tool cover -func=coverage.out
    
    - name: Run Node.js tests
      run: |
        cd vibrox-auth
        npm ci
        npm test
```

## Best Practices

### Test Organization

1. **Arrange-Act-Assert**: Structure tests clearly
2. **Test Isolation**: Each test should be independent
3. **Meaningful Names**: Use descriptive test names
4. **Single Responsibility**: Test one thing per test

### Test Data Management

1. **Fixtures**: Use test fixtures for consistent data
2. **Factories**: Create test data factories
3. **Cleanup**: Always clean up test data
4. **Randomization**: Use random data when appropriate

### Mocking

1. **Mock External Dependencies**: Database, APIs, etc.
2. **Use Interfaces**: Design for testability
3. **Verify Interactions**: Ensure mocks are called correctly
4. **Keep Mocks Simple**: Don't over-mock

### Coverage

1. **Aim for 80%+**: Good coverage target
2. **Focus on Critical Paths**: Test important business logic
3. **Don't Chase 100%**: Some code is hard to test
4. **Use Coverage Reports**: Identify untested areas

## Troubleshooting

### Common Issues

#### Test Database Connection

```bash
# Check if test database is running
docker ps | grep postgres

# Reset test database
docker-compose -f docker-compose.test.yml down -v
docker-compose -f docker-compose.test.yml up -d
```

#### Test Environment Variables

```bash
# Set test environment variables
export NODE_ENV=test
export DB_HOST=localhost
export DB_PORT=5433
export DB_NAME=vibrox_test
```

#### Flaky Tests

```bash
# Run tests multiple times to identify flaky tests
for i in {1..10}; do
    go test ./...
done
```

---

*This testing guide should be updated when new testing tools or strategies are adopted.*
