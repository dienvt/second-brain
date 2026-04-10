# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is an **Obsidian knowledge vault** designed as a **RAG-ready knowledge base**. It contains curated wiki notes, raw imported documents, and attachments. Version control is handled by the **obsidian-git** plugin with automated "vault backup" commits.

## Structure

```
Wiki/                          # Curated knowledge (RAG-ready, clean markdown)
  Engineering/                 # Technical knowledge by topic
    Golang/, Java/, Ruby/      # Programming languages
    Software Architecture/     # Architecture patterns
    Microservices/             # Distributed systems
    Algorithm/, Data structure/# CS fundamentals
    HTTP Certification/, Internet/, Proxy/ # Networking
    OS/, Storage/              # Systems
    Apache Kafka/, Kafka/      # Messaging
    Agile/, elastic/, TiDB/    # Tools & practices
    Paradigm/ (FP, OOP)       # Programming paradigms
    Front-End/, ReactJs/, ReactiveX/ # Frontend
    PM2/, Interface/, regex/, window/ # Misc tech
  Career/                      # Career path, management roadmap
  Projects/                    # Project documentation
  Work/                        # Work-related notes
  Personal/                    # Life wiki, reading, quotes

Raw/                           # Unprocessed input documents
  Notion-Import/               # 533 notes bulk-imported from Notion
    Go/, Blog/, Personal Home/, Troodon/,
    Career Path/, Enjoy Sport/, VNG Works/
  Documents/                   # PDFs and raw reference docs

Attachments/                   # All images, media, canvas files
Templates/                     # Note templates (Wiki Note, Raw Document)
```

## RAG Workflow

- **Wiki/** is the curated knowledge base — notes here should be clean, well-structured, and RAG-optimized
- **Raw/** holds unprocessed documents to be ingested and refined into Wiki notes
- Raw notes use `status: unprocessed | processing | processed` frontmatter to track processing state
- New wiki notes go in Wiki/ by default (configured in Obsidian settings)
- New attachments go in Attachments/ by default

## Obsidian Configuration

- Config lives in `.obsidian/` — avoid modifying directly
- Community plugins: `obsidian-importer`, `obsidian-git`
- Settings: auto-update links enabled, attachments → `Attachments/`, new files → `Wiki/`

## Working With This Vault

- Notes use Obsidian-flavored markdown: `[[wikilinks]]`, `![[embeds]]`, callouts, and frontmatter
- Many notes in `Raw/Notion-Import/` were bulk-imported from Notion and may have formatting artifacts
- The vault has no build system, linter, or test suite
- Git branch is `obsidiant`; commits follow the pattern `vault backup: YYYY-MM-DD HH:MM:SS`
