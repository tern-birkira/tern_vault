### Commit changes

Do some work, commit it, do some more work, commit that.

Learn the following commands:

`git status # Show modified and new files git add # Add files to the ongoing commit git commit # Commit it. Use the EDITOR environment variable to choose the editor.`

### Best Practices

Git commits must include a summary of the work done in English.

1st line: Brief title of the change. Try to describe the changes made rather then stating what task was implemented or which bug number was fixed.

2nd line: space

3rd line and more: should have the full description of why and what and if needed the how. Including the ticket number. e.g. DEV-5380

**Be Concise**: Keep the commit messages as concise as possible while providing enough context to understand the change.

**Use Imperative Mood**: Write the subject line as if you're giving a command (e.g., "fix bug" instead of "fixed bug").

**Explain Why**: If the commit is not self-explanatory, explain why the change was made.

**Separate Logic and Style Changes**: Avoid mixing code changes and style changes in the same commit.

**Reference Issues**: Reference related issues or tasks in the footer.

**Review Before Committing**: Review your commit message before finalizing the commit to ensure clarity and completeness.

#### Examples

**Good Commit Message**:

`Handle null values in response Prevent the API from returning null values in the response. Added a check to ensure all fields are populated with default values if they are null. Fixes DEV-5380`

**Bad Commit Messages**:

`minor fixes`

`refactored code`