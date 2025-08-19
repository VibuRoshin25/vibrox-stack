# Contributing Guidelines

## Overview

Thank you for your interest in contributing to the Vibrox Stack! This document provides guidelines for contributing to the project.

## Getting Started

### Prerequisites

1. Follow the [Development Setup Guide](development-setup.md)
2. Read the [Architecture Overview](../architecture/overview.md)

### First Steps

```bash
# Fork and clone the repository
git clone https://github.com/YOUR_USERNAME/vibrox-stack.git
cd vibrox-stack

# Add upstream remote
git remote add upstream https://github.com/VibuRoshin25/vibrox-stack.git

# Create feature branch
git checkout -b feature/your-feature-name
```

## Development Workflow

### Branch Naming

Use descriptive branch names:
- `feature/user-authentication`
- `bugfix/database-connection-issue`
- `docs/api-documentation-update`

### Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
type(scope): description

feat(auth): add JWT token refresh endpoint
fix(core): resolve database connection timeout
docs(api): update user management API documentation
```

### Types
- **feat**: New feature
- **fix**: Bug fix
- **docs**: Documentation changes
- **style**: Code style changes
- **refactor**: Code refactoring
- **test**: Adding or updating tests
- **chore**: Maintenance tasks

## Code Standards

### Go Standards

```go
// Format code
go fmt ./...
goimports -w .

// Run linter
golangci-lint run

// Example code style
func (s *UserService) CreateUser(user *models.User) (*models.User, error) {
    if err := s.validateUser(user); err != nil {
        return nil, fmt.Errorf("invalid user data: %w", err)
    }
    
    if err := s.db.Create(user).Error; err != nil {
        return nil, fmt.Errorf("failed to create user: %w", err)
    }
    
    return user, nil
}
```

### Node.js Standards

```javascript
// Format and lint
npm run format
npm run lint

// Example code style
async function authenticateUser(username, password) {
    try {
        const user = await getUserByUsername(username);
        if (!user) {
            throw new Error('User not found');
        }
        
        const isValid = await bcrypt.compare(password, user.passwordHash);
        if (!isValid) {
            throw new Error('Invalid password');
        }
        
        return generateToken(user);
    } catch (error) {
        logger.error('Authentication failed', { username, error });
        throw error;
    }
}
```

## Testing Guidelines

### Requirements

- Unit tests for all new functionality
- Integration tests for service interactions
- Aim for 80%+ code coverage

### Go Testing

```go
func TestUserService_CreateUser(t *testing.T) {
    // Arrange
    service := NewUserService(mockDB)
    user := &models.User{Name: "John", Email: "john@example.com"}
    
    // Act
    result, err := service.CreateUser(user)
    
    // Assert
    assert.NoError(t, err)
    assert.NotNil(t, result)
    assert.Equal(t, user.Name, result.Name)
}
```

### Node.js Testing

```javascript
describe('AuthService', () => {
    it('should authenticate valid user', async () => {
        // Arrange
        const username = 'testuser';
        const password = 'password123';
        
        // Act
        const result = await authService.authenticate(username, password);
        
        // Assert
        expect(result.success).to.be.true;
        expect(result.token).to.not.be.empty;
    });
});
```

## Pull Request Process

### Before Submitting

1. Ensure all tests pass
2. Run linters and formatters
3. Update documentation
4. Self-review your changes

### Pull Request Template

```markdown
## Description
Brief description of changes.

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing completed

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Documentation updated
- [ ] No new warnings generated
- [ ] Tests added/updated
```

### Review Process

1. Automated CI/CD checks
2. Code review by maintainers
3. Address feedback
4. Merge after approval

## Documentation

### Requirements

- Document complex logic and public APIs
- Update service README files
- Update API documentation
- Update architecture docs if needed

### Examples

```go
// UserService handles user-related business logic
type UserService struct {
    db *gorm.DB
}

// CreateUser creates a new user in the system
// Returns the created user or an error if creation fails
func (s *UserService) CreateUser(user *models.User) (*models.User, error) {
    // Implementation
}
```

```javascript
/**
 * Authenticates a user with username and password
 * @param {string} username - The user's username
 * @param {string} password - The user's password
 * @returns {Promise<Object>} Authentication result
 */
async function authenticateUser(username, password) {
    // Implementation
}
```

## Getting Help

### Communication

- **GitHub Issues**: Bug reports and feature requests
- **GitHub Discussions**: Questions and general discussion
- **Pull Requests**: Code reviews and feedback

### Resources

- [Development Setup Guide](development-setup.md)
- [Architecture Documentation](../architecture/overview.md)
- [Service Documentation](../services/)

### Asking Questions

When asking questions:

1. Be specific and provide details
2. Include your environment and setup
3. Show what you've tried
4. Use appropriate communication channels

## Recognition

We appreciate all contributors:

- **Code Contributors**: Developers who write code
- **Documentation Contributors**: Writers who improve docs
- **Bug Reporters**: Users who report issues
- **Feature Requesters**: Users who suggest improvements

All contributors are recognized in:
- Contributors list in README
- Release notes
- Special thanks for significant contributions

---

*These guidelines should be updated as the project evolves. Submit a pull request if you have suggestions for improvements.*
