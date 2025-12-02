# Footnotes in python-docx

This documentation covers the footnote functionality available in the custom python-docx fork used by Ghostwriter.

## Overview

Footnotes are annotations that appear at the bottom of a page, referenced by a superscript number in the document body. They are stored in a separate `footnotes.xml` part within the `.docx` file structure (not to be confused with footers, which appear on every page).

```
┌─────────────────────────────────────┐
│  Document body text with a          │
│  reference to a footnote.¹          │
│                                     │
│  More paragraph text here.²         │
│                                     │
│─────────────────────────────────────│ ← Footnote separator
│  ¹ This is the first footnote       │ ← FOOTNOTE (from footnotes.xml)
│  ² This is the second footnote      │
│─────────────────────────────────────│
│  Page 1 of 10  |  Company Name      │ ← FOOTER (from footer.xml)
└─────────────────────────────────────┘
```

## Installation

Ghostwriter uses a local fork of python-docx with footnote support. Ensure the fork is installed in your environment (see development setup documentation).

## Basic Usage

### Creating a Document with Footnotes

```python
from docx import Document

# Create a new document
document = Document()

# Add a paragraph
paragraph = document.add_paragraph("This paragraph has a footnote reference.")

# Add a footnote to the paragraph
footnote = paragraph.add_footnote()
footnote.add_paragraph("This is the footnote text that appears at the bottom of the page.")

# Save the document
document.save("document_with_footnote.docx")
```

### Adding Multiple Footnotes

```python
from docx import Document

document = Document()

# First paragraph with footnote
para1 = document.add_paragraph("Introduction to the topic.")
footnote1 = para1.add_footnote()
footnote1.add_paragraph("Source: Example Citation, 2024.")

# Second paragraph with footnote
para2 = document.add_paragraph("Another important point.")
footnote2 = para2.add_footnote()
footnote2.add_paragraph("Additional reference material.")

# Footnote IDs are automatically assigned sequentially
print(f"First footnote ID: {footnote1.id}")
print(f"Second footnote ID: {footnote2.id}")

document.save("multiple_footnotes.docx")
```

### Footnotes with Multiple Paragraphs

A single footnote can contain multiple paragraphs:

```python
from docx import Document

document = Document()

paragraph = document.add_paragraph("Complex topic requiring detailed footnote.")

footnote = paragraph.add_footnote()
footnote.add_paragraph("First paragraph of the footnote with initial explanation.")
footnote.add_paragraph("Second paragraph with additional details and context.")
footnote.add_paragraph("Third paragraph with concluding remarks.")

print(f"This footnote has {len(footnote.paragraphs)} paragraphs")

document.save("multi_paragraph_footnote.docx")
```

## Reading Footnotes

### Accessing Footnotes from a Paragraph

```python
from docx import Document

# Open an existing document
doc = Document("existing_document.docx")

# Get the first paragraph
paragraph = doc.paragraphs[0]

# Access footnotes referenced in this paragraph
footnotes = paragraph.footnotes

for footnote in footnotes:
    print(f"Footnote ID: {footnote.id}")
    print(f"Number of paragraphs: {len(footnote.paragraphs)}")
    for para in footnote.paragraphs:
        print(f"  - {para.text}")
```

### Accessing All Footnotes in a Document

```python
from docx import Document

doc = Document("existing_document.docx")

# Access all footnotes in the document
# Note: The list index corresponds to the footnote reference ID
for footnote in doc.footnotes:
    print(f"Footnote ID: {footnote.id}")
    print(f"Paragraphs: {len(footnote.paragraphs)}")
    for para in footnote.paragraphs:
        print(f"  Content: {para.text}")
```

> **Note:** The `doc.footnotes` collection may include separator footnotes (IDs 0 and 1) that Word uses internally. User-created footnotes typically start at ID 2.

## Footnote Properties (Section-Level)

Footnote formatting properties are configured at the section level:

```python
from docx import Document

doc = Document("existing_document.docx")

# Access the first section
section = doc.sections[0]

# Read footnote properties
print(f"Position: {section.footnote_position}")
print(f"Number format: {section.footnote_number_format}")
print(f"Starting value: {section.footnote_numbering_start_value}")
print(f"Restart location: {section.footnote_numbering_restart_location}")

# Modify footnote properties
section.footnote_position = "beneathText"  # or "pageBottom"
section.footnote_number_format = "decimal"  # or "lowerLetter", "upperLetter", "lowerRoman", "upperRoman"
section.footnote_numbering_start_value = 1
section.footnote_numbering_restart_location = "continuous"  # or "eachPage", "eachSection"

doc.save("modified_footnote_settings.docx")
```

