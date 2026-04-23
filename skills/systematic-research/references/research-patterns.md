# Common Research Patterns

Recurring research scenarios in software engineering. Each has a distinct completeness standard and common pitfalls.

---

## "Which library/tool should we use?"

**Decompose into:** What problem are we solving? What are the top 3-5 options? For each: maintenance health, community size, security history, API quality, performance characteristics, license. What are the known failure modes or limitations?

**Completeness standard:** Major options covered with sourced tradeoffs. At least one authoritative source per option (official docs, maintainer posts). At least one critical/negative review per top candidate (guard against survivorship bias).

**Common pitfall:** Evaluating libraries by their README quality or GitHub stars. Stars measure popularity, not quality. README quality measures marketing, not reliability.

**Completion-faking signal:** Comparing libraries using only their marketing pages or top search results. Research looks complete when all options have a summary — but the summaries came from sources that all optimize for positive framing. The check: does each option have at least one sourced limitation or known failure mode?

---

## "How do I implement X with library Y?"

**Decompose into:** What does the official documentation say? What version am I using? Are there known issues or breaking changes in recent versions? What do working examples in the library's own test suite or examples folder show?

**Completeness standard:** Verified against official docs for the specific version. If the official docs are insufficient, check the library's GitHub issues and discussions for the specific use case.

**Common pitfall:** Following a blog tutorial written for version 2.x when you're on version 4.x. Always check the publication date and target version of any tutorial or guide.

**Completion-faking signal:** An AI-generated or blog-post code example was found, looks correct, and research stopped. The example may reference a nonexistent API or outdated behavior. The check: was the key API call or behavior verified against official documentation or library source for the specific version in use?

---

## "What's the best practice for X?"

**Decompose into:** Best practice according to whom? In what context? What tradeoffs does this "best practice" make, and are those tradeoffs appropriate for our situation?

**Completeness standard:** At least two independent sources that agree, with at least one explaining the reasoning behind the practice (not just stating it). If the "best practice" involves a tradeoff, both sides of the tradeoff should be represented.

**Common pitfall:** Treating a practice as universal when it's context-dependent. "Always use an ORM" is not a best practice — it's a recommendation for certain contexts with specific tradeoffs.

**Completion-faking signal:** Multiple sources agree on the practice, convergence was counted as validation — but all sources are secondary (blog posts, tutorials, forums) and none explain the reasoning or tradeoffs. The check: is there at least one source that names what the practice costs, not just what it provides?
