# Working-Model Slices

A working model is the smallest evidence-backed account of the current system that can locate the relevant behavior and constrain a later change. It describes what exists; it does not choose the implementation.

| Slice | Establish | Enough when |
|---|---|---|
| Task boundary | Behavior under investigation and explicit non-goals | relevant and irrelevant behavior can be separated |
| Current design | Responsible modules, boundaries, and dependency direction | ownership and collaboration are explainable |
| Runtime path | One concrete entry-to-effect path, including important branches | the behavior-producing decisions and side effects are traced |
| Behavior owner | The code or boundary that makes the relevant decision or mutation | a reliable change point is distinguished from wrappers and callers |
| Data and state | Sources, shapes, transformations, defaults, persistence, and controls | values that determine behavior are accounted for |
| Contracts and consumers | Observable assumptions at internal, public, and persisted boundaries | affected consumers are named or explicitly bounded |
| Design intent | Documented rationale or the strongest supported explanation for the design | intent is labeled documented, inferred, conflicting, or unknown |
| Verification surface | Tests, checks, logs, or reproduction paths that expose current behavior | later work has an observable way to prove or preserve behavior |
| Unknowns | Missing evidence that could alter the model | each unknown is resolved, bounded, or marked blocking |

Do not force every slice. Deepen one only when the missing evidence could change the behavior owner, contract, intent, impact boundary, or verification surface. Stop when those conclusions are stable enough for implementation judgment to begin.

The resulting model may identify a reliable change point, but it must not choose a patch, abstraction, refactor, migration, or dependency redesign.
