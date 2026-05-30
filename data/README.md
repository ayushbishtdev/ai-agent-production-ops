# data/ — Column Definitions and Sources

## `platform-tco-comparison.csv`

Structured comparison of the three dominant AI agent orchestration platforms — Camunda, LangGraph, and CrewAI — across 13 total cost of ownership (TCO) dimensions.

### Column Definitions

| Column | Description |
|---|---|
| `dimension` | The specific TCO or capability dimension being evaluated |
| `camunda` | Assessment for Camunda (enterprise BPMN orchestration engine) |
| `langgraph` | Assessment for LangGraph (graph-based agentic framework by LangChain) |
| `crewai` | Assessment for CrewAI (role-based multi-agent framework) |
| `notes` | Source attribution, important caveats, or context for the row |

### Methodology

Assessments are drawn from technical analysis published in May 2026 at agileleadershipdayindia.org, specifically the article [Camunda vs LangGraph vs CrewAI: One Will Bankrupt You](https://agileleadershipdayindia.org/blogs/camunda-vs-langgraph-vs-crewai-orchestration/camunda-vs-langgraph-vs-crewai-orchestration.html).

**What "TCO" means here:** The total cost of ownership includes software licensing, engineering hours required to build features the framework doesn't include (observability, governance, state management, retry logic), and ongoing maintenance burden — not just license fees. A zero-license-cost framework with a 6-month governance build is not low-TCO.

### Key Figures to Use With Caution

- **CrewAI 50-agent ceiling:** The ~50 concurrent agent limit is based on observed production behaviour as of May 2026. CrewAI is actively developed; verify against the current release before using this figure in an architecture decision record.
- **Camunda enterprise pricing:** The "~$10K+/yr" figure is a floor estimate for enterprise SaaS. Actual pricing depends on process instances, cluster size, and negotiated enterprise agreements. Verify directly with Camunda.
- **LangSmith fees:** LangChain pricing changes frequently. The "usage-based fees" note reflects the May 2026 model; check docs.langchain.com for current pricing.

### Platforms Not in This Dataset

- **IBM watsonx Orchestrate** — enterprise orchestration with strong IBM ecosystem integration; not covered in the source articles
- **Salesforce Agentforce** — CRM-native orchestrator; only appropriate when all agents operate within the Salesforce ecosystem
- **Microsoft Copilot Studio** — Microsoft 365-native; similar ecosystem-lock constraints to Agentforce
- **AutoGen** — Microsoft's multi-agent research framework; production maturity as of May 2026 is below LangGraph and Camunda

### Update History

| Date | Change |
|---|---|
| May 2026 | Initial version — 13 dimensions across 3 platforms |

To submit corrections or additions, open an issue at the repository with a public source link.
