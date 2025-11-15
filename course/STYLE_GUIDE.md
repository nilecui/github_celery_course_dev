# Celery Course Development - Style Guide

**Last Updated:** 2025-11-15
**Purpose:** Ensure consistency, quality, and professionalism across all course materials

---

## Document Structure

### 1. Introduction
This style guide provides standards for all content created for the Celery Course Development Platform. All contributors MUST follow these guidelines to maintain consistency and quality.

### 2. Document Hierarchy
```
Course Materials/
├── Main Documents (README, OUTLINE, SPECIFICATIONS)
├── Module Content (foundation/, professional/, expert/)
├── Assessments (quizzes, exercises, projects)
├── Interactive Components (labs, simulators)
└── Supporting Documentation (guides, references)
```

---

## Writing Standards

### Tone and Voice
- **Professional yet Approachable:** Write in a professional tone that remains accessible to learners
- **Encouraging:** Use positive language that motivates learners
- **Clear and Direct:** Avoid ambiguity; be explicit in instructions
- **Inclusive:** Use gender-neutral language and avoid cultural biases

### Language Style
- **Active Voice:** Prefer active over passive voice
  - Good: "Create a Celery application using the decorator"
  - Bad: "A Celery application should be created by you"
- **Present Tense:** Use present tense for concepts and procedures
- **Second Person:** Address the reader directly as "you"
- **Consistent Terminology:** Use terms consistently across all materials

---

## Formatting Standards

### Headers and Sections
```markdown
# Main Title (Level 1)
## Section Title (Level 2)
### Subsection Title (Level 3)
#### Detailed Topic (Level 4)
```

### Code Blocks
```python
# Always include language identifier
from celery import Celery

app = Celery('myapp')

@app.task
def example_task():
    return "Hello World"
```

### Emphasis and Highlights
- **Bold:** For key terms and important concepts
- *Italics:* For emphasis and foreign terms
- `Code Format:` For inline code and configuration values
- **Warning:** Use for critical information
- **Note:** Use for supplementary information

### Lists
- Numbered lists for sequential steps
- Bulleted lists for non-sequential items
- Maintain parallel structure in list items

---

## Technical Content Standards

### Code Examples
1. **Complete and Runnable:** All code examples must be complete and functional
2. **Comments:** Include explanatory comments for complex code
3. **Error Handling:** Show proper error handling patterns
4. **Best Practices:** Demonstrate current best practices
5. **Version Specific:** Note Python/Celery version requirements

### Configuration Examples
```yaml
# Always provide context and explanations
version: '3.8'
services:
  celery-worker:
    image: celery:latest
    # Command explanation
    command: celery -A tasks worker --loglevel=info
```

### Commands and Shell Instructions
```bash
# Include expected output or explanation
$ celery -A tasks worker --loglevel=info
# Output: [2025-01-01 12:00:00,000: INFO/MainProcess] Connected to redis://localhost:6379
```

---

## Assessment Standards

### Quiz Questions
1. **Clear Wording:** Questions must be unambiguous
2. **Single Correct Answer:** Multiple choice should have one clearly correct answer
3. **Distractors:** Plausible but incorrect options
4. **Explanations:** Provide explanations for correct answers

### Exercise Structure
- **Learning Objectives:** Clear statement of what students will achieve
- **Prerequisites:** List any required knowledge or setup
- **Step-by-Step Instructions:** Numbered or sequential steps
- **Success Criteria:** Clear definition of completion
- **Expected Output:** Show what students should see

### Project Requirements
- **Problem Statement:** Clear description of the challenge
- **Functional Requirements:** Specific features to implement
- **Evaluation Criteria:** How projects will be assessed
- **Deliverables:** What students must submit

---

## Visual Standards

### Diagrams and Charts
- **Consistent Style:** Use the same style for similar diagrams
- **Clear Labels:** All components must be clearly labeled
- **Legible Text:** Minimum font size for readability
- **Alt Text:** Provide descriptions for accessibility

