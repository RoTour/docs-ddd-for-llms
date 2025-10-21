Aggregates are considered one of the least understood patterns among Domain-Driven Design's (DDD's) tactical tools, but employing certain rules of thumb can simplify their implementation. An Aggregate is defined as a cluster of Entities and Value Objects that must maintain **transactional consistency** throughout its life cycle.

The sources provide specific rules of thumb intended to guide the design of Aggregates:

### Explicit Rules for Defining Aggregates

1.  **Model True Invariants in Consistency Boundaries:** When attempting to discover Aggregates, you must identify the model's **true invariants**. An invariant is a business rule that requires **immediate and atomic transactional consistency**. The Aggregate itself is synonymous with the transactional consistency boundary, meaning everything inside the boundary must be consistent when the transaction commits. A properly designed Aggregate should be modifiable while keeping its invariants completely consistent within a single transaction.
2.  **Design Small Aggregates:** Aggregates should be designed to be small. Limit the Aggregate to just the **Root Entity** and a minimum number of attributes or Value-typed properties—only those necessary, and no more. Designing small Aggregates improves performance, enhances scalability, and biases the system toward transactional success, which makes the system more usable.
3.  **Reference Other Aggregates by Identity:** When one Aggregate needs to associate with another, it should reference the Root of the other Aggregate using its **globally unique identity**, rather than maintaining a direct object reference (or "pointer"). Using references by identity helps ensure that the referenced Aggregate is not incorrectly included within the referencing Aggregate's consistency boundary.
4.  **Use Eventual Consistency Outside the Boundary:** Any business rule or invariant that spans **across multiple Aggregates** is not expected to be instantly up-to-date. If executing a command on one Aggregate instance requires subsequent business logic to execute on one or more other Aggregates, you must use **eventual consistency**. This is typically achieved when the Aggregate command method publishes a **Domain Event** that is then delivered asynchronously to subscribers, which modify the other dependent Aggregates in separate transactions.

### Supplementary Guidance

In addition to the explicit rules, several key implementation and design guidelines apply to defining Aggregates:

*   **Single Aggregate Per Transaction:** The rule of thumb for Aggregates is to modify only a **single Aggregate instance per transaction** in all cases. Modifying multiple instances in a single transaction suggests that your consistency boundaries might be incorrectly defined.
*   **Root Entity:** Model one Entity as the Aggregate Root. The Root Entity must possess a **unique identity**.
*   **Favor Value Objects:** Design Aggregates to favor **Value Object parts** (properties holding a reference to a Value Object) instead of contained Entity parts whenever possible. This reduces overhead, uses less memory, and results in safer, more easily tested components due to immutability.
*   **Information Hiding:** When implementing Aggregates, adhere to the principles of **Law of Demeter** and **Tell, Don’t Ask** to ensure information hiding. The client should tell the Aggregate Root what to do using a command on its public interface, and should not ask the Root for internal parts to execute a command on them.
*   **Dependency Management:** Avoid injecting a Repository or Domain Service directly into an Aggregate instance. Dependencies should preferably be resolved *before* the Aggregate command method is invoked and passed in as parameters.
*   **Determining Consistency Need:** When faced with a complex domain scenario, ask whose job it is to ensure consistency. If it is the job of the user executing the use case, aim for transactional consistency; if it is the job of another user or the system, allow for eventual consistency.

### Reasons to Break the Rules

While adherence is encouraged, an experienced DDD practitioner may break the Aggregate Rules of Thumb only with good reason. These reasons include:

*   **User Interface Convenience:** When multiple Aggregate instances are created in a batch operation and are semantically no different than creating them one at a time.
*   **Lack of Technical Mechanisms:** If the project lacks asynchronous processing capabilities (such as messaging, timers, or background threads) needed for eventual consistency, and user-aggregate affinity is ensured, modifying multiple Aggregates in one transaction might be justified.
*   **Global Transactions:** When strictly required to adhere to global, two-phase commit transactions (due to legacy technologies or enterprise policies).
*   **Query Performance:** Holding direct object references to other Aggregates can sometimes ease Repository query performance issues, but this must be weighed against size and overall performance trade-offs.