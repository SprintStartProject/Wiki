# Naming and Ownership conventions

## Naming

In general, names should not be too long (max. ~60 chars) and concisely specify the document's content. Furthermore, the following
rules should be followed:

1. **Letters should be in lowercase**

   ```text
   ❌ ADR-001-AI-Python-Toolchain
    
   ✅ adr-001-ai-python-toolchain
   ```

2. **Use hyphens instead of whitespaces**

   ```text
   ❌ definition of done.md

   ✅ definition-of-done.md
   ```

3. **Use only letters, digits and hyphens**

   ```text
   ❌ usability-test_(1).md

   ✅ usability-test-1.md
   ```

For ADRs, there is a separate naming convention:

```text
adr-NNN-title.md
```

## Ownership

The person who created the document is responsible for it as its **owner**. If multiple people created the document, one of 
them must be designated as the owner.

The owner is responsible for:

- keeping the document up to date
- reviewing proposed changes
- archiving outdated content

However, this does not imply that only the owner of a document is allowed to change it. All team members may improve documentation;
ownership only defines who is responsible for maintaining accuracy.