### Tables
| Column Header | Column Header | Column Header |
|---------------|---------------|---------------|
| Row 1 Data    | Row 1 Data    | Row 1 Data    |
| Row 2 Data    | Row 2 Data    | Row 2 Data    |

### Warning and Note Boxes
```markdown
> **Note:** This is important supplementary information that students should know.

> **Warning:** Critical information that could cause errors or security issues if ignored.
```

---

## File Naming Conventions

### Documentation Files
- Use `UPPER_CASE_WITH_UNDERSCORES.md` for main documents
- Use `lowercase-with-dashes.md` for guides and tutorials
- Include version numbers in README: `README.md` (no version)

### Code Examples
- `example-feature.py` for standalone examples
- `tutorial_name.py` for tutorial files
- Use descriptive names that indicate purpose

### Directory Structure
```
module-number-topic/
├── README.md
├── examples/
├── exercises/
└── solutions/
```

---

## Quality Checklist

### Before Publishing
1. **Content Review**
   - [ ] All learning objectives are addressed
   - [ ] Content flows logically
   - [ ] Technical accuracy verified
   - [ ] Examples tested and working

2. **Format Review**
   - [ ] Consistent formatting applied
   - [ ] All links work correctly
   - [ ] Code blocks properly formatted
   - [ ] Spelling and grammar checked

3. **Accessibility Review**
   - [ ] Alt text for images
   - [ ] Sufficient color contrast
   - [ ] Clear headings structure
   - [ ] Descriptive link text

4. **Technical Review**
   - [ ] Code follows PEP 8
   - [ ] Security best practices
   - [ ] Performance considerations
   - [ ] Error handling included

---

## Specific Guidelines

### Introduction Sections
Each module MUST include:
- Overview paragraph explaining the module's purpose
- Learning objectives (3-5 bullet points)
- Prerequisites (if any)
- Estimated time commitment

### Code Documentation
```python
def process_data(data, options=None):
    """
    Process incoming data according to specified options.

    Args:
        data (dict): Input data to process
        options (dict, optional): Processing options. Defaults to None.

    Returns:
        dict: Processed data with results

    Raises:
        ValueError: If data format is invalid

    Example:
        >>> result = process_data({'key': 'value'})
        >>> print(result['status'])
        'success'
    """
```

### Step-by-Step Instructions
1. Use numbered lists for sequential steps
2. Start each step with an action verb
3. Include the expected result after critical steps
4. Provide troubleshooting tips for common issues

### Cross-References
- Use relative paths for internal links
- Reference specific sections: `See "Task Configuration" section`
- Link to external resources with descriptive text

---

## Common Errors to Avoid

1. **Vague Instructions**
   - Bad: "Configure the application"
   - Good: "Set the broker URL in celeryconfig.py to 'redis://localhost:6379'"

2. **Incomplete Code**
   - Always include imports
   - Show complete class/function definitions
   - Include error handling

3. **Inconsistent Terminology**
   - Choose one term and stick with it
   - Document term definitions in glossary
   - Update all occurrences when changing terms

4. **Missing Context**
   - Explain why something is done, not just how
   - Provide background for complex topics
   - Relate concepts to real-world applications

---

## Review Process

1. **Self-Review**
   - Check against this style guide
   - Test all code examples
   - Verify all links

2. **Peer Review**
   - Have another team member review content
   - Check for clarity and accuracy
   - Verify learning objectives are met

3. **Technical Review**
   - Code review by experienced developer
   - Security review for production-related content
   - Performance review for optimization content

4. **Final Approval**
   - Editor-in-chief approval
   - Ensure all checklist items complete
   - Verify version control documentation

---

## Updates and Maintenance

- Review and update style guide quarterly
- Document any deviations from standards
- Collect feedback from students and instructors
- Continuously improve based on usage patterns

---

## Conclusion

Following this style guide ensures high-quality, consistent, and professional course materials that provide the best learning experience for students. All contributors are responsible for understanding and implementing these standards.

**Remember:** Quality is not an act, it's a habit. Maintain these standards in every piece of content you create.