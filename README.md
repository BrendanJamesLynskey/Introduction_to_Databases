# 🗄️ Introduction to Databases — Presentation

An interactive slide deck covering relational theory, SQL, transactions, indexing, NoSQL models, and how to choose the right database engine. Aimed at mid-level software engineers.

## ▶ [Open the Presentation](https://YOUR-USERNAME.github.io/YOUR-REPO/)

> **Setup:** Enable GitHub Pages (Settings → Pages → Deploy from `main` branch, `/ (root)` directory), then replace the link above with your GitHub username and repository name.
>
> Alternatively, open `index.html` locally in any browser — the presentation works offline after first load.

---

## Contents

| # | Topic |
|---|-------|
| 01 | Why dedicated databases? |
| 02 | Database taxonomy — RDBMS, NoSQL, NewSQL |
| 03 | The relational model — tables, keys, foreign keys |
| 04 | SQL fundamentals — DDL, DML, DCL, TCL |
| 05 | JOIN types |
| 06 | Normalisation — 1NF through BCNF |
| 07 | ACID transactions |
| 08 | Isolation levels and read phenomena |
| 09 | Indexing — B-tree, hash, GIN, partial, covering |
| 10 | Query planning and EXPLAIN |
| 11 | Concurrency control — MVCC vs pessimistic locking |
| 12 | Storage engine internals — heap pages, WAL, LSM-tree |
| 13 | NoSQL — document databases |
| 14 | NoSQL — key-value and wide-column stores |
| 15 | NoSQL — graph and vector databases |
| 16 | CAP theorem and BASE |
| 17 | Replication and sharding |
| 18 | Schema design patterns — OLTP, star schema, EAV |
| 19 | Migrations and zero-downtime schema evolution |
| 20 | Choosing the right database |
| 21 | Summary and further reading |

---

## Slide Controls

| Action | Key |
|--------|-----|
| Next / Previous | `→` `←` or swipe |
| Overview | `Esc` |
| Fullscreen | `F` |
| Export to PDF | append `?print-pdf` to URL |

---

## Technology

[Reveal.js 4.6](https://revealjs.com) · [highlight.js](https://highlightjs.org) (Monokai) · Playfair Display + DM Sans + JetBrains Mono

Single self-contained `index.html` — no build step, no npm.

---

## References

- Kleppmann, M. *Designing Data-Intensive Applications*. O'Reilly, 2017
- Ramakrishnan, R. & Gehrke, J. *Database Management Systems*, 3rd ed. McGraw-Hill, 2002
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Use The Index, Luke](https://use-the-index-luke.com) — indexing and query optimisation
- Pavlo, A. *CMU 15-445: Database Systems* — [course recordings on YouTube](https://www.youtube.com/channel/UCHnBsf2rH-K7pn09rb3qvkA)
- Codd, E.F. "A Relational Model of Data for Large Shared Data Banks." *CACM*, 1970

## License

Educational use. Code examples provided as-is. Standards references are to publicly available documentation.
