# IBM i / RPG Codebase Exploration Prompts

A structured prompt library for exploring an IBM i / RPG codebase from both a **functional** (business domain) and **technical** (architecture and implementation) perspective. Intended for use with AI-assisted coding tools (Roo Code, Claude Code, etc.) or as a guide for manual code review.

---

## 1. Orientation & Inventory

### Functional
- "Give me an overview of the business domains represented in this codebase. What are the main functional areas (e.g. order management, billing, inventory)? Base your answer on program names, file names, and any inline comments."
- "List all programs and service programs, grouped by apparent business function. For each group, summarise in one sentence what it does."
- "Which programs appear to implement the most critical business processes? How can you tell?"

### Technical
- "Inventory all source members by type (RPGLE, SQLRPGLE, CLP, DSPF, PRTF, PF, LF, BNDDIR, SRVPGM). Produce a summary table with counts."
- "Identify the top-level entry programs — those that are called interactively by users or by job schedulers and are not called by other programs in this codebase."
- "List all binding directories and service programs. For each, list the exported procedures it provides."

---

## 2. Data Model & File Usage

### Functional
- "Describe the core business entities in this system based on the physical file (PF) definitions. What are the key tables and what does each represent?"
- "Which files hold master data (e.g. customers, products, chart of accounts) versus transactional data (e.g. orders, journal entries)? How do you distinguish them?"
- "Map the relationships between files: which files reference each other through key fields, and what business relationships do these imply (e.g. order header → order lines → product master)?"

### Technical
- "List all Physical Files (PFs) and their key fields. Flag any files that lack a primary key or use arrival sequence."
- "List all Logical Files (LFs) and map each to its underlying PF. For each LF, describe the access path it provides (select/omit criteria, key sequence changes)."
- "Identify all SQL tables and views defined inline in SQLRPGLE programs versus those defined as DDS source members. Are there overlaps or conflicts?"
- "Which programs open files for UPDATE or DELETE? Highlight any that perform mass updates without a WHERE clause equivalent."
- "Find all uses of `CHAIN`, `READ`, `READE`, `SETLL`, `SETGT` and classify them by the file being accessed. Which files have the most varied access patterns?"

---

## 3. Program Structure & Modularity

### Functional
- "For each major functional area, trace the call chain from the entry program down through called programs and procedures. Produce a call tree diagram."
- "Identify programs that appear to do too much — combining UI logic, business rules, and file I/O in a single source member. What responsibilities could be separated?"
- "Which business rules are implemented more than once across different programs? These are candidates for centralisation into a service program."

### Technical
- "Classify programs by their architectural role: interactive (display file driver), batch (report/update), service program (shared logic), utility (conversion, formatting). How well-separated are these concerns?"
- "List all `CALLP` and `CALL` statements. Distinguish static bound calls (CALLP to a procedure) from dynamic calls (CALL with a variable program name). Where are dynamic calls used and why?"
- "Identify all `/COPY` and `/INCLUDE` directives. What is the role of each copybook — data structures, constants, prototypes, or mixed? Are there copybooks that duplicate definitions from others?"
- "Find all exported procedures in service programs. Which procedures have no callers within this codebase (potential dead code or external API surface)?"
- "How is error handling implemented? Look for `%ERROR`, `MONITOR/ON-ERROR/ENDMON`, `*PSSR`, `INFSR`, and `ERRMSG`/`SNDPGMMSG`. Is the approach consistent?"

---

## 4. Business Logic Deep Dives

### Functional
- "Explain the order-to-cash process as implemented in this code. Walk through each step: order entry, validation, fulfilment, invoicing, and payment posting."
- "How is pricing calculated? Find all programs and procedures that compute a price or apply a discount. Describe the rules they implement."
- "What validation rules are enforced before a transaction is committed? Where are they implemented — in display file record formats, in RPG programs, or in database constraints?"
- "How does the system handle exceptional cases — credit holds, out-of-stock items, negative quantities, or date overrides? Find the relevant code and explain the logic."

### Technical
- "Locate all date/time arithmetic. Are dates handled as numeric fields (YYYYMMDD), `D` type fields, or SQL timestamps? Are there conversions between formats, and are they consistent?"
- "Identify all hard-coded constants (magic numbers, literal strings for status codes, company codes, account numbers). Which of these should be externalised as named constants or data area values?"
- "Find all uses of data areas (`IN`/`OUT`/`*DTAARA`). What state do they hold and which programs share them?"
- "Which programs use `%OPEN`/`%EOF`/`%FOUND` and which rely on implicit file status indicators? Are there programs that ignore file operation errors?"

---

## 5. User Interface & Interactivity

### Functional
- "List all display file source members (DSPFs). For each, describe the screen's business purpose based on field names and any constants visible in the DDS."
- "Trace the user journey through the main menu system. Which screens are reachable, and what transactions do they support?"
- "Which screens perform real-time validation (field-level validation using `CHECK` or program indicators) versus post-submit validation in the RPG program?"

### Technical
- "Identify all subfile (SFL) definitions in DSPF members. For each, describe the subfile load strategy used in the associated RPG program: page-at-a-time, load-all, or expanding."
- "Which display programs use `EXFMT` in a loop versus `READ` on a subfile control record? Are there any programs mixing both patterns?"
- "Find all uses of message subfiles and program message queues. How are user messages constructed and displayed?"

---

## 6. Batch Processing & Scheduling

