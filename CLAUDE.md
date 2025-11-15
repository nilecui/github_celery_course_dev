# CLAUDE.md - AI Assistant Guide for Celery Course Development

**Last Updated:** 2025-11-15
**Repository:** github_celery_course_dev
**Purpose:** Educational platform for creating comprehensive Celery training materials

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Repository Structure](#repository-structure)
3. [Development Workflows](#development-workflows)
4. [Key Conventions](#key-conventions)
5. [Working with Documentation](#working-with-documentation)
6. [Working with Examples](#working-with-examples)
7. [Docker Development Environment](#docker-development-environment)
8. [Custom Claude Code Agents](#custom-claude-code-agents)
9. [Git Workflow](#git-workflow)
10. [Common Tasks](#common-tasks)
11. [Best Practices](#best-practices)

---

## Project Overview

### What is This Repository?

This is **NOT** the Celery library source code. This is a **course development repository** designed to create comprehensive educational materials about Celery (the distributed task queue for Python).

**Key Facts:**
- **Type:** Educational Content Platform
- **Primary Goal:** Create high-quality training materials for Celery
- **Target Audience:** Developers learning distributed task processing with Celery
- **Content Types:** Documentation, examples, exercises, tutorials, case studies
- **AI Integration:** 11 specialized Claude Code agents for course development

### Technology Stack

- **Documentation:** Sphinx with `sphinx_celery` theme
- **Format:** reStructuredText (.rst files)
- **Examples:** Python 3.9+ (supports up to 3.13)
- **Development Environment:** Docker with docker-compose
- **Message Brokers:** RabbitMQ, Redis
- **Backends:** Redis, DynamoDB (local), Azure Storage (emulated)

---

## Repository Structure

```
/home/user/github_celery_course_dev/
├── .claude/                              # Claude Code configuration
│   ├── settings.local.json               # Claude settings
│   └── agents/                           # 11 specialized course development agents
│       ├── course-editor-in-chief.md     # Main quality assurance agent
│       ├── course-script-writer.md       # Lecture/video script creation
│       ├── learning-path-designer.md     # Curriculum design
│       ├── exercise-quiz-designer.md     # Assessment creation
│       ├── scenario-case-designer.md     # Case study design
│       ├── docs-gap-analyzer.md          # Documentation analysis
│       ├── feature-module-decomposer.md  # Module breakdown
│       ├── code-command-verifier.md      # Code validation
│       ├── demo-run-validator.md         # Demo validation
│       ├── api-config-indexer.md         # API indexing
│       └── github-repo-analyzer.md       # Repository analysis
│
├── celery/                               # Main course content
│   ├── docs/                             # Comprehensive documentation (100+ files)
│   │   ├── conf.py                       # Sphinx configuration
│   │   ├── index.rst                     # Main documentation index
│   │   ├── getting-started/              # Beginner tutorials
│   │   ├── userguide/                    # In-depth guides (19 major topics)
│   │   ├── tutorials/                    # Step-by-step tutorials
│   │   ├── django/                       # Django integration
│   │   ├── reference/                    # API reference
│   │   ├── internals/                    # Architecture docs
│   │   ├── history/                      # Changelog
│   │   ├── sec/                          # Security advisories
│   │   ├── Makefile                      # Build automation
│   │   └── make.bat                      # Windows build script
│   │
│   ├── examples/                         # 14 working example applications
│   │   ├── app/                          # Basic Celery app
│   │   ├── tutorial/                     # Simple task example
│   │   ├── django/                       # Django integration (full project)
│   │   ├── celery_http_gateway/          # HTTP API gateway
│   │   ├── periodic-tasks/               # Scheduled tasks with Beat
│   │   ├── eventlet/                     # Eventlet concurrency
│   │   ├── gevent/                       # Gevent concurrency
│   │   ├── pydantic/                     # Pydantic integration
│   │   ├── security/                     # TLS/security setup
│   │   ├── stamping/                     # Task versioning
│   │   ├── quorum-queues/                # RabbitMQ quorum queues
│   │   ├── next-steps/                   # Packaging tasks as module
│   │   └── resultgraph/                  # Task result graphs
│   │
│   └── docker/                           # Development environment
│       ├── Dockerfile                    # Multi-Python version image
│       ├── docker-compose.yml            # Service orchestration
│       ├── entrypoint                    # Container entrypoint
│       └── scripts/                      # Helper scripts
│
└── CLAUDE.md                             # This file
```

---

## Development Workflows

### Typical Course Development Workflow

1. **Content Planning**
   - Use `learning-path-designer` agent to structure curriculum
   - Break down topics into modules and lessons
   - Define learning objectives and prerequisites

2. **Documentation Writing**
   - Write in reStructuredText (.rst) format
   - Follow Sphinx conventions
   - Place files in appropriate `/celery/docs/` subdirectory

3. **Example Development**
   - Create working examples in `/celery/examples/`
   - Ensure examples are self-contained
   - Include clear comments and documentation

4. **Quality Assurance**
   - Use `code-command-verifier` to validate code examples
   - Use `demo-run-validator` to test example applications
   - Use `course-editor-in-chief` for comprehensive review

5. **Assessment Creation**
   - Use `exercise-quiz-designer` for exercises and quizzes
   - Use `scenario-case-designer` for practical case studies

6. **Documentation Build**
   - Build documentation: `cd celery/docs && make html`
   - Preview: `make livehtml` (port 7001 in Docker)
   - Check coverage: `make coverage`

7. **Final Review**
   - Use `docs-gap-analyzer` to identify missing content
   - Use `course-editor-in-chief` for consistency check
   - Validate all links and references

---

## Key Conventions

### Documentation Conventions

#### File Organization
- **Getting Started:** `/celery/docs/getting-started/` - Beginner content
- **User Guide:** `/celery/docs/userguide/` - Comprehensive guides
- **Tutorials:** `/celery/docs/tutorials/` - Step-by-step walkthroughs
- **API Reference:** `/celery/docs/reference/` - API documentation
- **Django:** `/celery/docs/django/` - Django-specific content

#### reStructuredText Best Practices

**Headers:**
```rst
Main Title
==========

Section Header
--------------

Subsection Header
~~~~~~~~~~~~~~~~~

Sub-subsection Header
^^^^^^^^^^^^^^^^^^^^^
```

**Code Blocks:**
```rst
.. code-block:: python

    from celery import Celery

    app = Celery('tasks', broker='pyamqp://guest@localhost//')

    @app.task
    def add(x, y):
        return x + y
```

**Links:**
```rst
:ref:`section-reference-label`
:doc:`other-document`
:class:`celery.app.Celery`
:func:`celery.task`
```

**Notes and Warnings:**
```rst
.. note::
    This is important information for the reader.

.. warning::
    This is a critical warning.

.. versionadded:: 5.0
    This feature was added in version 5.0.

.. deprecated:: 5.0
    This feature is deprecated.
```

### Code Example Conventions

#### Example Structure
Each example should contain:
1. **Clear purpose statement** - What does this example demonstrate?
2. **Prerequisites** - What needs to be installed/configured?
3. **Code files** - Well-commented, production-quality code
4. **README or docstring** - How to run the example
5. **Configuration** - Any required config files

#### Example File Naming
- `tasks.py` - Task definitions
- `celeryconfig.py` - Configuration (if needed)
- `myapp.py` or `app.py` - Application setup
- `requirements.txt` - Dependencies (if specific)

#### Code Quality Standards
- **Python Version:** Support Python 3.9+
- **Style:** Follow PEP 8
- **Comments:** Explain WHY, not just WHAT
- **Error Handling:** Include appropriate error handling
- **Imports:** Explicit and organized (stdlib, third-party, local)

### Django Example Conventions
- Use Django project structure: `proj/` for project, `demoapp/` for app
- Include `manage.py`, `settings.py`, `celery.py`, `tasks.py`
- Provide `requirements.txt` with Django and Celery versions
- Document integration steps in comments

---

## Working with Documentation

### Building Documentation

**Local Build:**
```bash
cd celery/docs
make html
# Output: _build/html/index.html
```

**Live Server (with auto-reload):**
```bash
cd celery/docs
make livehtml
# Server: http://localhost:7001
```

**In Docker:**
```bash
docker-compose up docs
# Access: http://localhost:7001
```

### Documentation Quality Checks

**Spell Check:**
```bash
cd celery/docs
make spelling
# Uses spelling_wordlist.txt for custom terms
```

**Coverage Check:**
```bash
cd celery/docs
make coverage
# Reports undocumented APIs
```

**API Consistency:**
```bash
cd celery/docs
make apicheck
# Validates API references
```

**Configuration Check:**
```bash
cd celery/docs
make configcheck
# Validates configuration examples
```

### Adding New Documentation

1. **Create .rst file** in appropriate directory
2. **Add to index** in parent directory's `index.rst` or appropriate toctree
3. **Use proper headers** (see conventions above)
4. **Include code examples** with proper syntax highlighting
5. **Cross-reference** related topics using `:ref:`, `:doc:`, etc.
6. **Build and verify** with `make html`

### Documentation Structure Guidelines

**User Guide Topics (19 major areas):**
1. Application setup (`application.rst`)
2. Task definition (`tasks.rst`)
3. Task calling patterns (`calling.rst`)
4. Workflow primitives (`canvas.rst`)
5. Configuration (`configuration.rst`)
6. Workers (`workers.rst`)
7. Periodic tasks (`periodic-tasks.rst`)
8. Routing (`routing.rst`)
9. Signals (`signals.rst`)
10. Monitoring (`monitoring.rst`)
11. Testing (`testing.rst`)
12. Security (`security.rst`)
13. Daemonization (`daemonizing.rst`)
14. Debugging (`debugging.rst`)
15. Extensions (`extending.rst`)
16. Optimization (`optimizing.rst`)
17. Concurrency models (`concurrency/`)
18. Additional guides in userguide/

---

## Working with Examples

### Example Categories

#### 1. Basic Examples
- **app/** - Minimal Celery application
- **tutorial/** - First steps (simple add task)

#### 2. Integration Examples
- **django/** - Full Django project with Celery
- **celery_http_gateway/** - HTTP API to task execution
- **pydantic/** - Pydantic model validation

#### 3. Concurrency Examples
- **eventlet/** - Eventlet worker pool + webcrawler demo
- **gevent/** - Gevent worker pool

#### 4. Advanced Features
- **periodic-tasks/** - Beat scheduler configuration
- **stamping/** - Task versioning and revocation
- **resultgraph/** - Task dependency graphs
- **security/** - TLS/SSL security setup

#### 5. Infrastructure Examples
- **quorum-queues/** - RabbitMQ quorum queues setup
- **next-steps/** - Packaging tasks as Python module

### Running Examples

**Standalone Examples:**
```bash
cd celery/examples/<example-name>
# Start worker
celery -A <module> worker --loglevel=info

# In another terminal, run tasks
python -c "from <module> import <task>; <task>.delay(args)"
```

**Django Example:**
```bash
cd celery/examples/django
pip install -r requirements.txt
python manage.py migrate
# Start worker
celery -A proj worker --loglevel=info
# Start Django
python manage.py runserver
```

**Docker Environment:**
```bash
docker-compose up -d rabbit redis
docker-compose run --rm celery bash
# Inside container, navigate to example and run
```

### Creating New Examples

1. **Create directory:** `/celery/examples/<example-name>/`
2. **Add core files:**
   - `tasks.py` - Task definitions
   - `app.py` or `myapp.py` - Celery app setup
   - `celeryconfig.py` - Configuration (if needed)
3. **Document usage:**
   - Add clear docstrings
   - Include example invocations in comments
   - Create README if complex
4. **Test thoroughly:**
   - Use `demo-run-validator` agent
   - Test with different Python versions (3.9, 3.10, 3.11, 3.12, 3.13)
   - Verify with Docker environment
5. **Reference in docs:**
   - Link from appropriate documentation page
   - Include in tutorials if instructive

---

## Docker Development Environment

### Services

**docker-compose.yml provides 6 services:**

1. **celery** - Main development container
   - Image: `celery/celery:dev`
   - Python versions: 3.13, 3.12, 3.11, 3.10, 3.9, PyPy3.11 (via pyenv)
   - Volume: Repository mounted at `/celery`
   - Command: Keeps container running for interactive use

2. **rabbit** - RabbitMQ message broker
   - Image: `rabbitmq:3-management`
   - Port: 5672 (AMQP), 15672 (management UI)
   - Credentials: guest/guest
   - Management UI: http://localhost:15672

3. **redis** - Redis results backend
   - Image: `redis:latest`
   - Port: 6379

4. **dynamodb** - AWS DynamoDB Local
   - Image: `amazon/dynamodb-local`
   - Port: 8000

5. **azurite** - Azure Storage Emulator
   - Image: `mcr.microsoft.com/azure-storage/azurite`
   - Ports: 10000 (blob), 10001 (queue), 10002 (table)

6. **docs** - Sphinx documentation server
   - Image: Same as celery
   - Port: 7001
   - Command: `make livehtml` (auto-reload on changes)
   - Access: http://localhost:7001

### Common Docker Commands

**Start all services:**
```bash
docker-compose up -d
```

**Interactive shell in celery container:**
```bash
docker-compose run --rm celery bash
```

**Run with specific Python version (in container):**
```bash
pyenv global 3.13  # or 3.12, 3.11, 3.10, 3.9, pypy3.11
python --version
```

**View logs:**
```bash
docker-compose logs -f celery
docker-compose logs -f rabbit
```

**Stop all services:**
```bash
docker-compose down
```

**Rebuild celery image (after Dockerfile changes):**
```bash
docker-compose build celery
```

**Run tests in Docker:**
```bash
docker-compose run --rm celery bash -c "cd /celery && pytest"
```

### Environment Variables

**Configured in docker-compose.yml:**
- `TEST_BROKER=pyamqp://rabbit:5672` - RabbitMQ for testing
- `TEST_BACKEND=redis://redis` - Redis for testing
- `PYTHONUNBUFFERED=1` - Unbuffered output
- `PYTHONDONTWRITEBYTECODE=1` - Prevent .pyc files

---

## Custom Claude Code Agents

### Agent Overview

11 specialized agents are available in `.claude/agents/`. All use `model: opus` for advanced reasoning.

### Agent Usage Guide

#### 1. **course-editor-in-chief** - Quality Assurance & Review

**When to Use:**
- Final review before publishing course materials
- Ensuring consistency across chapters/sections
- Checking terminology and style consistency
- Validating cross-references and links

**Example:**
```
Use the course-editor-in-chief agent to review all documentation
in celery/docs/userguide/ for consistency and quality.
```

#### 2. **learning-path-designer** - Curriculum Design

**When to Use:**
- Designing course structure and progression
- Creating learning paths for different skill levels
- Organizing topics into logical modules
- Defining prerequisites and dependencies

**Example:**
```
Use the learning-path-designer agent to create a learning path for
intermediate Python developers new to distributed task processing.
```

#### 3. **exercise-quiz-designer** - Assessment Creation

**When to Use:**
- Creating exercises for each chapter
- Designing quizzes and assessments
- Developing hands-on coding challenges
- Building knowledge checks

**Example:**
```
Use the exercise-quiz-designer agent to create exercises for
the periodic tasks chapter, covering Beat scheduler and crontab syntax.
```

#### 4. **scenario-case-designer** - Case Study Development

**When to Use:**
- Creating real-world scenarios
- Designing practical case studies
- Building project-based learning exercises
- Developing capstone projects

**Example:**
```
Use the scenario-case-designer agent to create a case study
demonstrating a background job processing system for an e-commerce platform.
```

#### 5. **course-script-writer** - Lecture Script Creation

**When to Use:**
- Writing lecture scripts for video courses
- Creating instructor notes
- Developing presentation materials
- Writing narration for tutorials

**Example:**
```
Use the course-script-writer agent to create a lecture script
for introducing Celery workers and their configuration.
```

#### 6. **docs-gap-analyzer** - Documentation Analysis

**When to Use:**
- Identifying missing documentation
- Finding inconsistencies in coverage
- Discovering outdated information
- Analyzing documentation completeness

**Example:**
```
Use the docs-gap-analyzer agent to identify gaps in the
security documentation section.
```

#### 7. **feature-module-decomposer** - Module Breakdown

**When to Use:**
- Breaking down complex features into modules
- Analyzing feature dependencies
- Creating module-based curriculum structure
- Understanding codebase organization

**Example:**
```
Use the feature-module-decomposer agent to break down
Celery's canvas feature (chains, groups, chords) into teachable modules.
```

#### 8. **code-command-verifier** - Code Validation

**When to Use:**
- Validating code examples in documentation
- Verifying command-line instructions
- Testing code snippets
- Ensuring code examples are runnable

**Example:**
```
Use the code-command-verifier agent to validate all Python code
examples in the getting-started tutorials.
```

#### 9. **demo-run-validator** - Demo Validation

**When to Use:**
- Testing example applications
- Validating demo functionality
- Ensuring examples work with different configurations
- Testing across Python versions

**Example:**
```
Use the demo-run-validator agent to validate the Django
integration example works correctly.
```

#### 10. **api-config-indexer** - API Documentation

**When to Use:**
- Indexing API endpoints and configurations
- Cataloging configuration parameters
- Documenting command-line tools
- Creating API reference materials

**Example:**
```
Use the api-config-indexer agent to index all Celery
configuration parameters for the configuration reference.
```

#### 11. **github-repo-analyzer** - Repository Analysis

**When to Use:**
- Understanding repository structure
- Analyzing codebases for course material creation
- Creating repository overviews
- Documenting project organization

**Example:**
```
Use the github-repo-analyzer agent to analyze the Celery
GitHub repository and create an architecture overview.
```

### Agent Selection Matrix

| Task | Primary Agent | Supporting Agents |
|------|---------------|-------------------|
| **Plan new course module** | learning-path-designer | feature-module-decomposer |
| **Write documentation** | course-script-writer | docs-gap-analyzer |
| **Create exercises** | exercise-quiz-designer | scenario-case-designer |
| **Validate content** | course-editor-in-chief | code-command-verifier |
| **Test examples** | demo-run-validator | code-command-verifier |
| **API documentation** | api-config-indexer | docs-gap-analyzer |
| **Repository analysis** | github-repo-analyzer | feature-module-decomposer |

---

## Git Workflow

### Branch Naming Convention

**Feature branches MUST follow this pattern:**
```
claude/claude-md-<session-id>
```

**Current branch:**
```
claude/claude-md-mhzpnqme0q9d4s2z-019mPfHVWg5PT1Uc7CWSEWME
```

**CRITICAL:** Push will fail with 403 error if branch name doesn't match the pattern.

### Commit Guidelines

**Commit Message Format:**
```
<type>: <short description>

<detailed description if needed>
```

**Types:**
- `docs:` - Documentation changes
- `examples:` - Example application changes
- `course:` - Course material updates
- `agents:` - Claude agent modifications
- `docker:` - Docker configuration changes
- `fix:` - Bug fixes
- `test:` - Testing changes

**Examples:**
```
docs: Add advanced canvas patterns to userguide

Add detailed examples of chain, group, and chord combinations
including error handling and result processing.
```

```
examples: Create new Django REST API example

Demonstrate integration of Celery with Django REST Framework
for asynchronous API endpoints.
```

### Push Workflow

**Always use `-u` flag on first push:**
```bash
git push -u origin claude/claude-md-mhzpnqme0q9d4s2z-019mPfHVWg5PT1Uc7CWSEWME
```

**Retry logic for network failures:**
- Retry up to 4 times with exponential backoff (2s, 4s, 8s, 16s)
- Only for network errors, not authentication failures

**Fetch specific branches:**
```bash
git fetch origin claude/claude-md-mhzpnqme0q9d4s2z-019mPfHVWg5PT1Uc7CWSEWME
```

### Current Repository State

```
Remote: http://local_proxy@127.0.0.1:57268/git/nilecui/github_celery_course_dev
Branch: claude/claude-md-mhzpnqme0q9d4s2z-019mPfHVWg5PT1Uc7CWSEWME
Status: Clean working tree
Recent commits:
  e284429 init
  0ec4062 first commit
```

---

## Common Tasks

### 1. Adding New Documentation Page

```bash
# 1. Create new .rst file
touch celery/docs/userguide/my-new-topic.rst

# 2. Edit the file with proper structure
cat > celery/docs/userguide/my-new-topic.rst << 'EOF'
.. _my-new-topic:

===============
My New Topic
===============

Introduction paragraph explaining the topic.

Prerequisites
=============

- Prerequisite 1
- Prerequisite 2

Basic Example
=============

.. code-block:: python

    from celery import Celery

    app = Celery('myapp')

    @app.task
    def my_task():
        return "Hello World"

.. seealso::

    :ref:`related-topic`
        Related topic description.
EOF

# 3. Add to appropriate toctree
# Edit celery/docs/userguide/index.rst and add:
# .. toctree::
#    my-new-topic

# 4. Build and verify
cd celery/docs
make html
# Check: _build/html/userguide/my-new-topic.html
```

### 2. Creating New Example

```bash
# 1. Create example directory
mkdir -p celery/examples/my-example

# 2. Create task file
cat > celery/examples/my-example/tasks.py << 'EOF'
"""Example demonstrating [specific feature].

Usage:
    # Start worker
    celery -A tasks worker --loglevel=info

    # Run tasks
    python -c "from tasks import my_task; my_task.delay()"
"""
from celery import Celery

app = Celery('my-example', broker='pyamqp://guest@localhost//')

@app.task
def my_task():
    """Demonstrate [specific feature]."""
    return "Task completed"

if __name__ == '__main__':
    # Example invocation
    result = my_task.delay()
    print(f"Task ID: {result.id}")
EOF

# 3. Test the example
cd celery/examples/my-example

# Start worker
celery -A tasks worker --loglevel=info &

# Run task
python -c "from tasks import my_task; print(my_task.delay())"

# 4. Validate with agent
# Use demo-run-validator agent to verify
```

### 3. Building Documentation Locally

```bash
# Clean build
cd celery/docs
make clean
make html

# Live preview with auto-reload
make livehtml
# Open http://localhost:7001

# Check for errors
make linkcheck  # Verify external links
make coverage   # Check documentation coverage
make spelling   # Spell check
```

### 4. Testing Across Python Versions (Docker)

```bash
# Start Docker environment
docker-compose up -d rabbit redis

# Test with Python 3.13
docker-compose run --rm celery bash -c "
pyenv global 3.13 &&
python --version &&
cd /celery/examples/tutorial &&
celery -A tasks worker --loglevel=info &
sleep 5 &&
python -c 'from tasks import add; print(add.delay(4, 4).get())'
"

# Test with Python 3.9
docker-compose run --rm celery bash -c "
pyenv global 3.9 &&
python --version &&
cd /celery/examples/tutorial &&
celery -A tasks worker --loglevel=info &
sleep 5 &&
python -c 'from tasks import add; print(add.delay(4, 4).get())'
"
```

### 5. Validating All Code Examples

```bash
# Use code-command-verifier agent
# Example prompt:
"Use the code-command-verifier agent to validate all Python code
examples in celery/docs/getting-started/ and report any issues."
```

### 6. Creating Course Assessment

```bash
# Use exercise-quiz-designer agent
# Example prompt:
"Use the exercise-quiz-designer agent to create 10 quiz questions
covering the topics in celery/docs/userguide/tasks.rst, including
basic task definition, decorators, and task states."
```

### 7. Analyzing Documentation Gaps

```bash
# Use docs-gap-analyzer agent
# Example prompt:
"Use the docs-gap-analyzer agent to analyze celery/docs/userguide/
and identify any missing topics or incomplete sections compared to
the actual Celery API."
```

### 8. Creating Learning Path

```bash
# Use learning-path-designer agent
# Example prompt:
"Use the learning-path-designer agent to create a 4-week learning
path for Python developers new to Celery, progressing from basics
to advanced workflows and production deployment."
```

---

## Best Practices

### Documentation Best Practices

1. **Clarity First**
   - Write for beginners, even in advanced sections
   - Define terms before using them
   - Use concrete examples, not just abstract descriptions

2. **Code Examples**
   - Always include working, runnable examples
   - Show complete code, not fragments (unless for illustration)
   - Include expected output in comments
   - Demonstrate error handling

3. **Structure**
   - Follow consistent header hierarchy
   - Use cross-references extensively
   - Include "See Also" sections
   - Add version annotations (versionadded, versionchanged, deprecated)

4. **Testing**
   - Build documentation before committing
   - Check for broken links
   - Validate code examples with `code-command-verifier`
   - Review spelling

### Example Best Practices

1. **Self-Contained**
   - Each example should run independently
   - Include all necessary imports
   - Document prerequisites clearly

2. **Educational Value**
   - Focus on one concept per example
   - Progress from simple to complex
   - Add comments explaining WHY, not just WHAT

3. **Production-Ready Patterns**
   - Show best practices, not just "it works" code
   - Include error handling
   - Demonstrate proper configuration
   - Show logging and monitoring setup

4. **Compatibility**
   - Test with Python 3.9, 3.10, 3.11, 3.12, 3.13
   - Document version-specific features
   - Use type hints where helpful
   - Follow modern Python conventions

### Agent Usage Best Practices

1. **Use Specific Agents for Specific Tasks**
   - Don't use general-purpose approaches when specialized agents exist
   - Combine agents for complex workflows
   - Reference agent output in commit messages

2. **Validate Generated Content**
   - Always review agent output before committing
   - Test generated code examples
   - Verify accuracy of technical content

3. **Iterative Improvement**
   - Use `docs-gap-analyzer` regularly to find issues
   - Use `course-editor-in-chief` for periodic quality reviews
   - Continuously validate examples with `demo-run-validator`

### Git Best Practices

1. **Commit Frequently**
   - Small, focused commits
   - Clear, descriptive messages
   - One logical change per commit

2. **Branch Management**
   - Always work on the designated Claude branch
   - Never push to wrong branch
   - Keep branch up-to-date with main (if needed)

3. **Quality Gates**
   - Build documentation before committing
   - Run validation agents on changed content
   - Verify examples work before committing

### Docker Best Practices

1. **Resource Management**
   - Stop services when not in use: `docker-compose down`
   - Clean up volumes periodically
   - Monitor container logs for issues

2. **Consistency**
   - Always test in Docker to match production environment
   - Use consistent Python versions across examples
   - Document which services are needed for each example

3. **Troubleshooting**
   - Check logs: `docker-compose logs -f <service>`
   - Verify service connectivity from celery container
   - Rebuild images after dependency changes

---

## Quick Reference

### Key Directories

```
celery/docs/getting-started/   # Beginner tutorials
celery/docs/userguide/         # Comprehensive guides (19 topics)
celery/docs/tutorials/         # Step-by-step tutorials
celery/docs/reference/         # API reference
celery/examples/               # 14 working examples
.claude/agents/                # 11 specialized agents
```

### Key Commands

```bash
# Documentation
cd celery/docs && make html         # Build HTML docs
cd celery/docs && make livehtml     # Live preview (port 7001)

# Docker
docker-compose up -d                # Start services
docker-compose run --rm celery bash # Interactive shell
docker-compose logs -f celery       # View logs

# Testing Examples
celery -A tasks worker --loglevel=info  # Start worker
python -c "from tasks import add; add.delay(4, 4)"  # Run task

# Git
git status                          # Check status
git add .                           # Stage changes
git commit -m "type: description"   # Commit
git push -u origin <branch>         # Push (use full branch name)
```

### Important Files

```
celery/docs/conf.py                 # Sphinx configuration
celery/docs/index.rst               # Main doc index
celery/docker/docker-compose.yml    # Service definitions
.claude/agents/course-editor-in-chief.md  # Main QA agent
```

### Service URLs (Docker)

```
RabbitMQ Management: http://localhost:15672 (guest/guest)
Documentation Live:  http://localhost:7001
Redis:              localhost:6379
DynamoDB Local:     localhost:8000
```

---

## Troubleshooting

### Documentation Build Fails

**Issue:** `make html` fails with errors

**Solutions:**
1. Check RST syntax: ensure proper indentation
2. Verify cross-references: all `:ref:`, `:doc:` targets exist
3. Check code blocks: proper `.. code-block::` syntax
4. Review Sphinx output for specific error line numbers
5. Use `make clean && make html` for fresh build

### Example Won't Run

**Issue:** Example code fails to execute

**Solutions:**
1. Verify broker is running: RabbitMQ or Redis
2. Check connection URL in code
3. Ensure Celery is installed: `pip install celery`
4. Check Python version compatibility
5. Use Docker environment for consistent setup

### Git Push Fails with 403

**Issue:** `git push` returns 403 Forbidden

**Solutions:**
1. **Verify branch name:** MUST start with `claude/` and end with session ID
2. Check current branch: `git branch --show-current`
3. Use exact branch name provided in instructions
4. **DO NOT** create custom branch names

### Docker Services Won't Start

**Issue:** `docker-compose up` fails

**Solutions:**
1. Check port conflicts: 5672, 6379, 7001, 15672 must be free
2. Verify Docker daemon is running
3. Check disk space: `df -h`
4. Review logs: `docker-compose logs`
5. Rebuild: `docker-compose build`

### Agent Not Producing Expected Results

**Issue:** Claude agent output is not helpful

**Solutions:**
1. Provide more context in the prompt
2. Specify desired output format
3. Break complex tasks into smaller steps
4. Use multiple agents in sequence
5. Review agent description to ensure correct agent choice

---

## Additional Resources

### Celery Resources

- **Official Docs:** https://docs.celeryq.dev
- **GitHub:** https://github.com/celery/celery
- **Community:** celery/docs/community.rst
- **Security:** celery/docs/sec/ (security advisories)

### Sphinx Resources

- **Sphinx Documentation:** https://www.sphinx-doc.org
- **reStructuredText:** https://docutils.sourceforge.io/rst.html
- **sphinx_celery:** Custom Celery theme and extensions

### Docker Resources

- **Docker Compose:** https://docs.docker.com/compose/
- **Docker Best Practices:** https://docs.docker.com/develop/dev-best-practices/

### Python Resources

- **PEP 8 Style Guide:** https://pep8.org
- **Type Hints:** https://docs.python.org/3/library/typing.html
- **Python 3.9+ Features:** https://docs.python.org/3/whatsnew/

---

## Appendix: Example Templates

### Documentation Page Template

```rst
.. _topic-reference-name:

====================
Page Title Here
====================

Brief introduction explaining what this page covers and why it's important.

.. contents::
    :local:
    :depth: 2

Prerequisites
=============

Before reading this guide, you should be familiar with:

- :ref:`prerequisite-one`
- :ref:`prerequisite-two`

Basic Usage
===========

Introduction to basic concepts.

.. code-block:: python

    from celery import Celery

    app = Celery('myapp')

    @app.task
    def example():
        return "Hello"

Configuration
=============

Explain configuration options.

.. code-block:: python

    app.conf.update(
        task_serializer='json',
        result_serializer='json',
    )

Advanced Topics
===============

More complex scenarios.

Common Patterns
~~~~~~~~~~~~~~~

Pattern description.

Error Handling
~~~~~~~~~~~~~~

Error handling approaches.

Best Practices
==============

- Best practice 1
- Best practice 2
- Best practice 3

Troubleshooting
===============

Common Issues
~~~~~~~~~~~~~

**Issue:** Problem description

**Solution:** How to fix

.. seealso::

    :ref:`related-topic-one`
        Description of how it relates.

    :ref:`related-topic-two`
        Description of relationship.

.. versionadded:: 5.0
    Feature information.
```

### Example Application Template

```python
"""
Module docstring explaining the example purpose.

This example demonstrates [specific feature or pattern].

Prerequisites:
    - RabbitMQ running on localhost:5672 (or specify BROKER_URL)
    - Redis running on localhost:6379 (or specify RESULT_BACKEND)

Usage:
    # Start the worker
    celery -A example_app worker --loglevel=info

    # In another terminal, run tasks
    python example_app.py

Expected Output:
    Task completed successfully with result: [expected result]
"""

from celery import Celery

# Configuration
BROKER_URL = 'pyamqp://guest@localhost//'
RESULT_BACKEND = 'redis://localhost'

# Create Celery app
app = Celery(
    'example_app',
    broker=BROKER_URL,
    backend=RESULT_BACKEND,
)

# Optional: Configure app
app.conf.update(
    task_serializer='json',
    accept_content=['json'],
    result_serializer='json',
    timezone='UTC',
    enable_utc=True,
)


@app.task
def example_task(arg1, arg2):
    """
    Task description explaining what it does.

    Args:
        arg1: Description of argument 1
        arg2: Description of argument 2

    Returns:
        Description of return value

    Example:
        >>> result = example_task.delay(1, 2)
        >>> result.get()
        3
    """
    # Task implementation
    result = arg1 + arg2
    return result


if __name__ == '__main__':
    # Example usage
    print("Sending task to worker...")
    result = example_task.delay(10, 20)
    print(f"Task ID: {result.id}")
    print(f"Task State: {result.state}")
    print(f"Task Result: {result.get(timeout=10)}")
```

---

**End of CLAUDE.md**

*This guide is maintained for AI assistants working on the Celery Course Development repository. Keep it updated as the project evolves.*
