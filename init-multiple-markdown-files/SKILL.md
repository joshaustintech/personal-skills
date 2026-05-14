---
name: init-multiple-markdown-files
description: >
  Initializes a repo's agent-instruction docs by splitting content across multiple Markdown files
  (max 200 lines each) with CLAUDE.md as a table of contents. For multi-project repos, creates a
  CLAUDE.md per subproject and links them from the root CLAUDE.md.
---
# Main instruction

Treat CLAUDE.md as a table of contents, with additional markdown split into multiple Markdown files not exceeding 200
lines of code. Each link to a Markdown file should describe when you want to read it and when you don't want to read it
so that Claude doesn't load too many Markdown files up front. The final instruction in CLAUDE.md should be "Be brief."

# For multi-project repos ONLY
For each subproject, create or update a CLAUDE.md file with more details about the subproject. A subproject contains a
build or execution configuration file (such as pom.xml, build.gradle, package.json, *.gemspec, Gemfile, etc.).

Then update the project root @CLAUDE.md to be a table of contents pointing to each subproject's respective CLAUDE.md
file.