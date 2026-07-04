# Development Stages

## Why Stages

When I sat down to plan what and how to implement CrestPace, it was overwhelming. The sheer number of tasks, their complexity, and their interconnectedness made it difficult to break the work into issues in Linear that would guide development in a clear, structured way.

The team and I were initially focused on feature-based development. But the number of features we planned and the depth we wanted for each one -- to display strong engineering brilliance and allow deep dives into those systems, recreating different solutions from scratch -- meant that feature-based planning alone wasn't cutting it. Feature-based planning tells you *what* to build, but it does not tell you *when* a component needs to exist relative to everything the user touches.

## The Approach: User-Flow-Driven Stages

The solution was to base development on the user flow. This is not just about the front end. When a user comes to the site, they see a landing page, and from there they need to create an account, sign in, complete KYC, and reach their dashboard. Each of those steps triggers backend work: wallet creation, contract events, schema scaffolding, and more. The user's path through the application naturally defines the order in which the backend must be built.

Structuring work around the user flow makes it possible to focus on one slice of the platform at a time -- thinking clearly about how it works, how the user interacts with it, and how to protect and handle edge cases -- instead of trying to design every feature in isolation and worrying about how it connects to the rest of the platform.

This approach helps build better engineering capability and knowledge about how the system works under the hood, because each stage forces a deep understanding of a specific cross-section of the app.

## Stage Structure

Each stage lives in its own folder under `stages/` and contains four files:

| File | Purpose |
|---|---|
| `features.md` | The features that will be built during this stage |
| `architecture_decisions.md` | The technical decisions and rationale behind those features |
| `user_flow.md` | The expected paths a user takes through this stage |
| `edge_cases.md` | Everything that could go wrong -- error handling, retry logic, security vulnerabilities, and production-grade considerations |

## Stages

| Stage | Folder | Focus |
|---|---|---|
| 1 | [stage1](stages/stage1) | Authentication layer -- sign-in, KYC, account creation |
| 2 | stage2 | *To be defined* |
| 3 | stage3 | *To be defined* |