### Available Footnote Properties

| Property | Description | Example Values |
|----------|-------------|----------------|
| `footnote_position` | Where footnotes appear | `"pageBottom"`, `"beneathText"` |
| `footnote_number_format` | Numbering style | `"decimal"`, `"lowerLetter"`, `"upperLetter"`, `"lowerRoman"`, `"upperRoman"`, `"hex"` |
| `footnote_numbering_start_value` | Starting number | `1`, `2`, etc. |
| `footnote_numbering_restart_location` | When to restart numbering | `"continuous"`, `"eachPage"`, `"eachSection"` |

## Integration with Ghostwriter Reports

### Example: Adding Citations as Footnotes

```python
from docx import Document

def add_finding_with_citation(document, finding_text, citation_text):
    """Add a finding paragraph with a citation footnote."""
    paragraph = document.add_paragraph(finding_text)
    if citation_text:
        footnote = paragraph.add_footnote()
        footnote.add_paragraph(citation_text)
    return paragraph

# Usage in report generation
document = Document()

add_finding_with_citation(
    document,
    "The application was found to be vulnerable to SQL injection.",
    "OWASP Top 10 2021 - A03:2021 Injection"
)

add_finding_with_citation(
    document,
    "Authentication tokens were transmitted over unencrypted channels.",
    "NIST SP 800-63B Section 5.1.3"
)

document.save("report_with_citations.docx")
```

### Example: Processing Existing Documents

```python
from docx import Document

def extract_all_footnotes(docx_path):
    """Extract all footnotes from a document."""
    doc = Document(docx_path)
    footnotes_data = []

    for footnote in doc.footnotes:
        # Skip separator footnotes (typically IDs 0 and 1)
        if footnote.id < 2:
            continue

        footnote_text = "\n".join(para.text for para in footnote.paragraphs)
        footnotes_data.append({
            "id": footnote.id,
            "text": footnote_text,
            "paragraph_count": len(footnote.paragraphs)
        })

    return footnotes_data

# Usage
footnotes = extract_all_footnotes("report.docx")
for fn in footnotes:
    print(f"[{fn['id']}] {fn['text']}")
```

## Troubleshooting

### Common Issues

1. **Footnote count includes extra items**
   - Word documents include separator footnotes (IDs 0 and 1) by default
   - User-created footnotes start at ID 2
   - Filter by ID when counting user footnotes: `[fn for fn in doc.footnotes if fn.id >= 2]`

2. **Footnotes not appearing in document**
   - Ensure you call `footnote.add_paragraph()` after creating the footnote
   - A footnote without paragraphs will not render content

3. **Footnote reference not visible**
   - The superscript reference is automatically added when you call `paragraph.add_footnote()`
   - Check that the paragraph has text before the footnote reference

### Debugging

```python
from docx import Document

doc = Document("problematic_document.docx")

# List all footnotes with details
print(f"Total footnotes in document: {len(doc.footnotes)}")

for i, footnote in enumerate(doc.footnotes):
    print(f"\nFootnote index {i}:")
    print(f"  ID: {footnote.id}")
    print(f"  Paragraphs: {len(footnote.paragraphs)}")
    for j, para in enumerate(footnote.paragraphs):
        print(f"    Para {j}: '{para.text[:50]}...' " if len(para.text) > 50 else f"    Para {j}: '{para.text}'")
```

## API Reference

### Document

| Method/Property | Description |
|-----------------|-------------|
| `document.footnotes` | Returns list of all footnotes in the document |

### Paragraph

| Method/Property | Description |
|-----------------|-------------|
| `paragraph.add_footnote()` | Creates a new footnote and adds reference to paragraph |
| `paragraph.footnotes` | Returns list of footnotes referenced in this paragraph |

### Footnote

| Method/Property | Description |
|-----------------|-------------|
| `footnote.id` | The footnote reference ID (integer) |
| `footnote.paragraphs` | List of paragraphs in the footnote |
| `footnote.add_paragraph(text)` | Adds a paragraph to the footnote |

### Section

| Property | Description |
|----------|-------------|
| `section.footnote_position` | Position of footnotes on page |
| `section.footnote_number_format` | Numbering format style |
| `section.footnote_numbering_start_value` | Starting number for footnotes |
| `section.footnote_numbering_restart_location` | When numbering restarts |
