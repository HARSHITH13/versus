# Contributing to Versus

First off, thank you for considering contributing to Versus! It's people like you that make Versus such a great tool.

## Code of Conduct

This project and everyone participating in it is governed by our Code of Conduct. By participating, you are expected to uphold this code.

## How Can I Contribute?

### Reporting Bugs

Before creating bug reports, please check the existing issues as you might find out that you don't need to create one. When you are creating a bug report, please include as many details as possible:

* **Use a clear and descriptive title** for the issue to identify the problem.
* **Describe the exact steps which reproduce the problem** in as many details as possible.
* **Provide specific examples to demonstrate the steps**.
* **Describe the behavior you observed after following the steps** and point out what exactly is the problem with that behavior.
* **Explain which behavior you expected to see instead and why.**
* **Include screenshots and animated GIFs** if possible.

### Suggesting Enhancements

Enhancement suggestions are tracked as GitHub issues. When creating an enhancement suggestion, please include:

* **Use a clear and descriptive title** for the issue to identify the suggestion.
* **Provide a step-by-step description of the suggested enhancement** in as many details as possible.
* **Provide specific examples to demonstrate the steps**.
* **Describe the current behavior** and **explain which behavior you expected to see instead** and why.
* **Explain why this enhancement would be useful** to most Versus users.

### Pull Requests

* Fill in the required template
* Do not include issue numbers in the PR title
* Follow the [C# Coding Conventions](https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions)
* Include thoughtfully-worded, well-structured tests
* Document new code based on the [Documentation Styleguide](#documentation-styleguide)
* End all files with a newline

## Development Setup

### Prerequisites

- .NET 8 SDK
- Docker Desktop
- Visual Studio 2022 or VS Code
- Azure CLI (for Azure development)

### Setting Up Your Development Environment

1. Fork the repository
2. Clone your fork:
   ```bash
   git clone https://github.com/your-username/versus.git
   cd versus
   ```

3. Add the upstream repository:
   ```bash
   git remote add upstream https://github.com/original-owner/versus.git
   ```

4. Start local infrastructure:
   ```bash
   docker-compose up -d
   ```

5. Run database migrations:
   ```bash
   cd src/Versus.Infrastructure
   dotnet ef database update --startup-project ../Versus.Api
   ```

6. Build the solution:
   ```bash
   dotnet build
   ```

7. Run tests:
   ```bash
   dotnet test
   ```

### Development Workflow

1. Create a new branch:
   ```bash
   git checkout -b feature/my-new-feature
   ```

2. Make your changes and commit them:
   ```bash
   git add .
   git commit -m "Add some feature"
   ```

3. Push to your fork:
   ```bash
   git push origin feature/my-new-feature
   ```

4. Create a Pull Request

## Styleguides

### Git Commit Messages

* Use the present tense ("Add feature" not "Added feature")
* Use the imperative mood ("Move cursor to..." not "Moves cursor to...")
* Limit the first line to 72 characters or less
* Reference issues and pull requests liberally after the first line
* Consider starting the commit message with an applicable emoji:
    * 🎨 `:art:` when improving the format/structure of the code
    * 🐎 `:racehorse:` when improving performance
    * 📝 `:memo:` when writing docs
    * 🐛 `:bug:` when fixing a bug
    * 🔥 `:fire:` when removing code or files
    * ✅ `:white_check_mark:` when adding tests
    * 🔒 `:lock:` when dealing with security
    * ⬆️ `:arrow_up:` when upgrading dependencies
    * ⬇️ `:arrow_down:` when downgrading dependencies

### C# Styleguide

* Follow [Microsoft's C# Coding Conventions](https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions)
* Use `var` when the type is obvious
* Use meaningful variable and method names
* Keep methods small and focused (Single Responsibility Principle)
* Write unit tests for new functionality
* Maintain code coverage above 80%

### Documentation Styleguide

* Use [Markdown](https://guides.github.com/features/mastering-markdown/)
* Reference methods and classes in backticks: \`ClassName\`
* Include code examples when applicable
* Keep documentation up to date with code changes

## Testing

### Running Tests

```bash
# Run all tests
dotnet test

# Run tests with coverage
dotnet test --collect:"XPlat Code Coverage"

# Run specific test project
dotnet test tests/Versus.Domain.UnitTests/
```

### Writing Tests

* Follow the AAA pattern (Arrange, Act, Assert)
* Use descriptive test method names: `Should_ReturnTrue_When_InputIsValid`
* Use FluentAssertions for assertions
* Mock external dependencies with Moq
* Keep tests isolated and independent

Example:
```csharp
[Test]
public void Should_CreateTournament_When_ValidDataProvided()
{
    // Arrange
    var command = new CreateTournamentCommand
    {
        Name = "Test Tournament",
        StartDate = DateTime.UtcNow.AddDays(7),
        EndDate = DateTime.UtcNow.AddDays(14)
    };

    // Act
    var result = Tournament.Create(command.Name, command.StartDate, command.EndDate);

    // Assert
    result.Should().NotBeNull();
    result.Name.Should().Be(command.Name);
    result.Status.Should().Be(TournamentStatus.Draft);
}
```

## Architecture Guidelines

### Clean Architecture

Follow the principle of dependency inversion:
* Domain layer has no dependencies
* Application layer depends only on Domain
* Infrastructure layer depends on Application and Domain
* Presentation layer depends on Application (not Infrastructure)

### CQRS

* Commands: Operations that change state
* Queries: Operations that return data
* Keep commands and queries separate
* Use MediatR for handling commands and queries

### Domain-Driven Design

* Rich domain models with behavior
* Value objects for concepts without identity
* Aggregates to enforce consistency
* Domain events for cross-aggregate communication

## Pull Request Process

1. Ensure all tests pass
2. Update documentation if needed
3. Update the README.md with details of changes if applicable
4. The PR will be merged once you have the sign-off of at least one maintainer

## Recognition

Contributors who make significant contributions will be recognized in:
* The README.md file
* Release notes
* Our website (when launched)

## Questions?

Feel free to contact the project maintainers if you have any questions or need help getting started.

Thank you for contributing to Versus! 🎉