### Functional
- "Identify all batch programs. For each, describe what business process it automates and at what frequency it is likely to run (end-of-day, month-end, on-demand)."
- "Which batch programs produce printed output (PRTF) or spool files? What reports do they generate and who are the likely consumers?"
- "Are there dependencies between batch jobs — programs that must run in sequence or that share intermediate work files? Document the implied job stream."

### Technical
- "List all CL programs. For each, identify whether it is a job stream controller, a submission wrapper, or an environment setup routine."
- "Find all `SBMJOB` calls. What programs are submitted, and are job parameters (job queue, output queue, library list) hard-coded or parameterised?"
- "Identify work files (temporary PFs created and deleted within a batch run). Are they created with `CRTPF` in a CL, or are they permanent objects reused between runs?"

---

## 7. Integration & External Interfaces

### Functional
- "Does this codebase interface with external systems? Look for file-based imports/exports, socket calls, HTTP service calls, or data queue usage. Describe each integration point and its business purpose."
- "How is data imported from external sources — flat files in IFS, data queues, or direct database writes from another system? What validation is applied to inbound data?"
- "How is data exported — outbound files, printed forms, EDI segments, or API responses? Which downstream systems are implied?"

### Technical
- "Find all IFS access (using `%OPEN` on IFS paths, `QP0LLIB1`, or `QSYS2.IFS_READ_UTF8`). What files are read or written and in what format?"
- "Identify all HTTP or web service calls. Are they using Scott Klement's HTTPAPI, IBM's `QzhbCgiUtils`, or a third-party library?"
- "Find all data queue (`DTAQ`) interactions: `QSNDDTAQ`, `QRCVDTAQ`, or the `DataQueue` class in ILE. What messages are exchanged and with which programs?"
- "Are there any SQL stored procedures, user-defined functions (UDFs), or triggers defined in this codebase? What do they do and who calls them?"

---

## 8. Performance & Scalability

### Functional
- "Which programs are most likely to be performance-sensitive based on the volume of data they process or their frequency of execution?"

### Technical
- "Identify all full-table scans: RPG programs using `READ` in a loop without `SETLL`/`SETGT` positioning, or SQL `SELECT` statements without a `WHERE` clause on a keyed column."
- "Find all SQL `SELECT *` statements. Are there cases where a subset of columns would suffice, reducing I/O?"
- "Identify programs that open and close files repeatedly within a loop. Would keeping files open across iterations improve throughput?"
- "Are there any programs that hold database locks for extended periods — for example, using `UPDATE` without immediate `WRITE`, or `*ALL` lock levels on `USROPN` files?"

---

## 9. Modernisation Readiness

### Functional
- "Which functional areas have the most technical debt based on age indicators (fixed-format RPG, `F`-spec file declarations, `I`-specs, `O`-specs)? Prioritise them for modernisation."
- "Are there business rules embedded in DDS `COMP`, `RANGE`, or `VALUES` keywords that should be moved to the application layer for testability?"

### Technical
- "Classify source members by RPG generation: OPM RPG/400 (no `H`-spec or `OPTION(*SRCSTMT)`), ILE RPG with cycle (`H`-spec present, no `NOMAIN`), or fully free-form (`**FREE` header). What proportion is in each category?"
- "Identify all fixed-format F-specs and D-specs. Which programs are candidates for conversion to free-format using `CVTRPGSRC` or RDi refactoring?"
- "Find all uses of `MOVE`, `MOVEL`, `Z-ADD`, `Z-SUB`, `MVR` — legacy op-codes that should be replaced with modern assignment and built-in functions."
- "Which programs have no procedure boundary (`BEGSR`/`ENDSR` only, no `DCL-PROC`)? These are candidates for refactoring into modular procedure-based designs."
- "Identify all global variables (D-specs at module level outside a procedure). Which of these could be scoped to a procedure to reduce coupling?"
- "Are there existing unit test programs (programs whose names suggest `TEST`, `UT`, or similar conventions)? What is the test coverage story?"

---

## 10. Security & Compliance

### Functional
- "Which programs handle sensitive data — financial amounts, personal information, access credentials? Are there any obvious risks in how this data is processed or displayed?"
- "How is user authorisation enforced? Look for authority checking (`CHKOBJ`, `RVKOBJAUT`), adopted authority (`USRPRF(*OWNER)`), or application-level permission tables."

### Technical
- "Find all programs compiled with `USRPRF(*OWNER)` (adopted authority). Is this necessary for each, or is it over-privileged?"
- "Identify all SQL statements that use string concatenation to build dynamic queries. Flag any that incorporate user input without sanitisation (SQL injection risk)."
- "Are passwords, encryption keys, or connection strings stored in source code, data areas, or user spaces? Identify all occurrences."

---

## Usage Tips

- **Start broad, drill down**: Begin with orientation prompts, then use the call tree and data model results to focus deeper dives on the most critical paths.
- **Combine perspectives**: Pair a functional prompt with its technical counterpart (e.g. "What does the pricing logic do?" + "How is it structured — procedures, copybooks, shared state?").
- **Iterate**: RPG codebases are dense. Use the AI's initial response to formulate follow-up prompts — "Now explain how `CALCULATE_PRICE` handles the volume discount tiers in detail."
- **Ask for diagrams**: Request Mermaid flowcharts or sequence diagrams for call flows and data flows where supported by your tooling.
- **Ground in specifics**: Reference actual member names once discovered — "Explain what `ORDHDRR` does, given it calls `PRCCLC` and reads `ORDHDRPF`."
