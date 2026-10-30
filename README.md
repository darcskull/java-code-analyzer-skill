# Java Code Analyzer - Devin Skill

A Devin CLI skill for analyzing and explaining Java code, identifying issues, and suggesting improvements.

## About

This skill extends Devin CLI with specialized Java code analysis capabilities. When invoked, it provides comprehensive insights into Java codebases including:

- **Code Structure & Architecture**: Class relationships, inheritance hierarchies, design patterns
- **Code Quality**: Code smells, anti-patterns, naming conventions, method complexity
- **Potential Issues**: Bugs, thread safety concerns, security vulnerabilities, performance bottlenecks
- **Best Practices**: Java best practices, exception handling, collection usage, logging
- **Dependencies**: Unused imports, deprecated APIs, external dependencies

## Installation

### Project-Specific Installation

1. Clone this repository
2. The skill is located in `.devin/skills/java-code-analyzer/`
3. It will be automatically available when using Devin CLI in this project

### Global Installation (Available in All Projects)

**Windows:**
```bash
mkdir %APPDATA%\devin\skills\java-code-analyzer
copy .devin\skills\java-code-analyzer\SKILL.md %APPDATA%\devin\skills\java-code-analyzer\
```

**Linux/macOS:**
```bash
mkdir -p ~/.config/devin/skills/java-code-analyzer
cp .devin/skills/java-code-analyzer/SKILL.md ~/.config/devin/skills/java-code-analyzer/
```

## Central Repository with Submodules

This repository serves as a central hub for Java development projects using the Devin CLI Java Code Analyzer skill. Related projects are included as Git submodules for convenient access and development.

### Current Submodules

- **[sandbox-interview](projects/sandbox-interview)** - Java interview sandbox project

### Managing Submodules

#### Clone with Submodules
```bash
git clone --recursive https://github.com/darcskull/java-code-analyzer-skill.git
```

#### Initialize Submodules (if cloned without --recursive)
```bash
git submodule update --init --recursive
```

#### Update Submodules
```bash
# Update all submodules to latest commit
git submodule update --remote

# Update specific submodule
cd projects/sandbox-interview
git pull origin main
cd ../..
git add projects/sandbox-interview
git commit -m "Update sandbox-interview submodule"
```

#### Working with Submodules
```bash
# Navigate to a submodule
cd projects/sandbox-interview

# Make changes in the submodule
git add .
git commit -m "Your changes"
git push origin main

# Update the parent repository to track the new commit
cd ../..
git add projects/sandbox-interview
git commit -m "Update sandbox-interview to latest commit"
```

### Adding New Submodules

To add a new Java project as a submodule:

```bash
git submodule add https://github.com/USERNAME/PROJECT-NAME.git projects/PROJECT-NAME
git add .gitmodules projects/PROJECT-NAME
git commit -m "Add PROJECT-NAME as submodule"
```

## Usage

Once installed, invoke the skill in Devin CLI:

```bash
/java-code-analyzer
```

You can also specify a file or pattern to analyze:

```bash
/java-code-analyzer src/main/java/com/example/MyClass.java
```

Or analyze a specific package:

```bash
/java-code-analyzer src/main/java/com/example/
```

## Features

### Structured Analysis Output

The skill provides a structured analysis including:

- **Summary**: Overview of the codebase or file
- **Architecture**: Class relationships and design patterns
- **Findings**: Strengths, issues (with severity), and suggestions
- **Code Examples**: Before/after examples for improvements
- **File References**: Specific files and line numbers with clickable citations

### Auto-Approved Tools

The skill has the following tools auto-approved:
- `read` - Read file contents
- `grep` - Search for patterns in code
- `glob` - Find files matching patterns
- `exec` - Execute shell commands

### Triggers

The skill can be invoked by:
- User: `/java-code-analyzer`
- Model: Auto-invoked when the AI detects Java code analysis needs

## Example Output

```
### Summary
This codebase contains a Spring Boot application with 3 main components: UserController, UserService, and UserRepository.

### Architecture
- UserController: REST API layer
- UserService: Business logic layer
- UserRepository: Data access layer
- Design Pattern: Repository pattern, MVC architecture

### Findings

**Strengths:**
- Clean separation of concerns
- Proper use of dependency injection
- Consistent naming conventions

**Issues:**
- [HIGH] UserService has a method with 45 lines - consider splitting
- [MEDIUM] UserController lacks input validation
- [LOW] Unused import in UserService.java

**Suggestions:**
- Extract the large method in UserService into smaller helper methods
- Add @Valid annotation to request parameters
- Remove unused imports to reduce clutter
```

## Development

### Modifying the Skill

To modify the skill's behavior, edit `.devin/skills/java-code-analyzer/SKILL.md`. The file contains:

1. **Frontmatter**: Metadata (name, description, allowed-tools, permissions)
2. **Prompt Content**: Instructions for the AI agent

### Testing the Skill

After making changes, test the skill:

```bash
/java-code-analyzer
```

Iterate on the prompt until the output matches your expectations.

## License

This skill is provided as-is for educational and development purposes.

## Contributing

Feel free to submit issues and pull requests to improve the skill's capabilities.

## Documentation

This project includes comprehensive documentation for AI agents, governance, and best practices:

- **[Agents.md](Agents.md)** - AI agent behavior rules and guidelines
- **[Governance](docs/governance/governance.md)** - Project governance policies and procedures
- **[Submodule Management](docs/submodule-management.md)** - Guide for managing Git submodules
- **[Knowledge Base](docs/knowledge-base/)** - Reference materials and best practices
  - [Java Coding Conventions](docs/knowledge-base/java-best-practices/coding-conventions.md)
  - [Design Patterns](docs/knowledge-base/design-patterns/common-patterns.md)
  - [Secure Coding](docs/knowledge-base/security/secure-coding.md)
  - [Performance Optimization](docs/knowledge-base/performance/optimization.md)
  - [Testing Strategies](docs/knowledge-base/testing/testing-strategies.md)

## Related Resources

- [Devin CLI Documentation](https://devin.ai/docs)
- [Creating Skills Guide](https://devin.ai/docs/extensibility/skills/creating-skills)
- [Java Best Practices](https://github.com/java-best-practices)
