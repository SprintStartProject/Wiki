# How to contribute to the Wiki

## 1. Create a new branch

Before contributing to the wiki, always ensure to create a seperate branch originating from `main`.

## 2. Choose the correct document type

Before starting to actually write the document, you need to define which Diátaxis class your document belongs to.
If you are not sure, there is a selection of document types mapped to their corresponding Diátaxis type:
[document-classification.md](../references/document-classification.md).
This decision is important because it actually helps you understand how to best write your document so that it serves
its purpose.

## 3. Select the correct folder

Choose where to put your document. Usually, corresponding subfolders for common document types should already exist, if this
is not the case, however, you can (provisionally) add a single file or add a new subfolder if more documents of the same type
are likely to be added in the future.

## 4. Look for existing templates

If your document belongs to a commonly used artifact type (e.g. ADRs), a template file might exist (usually stored in
[references](https://github.com/SprintStartProject/Wiki/tree/diataxis-evaluation/docs/source/references)) that you are expected to use. Tutorials or guides that help you writing such a document might also exist.

## 5. Follow naming conventions

Make sure to follow naming conventions when deciding what to name your document. You can find them [here](../references/conventions/naming-and-ownership-conventions.md).

## 6. Create a PR

After finishing writing your document, create a pull request and explain follow-up actions for the assignee(s) (e.g. 
'update index.md'). You are also adviced to directly assign someone from your team as a reviewer; otherwise, the PR might
go unnoticed.
