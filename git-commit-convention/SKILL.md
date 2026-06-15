---
name: git-commit-convention
description: Mandatory rules for writing commit messages for the project.
---

# Git Commit Message Convention

When creating a commit for the project, you **MUST** adhere to the following rules:

1. **Language**: Commit messages must be written in **Vietnamese**.
2. **Affected Menu**: You must clearly log/note the **name of the affected menu** caused by the changes in the commit content (e.g., `=> Menu bị ảnh hưởng: Quản lý danh sách hoạt động cộng đồng`).
3. **Structure**:
   - Commit title (line 1): Follow the format `<type>(<scope>): <short summary>`.
     - `<type>` MUST be one of the standard types: `feat`, `fix`, `chore`, `ci`, `build`, `docs`, `refactor`, `test`, `perf`, `style`. Absolutely DO NOT use non-standard words like `update` or "Cập nhật".
     - `<scope>` (in parentheses) **MUST** be the name of the affected project/package (e.g., `sae-new`, `aq-core-framework`, `core-ui`). If changes affect 2 or more packages, separate them with a comma (e.g., `refactor(sae-new,core-ui): ...`). Absolutely do not use feature folder names as the scope.
     - `<short summary>` **MUST** be written in **Vietnamese** (e.g., `sắp xếp lại menu và xoá file jwtUtils`).
   - Commit body: Explain the changes in more detail (if necessary) and include the note about the affected menu.
4. **Execution Process**: Absolutely **DO NOT** execute `git commit` commands automatically. You are only allowed to generate the commit message content and print it in the chat.
5. **Display Format**: ABSOLUTELY DO NOT generate the commit message as a Terminal command (e.g., `git commit -m "..."`). Please print the ENTIRE commit content (including the Title, an empty line, and the Body) grouped into **ONE single raw text code block**. Do not separate the title and body into different texts, so the user can click Copy once and paste it directly into a Git GUI interface (like Fork, SourceTree) quickly.
6. **Separate Commits for Skills and Code**: If your work involves updates/edits in both the `.agents/skills` directory (guides/rules) and the project's source code directory, you **MUST** generate 2 separate commit messages. One commit specifically for the skills directory (since skills are stored in a separate repo/part), and one commit for the project source code. Absolutely do not merge them into 1 commit message.
