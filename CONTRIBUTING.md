# Contributing Guide

Thank you for helping improve the Qiu Biophysics tutorials. Small contributions are valuable, especially when they make the material clearer for the next student.

## Ways To Contribute

You can contribute by:

- Reporting a problem
- Fixing a typo
- Improving an explanation
- Adding a missing step
- Adding a small example
- Testing a tutorial and sharing what happened

Good student projects for this repository include:

- A new short tutorial in `tutorials/`
- A small worked example connected to an existing tutorial
- A clearer explanation of a confusing step
- A troubleshooting note based on a real problem you solved
- A small public example dataset in `data/`

## Before You Start

If your change is small, such as a typo or one sentence, you can open a pull request directly.

If your change is larger, such as a new tutorial or major rewrite, please open an issue first so maintainers can discuss the plan with you.

## Contribution Workflow

1. Fork this repository to your own GitHub account.
2. Create a new branch with a short name, such as `fix-setup-notes` or `add-python-example`.
3. Make your change.
4. Check that the tutorial still reads clearly from start to finish.
5. Open a pull request.
6. Wait for review and respond to comments.

Please do not push directly to `main`. Use a branch and open a pull request so a maintainer can review the change before it becomes part of the public tutorial collection.

## Adding A New Tutorial

Place new tutorials in the `tutorials/` folder.

Use a clear file name:

```text
07-topic-name.md
```

At the top of the tutorial, include:

```markdown
# Tutorial Title

Author: Your Name

## Goal

What the reader will learn.

## Requirements

Software, accounts, files, or background knowledge needed.
```

After adding the tutorial, update `tutorials/README.md` so readers can find it.

## Adding A New Example

Small examples can go inside the relevant tutorial. If the example needs separate files, place them in a clearly named folder:

```text
tutorials/examples/example-name/
```

Only include files that are needed for the example. Keep example files small.

## Pull Request Checklist

Before submitting, please check:

- The tutorial instructions are clear for a beginner.
- Any code examples have been tested if possible.
- No private or sensitive data has been added.
- Figures and data files are small and necessary.
- New files are placed in the correct folder.

## Writing Style

Please write for a student who is seeing the topic for the first time.

Prefer:

- Short sections
- Concrete examples
- Clear setup instructions
- Plain language
- Explanations of unfamiliar terms

Avoid:

- Assuming advanced background knowledge
- Uploading large files
- Adding private data
- Making unrelated changes in the same pull request

## Review

Maintainers may ask questions or request changes. This is normal. Review is part of making the tutorial stronger and easier for everyone to use.
