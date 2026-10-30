# Code Analyzer Skills - Devin CLI

A collection of Devin CLI skills for comprehensive code analysis across multiple programming languages.

## About

This repository extends Devin CLI with specialized code analysis capabilities for Java and Python. When invoked, these skills provide comprehensive insights into codebases including:

### Java Code Analyzer
- **Code Structure & Architecture**: Class relationships, inheritance hierarchies, design patterns
- **Code Quality**: Code smells, anti-patterns, naming conventions, method complexity
- **Security**: SQL injection, XSS, authentication, proper exception handling
- **Performance**: String operations, collections, database optimization, memory management
- **Design Patterns**: Singleton, Factory, Builder, Strategy, Observer, DAO, DTO, Service Layer
- **Testing**: Unit tests, integration tests, test coverage
- **Best Practices**: Java conventions, proper resource management, logging

### Python Code Analyzer
- **Code Structure & Architecture**: Module organization, package structure, framework patterns
- **Code Quality**: PEP 8 compliance, naming conventions, type hints, docstrings
- **Security**: SQL injection, command injection, input validation, dependency vulnerabilities
- **Performance**: List comprehensions, data structures, generators, async/await
- **Design Patterns**: Singleton, Factory, Strategy, Observer, Decorator, Context Manager
- **Testing**: pytest, unittest, fixtures, test coverage
- **Best Practices**: Python idioms, proper exception handling, context managers

## Installation

### Project-Specific Installation

1. Clone this repository
2. The skills are located in `.devin/skills/`
3. They will be automatically available when using Devin CLI in this project

### Global Installation (Available in All Projects)

**Windows:**
```bash
# Java Code Analyzer
mkdir %APPDATA%\devin\skills\java-code-analyzer
copy .devin\skills\java-code-analyzer\SKILL.md %APPDATA%\devin\skills\java-code-analyzer\

# Python Code Analyzer
mkdir %APPDATA%\devin\skills\python-code-analyzer
copy .devin\skills\python-code-analyzer\SKILL.md %APPDATA%\devin\skills\python-code-analyzer\
```

**Linux/macOS:**
```bash
# Java Code Analyzer
mkdir -p ~/.config/devin/skills/java-code-analyzer
cp .devin/skills/java-code-analyzer/SKILL.md ~/.config/devin/skills/java-code-analyzer/

# Python Code Analyzer
mkdir -p ~/.config/devin/skills/python-code-analyzer
cp .devin/skills/python-code-analyzer/SKILL.md ~/.config/devin/skills/python-code-analyzer/
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

### Java Code Analyzer

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

### Python Code Analyzer

Invoke the Python code analyzer:

```bash
/python-code-analyzer
```

Specify a file or pattern to analyze:

```bash
/python-code-analyzer src/main/python/example/module.py
```

Or analyze a specific package:

```bash
/python-code-analyzer src/main/python/example/
```

## Features

### Structured Analysis Output

Both skills provide a structured analysis including:

- **Summary**: Overview of the codebase or file
- **Architecture**: Class/module relationships and design patterns
- **Code Quality Assessment**: Naming conventions, code smells, documentation
- **Security Findings**: Vulnerabilities with severity levels
- **Performance Analysis**: Optimization opportunities
- **Design Patterns**: Pattern identification and implementation
- **Testing Assessment**: Test coverage and quality
- **Findings**: Strengths, issues (with severity), and suggestions
- **Code Examples**: Before/after examples for improvements
- **Action Items**: Prioritized improvement list
- **File References**: Specific files and line numbers with clickable citations

### Auto-Approved Tools

Both skills have the following tools auto-approved:
- `read` - Read file contents
- `grep` - Search for patterns in code
- `glob` - Find files matching patterns
- `exec` - Execute shell commands

### Triggers

The skills can be invoked by:
- User: `/java-code-analyzer` or `/python-code-analyzer`
- Model: Auto-invoked when the AI detects code analysis needs

## Example Output

### Java Code Analyzer Example

```
### Summary
This codebase contains a Spring Boot application with 3 main components: UserController, UserService, and UserRepository.

### Architecture
- UserController: REST API layer
- UserService: Business logic layer
- UserRepository: Data access layer
- Design Pattern: Repository pattern, MVC architecture

### Security Findings
- [HIGH] SQL injection vulnerability in UserRepository.findById()
- [MEDIUM] Lack of input validation in UserController.createUser()

### Performance Analysis
- [MEDIUM] N+1 query problem in UserService.getUsersWithOrders()
- [LOW] String concatenation in loop in ReportGenerator.generate()

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

### Python Code Analyzer Example

```
### Summary
This codebase contains a FastAPI application with 3 main modules: user_controller, user_service, and user_repository.

### Architecture
- user_controller: REST API layer
- user_service: Business logic layer
- user_repository: Data access layer
- Design Pattern: Repository pattern, Clean Architecture

### Security Findings
- [HIGH] SQL injection vulnerability in user_repository.get_user()
- [MEDIUM] Hardcoded API key in config.py
- [LOW] Use of eval() in utils.py

### Performance Analysis
- [MEDIUM] Inefficient loop in data_processor.process()
- [LOW] String concatenation in loop in report_generator.generate()

### Code Quality Assessment
- [MEDIUM] Missing type hints in user_service.py
- [LOW] Inconsistent docstring style (mix of Google and NumPy)

### Findings

**Strengths:**
- Clean module organization
- Proper use of async/await
- Good separation of concerns

**Issues:**
- [HIGH] Mutable default argument in user_service.create_user()
- [MEDIUM] Missing __init__.py in models directory
- [LOW] Unused import in utils.py

**Suggestions:**
- Replace mutable default argument with None
- Add __init__.py to models directory
- Remove unused imports
```

## Development

### Modifying the Skills

To modify a skill's behavior, edit the corresponding SKILL.md file:

- **Java Code Analyzer**: `.devin/skills/java-code-analyzer/SKILL.md`
- **Python Code Analyzer**: `.devin/skills/python-code-analyzer/SKILL.md`

Each file contains:
1. **Frontmatter**: Metadata (name, description, allowed-tools, permissions)
2. **Prompt Content**: Instructions for the AI agent

### Testing the Skills

After making changes, test the corresponding skill:

```bash
/java-code-analyzer
# or
/python-code-analyzer
```

Iterate on the prompt until the output matches your expectations.

### Adding New Skills

To add a new language analyzer:

1. Create a new directory: `.devin/skills/<language>-code-analyzer/`
2. Create a SKILL.md file with appropriate frontmatter and prompt content
3. Update this README with the new skill
4. Test the skill thoroughly
5. Commit and push the changes

## License

These skills are provided as-is for educational and development purposes.

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

### Skill-Specific Documentation

- **[Java Code Analyzer SKILL.md](.devin/skills/java-code-analyzer/SKILL.md)** - Full Java skill definition
- **[Python Code Analyzer SKILL.md](.devin/skills/python-code-analyzer/SKILL.md)** - Full Python skill definition

## Related Resources

- [Devin CLI Documentation](https://devin.ai/docs)
- [Creating Skills Guide](https://devin.ai/docs/extensibility/skills/creating-skills)
- [Java Best Practices](https://github.com/java-best-practices)
