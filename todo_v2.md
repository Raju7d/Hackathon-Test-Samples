# 🤖 Agentic AI CI/CD Debugger - Team Task Document

**Project:**  An Agentic AI System for CI/CD Pipelines & Production Code Auto-Diagnosis, Auto-Fixing, and Human-Approved Deployment
**Status:** In Development (Phase 2)  
**Last Updated:** November 29, 2025  
**Team:** Ops-timus-Prime
---------------------------------------------------------------------------------------------------

## 📋 Executive Summary

We are building an **Agentic AI CI/CD Debugger** that automates the entire process of diagnosing pipeline errors, fixing code, and submitting pull requests with **human-in-the-loop approval** at critical checkpoints.

### Key Differentiators
- ✅ **Fully Agentic:** Multi-agent workflow with LangGraph state management
- ✅ **Smart Code Analysis:** Extracts repo structure, identifies affected files
- ✅ **Human Oversight:** Approval gates before GitHub operations
- ✅ **End-to-End:** Error logs → Fixed code → GitHub PR (completely automated)

---

## 🏗️ System Architecture

### Level 1: High-Level Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                         USER INTERFACE                               │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ 1. Upload Pipeline Error Logs (text file)                   │  │
│  │ 2. System automatically diagnoses and fixes                 │  │
│  │ 3. Human approves branch creation                           │  │
│  │ 4. Human approves pull request                              │  │
│  │ 5. PR is created (automated)                                │  │
│  └──────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    AGENTIC WORKFLOW (LangGraph)                      │
│                                                                      │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐ │
│  │ Error Diagnosis  │→ │ Repository Search│→ │ Code Analysis    │ │
│  │ Agent            │  │ Agent            │  │ Agent            │ │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘ │
│           │                    │                     │              │
│           └────────────────────┴─────────────────────┘              │
│                                  │                                  │
│                                  ▼                                  │
│                    ┌──────────────────────────┐                     │
│                    │ AI Fix Generation Agent  │                     │
│                    │ (Gemini LLM)             │                     │
│                    └──────────────────────────┘                     │
│                                  │                                  │
│                                  ▼                                  │
│                    ┌──────────────────────────┐                     │
│                    │ Human Approval Gate #1   │                     │
│                    │ (Branch Creation)        │                     │
│                    └──────────────────────────┘                     │
│                                  │                                  │
│                    ┌─────────────┴──────────────┐                   │
│                    │ (Approved)                 │ (Rejected)        │
│                    ▼                            ▼                   │
│        ┌──────────────────────┐      ┌──────────────────┐          │
│        │ GitHub Agent: Create │      │ Exit Gracefully  │          │
│        │ Branch & Commit Code │      │ (Log rejection)  │          │
│        └──────────────────────┘      └──────────────────┘          │
│                    │                                                │
│                    ▼                                                │
│        ┌──────────────────────┐                                     │
│        │ Human Approval Gate #2│                                    │
│        │ (Pull Request)        │                                    │
│        └──────────────────────┘                                     │
│                    │                                                │
│        ┌───────────┴──────────────┐                                 │
│        │ (Approved)               │ (Rejected)                      │
│        ▼                          ▼                                 │
│   ┌─────────────┐      ┌──────────────────────┐                    │
│   │ GitHub Agent│      │ Exit, Keep Branch    │                    │
│   │ Create PR   │      │ (User can merge PRs) │                    │
│   └─────────────┘      └──────────────────────┘                    │
│        │                                                            │
│        ▼                                                            │
│   ┌─────────────────────────────────────────┐                      │
│   │ ✅ COMPLETE: PR Created & Ready to Merge │                      │
│   └─────────────────────────────────────────┘                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

### Level 2: Agent Interactions (LangGraph State Machine)

```
START
  │
  ▼
┌──────────────────────────────────────────────┐
│ STATE: uploaded_error_logs                   │
│ • User uploads: pipeline_errors.txt          │
│ • Parse error messages                       │
│ • Extract error types (version, SKU, etc)    │
└──────────────────────────┬───────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────┐
│ AGENT: ErrorDiagnosisAgent                   │
│ • Analyze error logs                         │
│ • Extract error categories                   │
│ • Identify affected services/modules         │
│ • Recommend repo & file paths                │
└──────────────────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────┐
│ STATE: error_diagnosed                       │
│ • error_type: "InvalidSKU" / "VersionMismatch" │
│ • affected_services: ["functionapp", "keyvault"] │
│ • suggested_files: ["main.tf", "variables.tf"] │
└──────────────────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────┐
│ AGENT: RepositorySearchAgent                 │
│ • Connect to GitHub API                      │
│ • Locate repository (from error context)     │
│ • Fetch file structure                       │
│ • Identify relevant files                    │
└──────────────────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────┐
│ STATE: repo_analyzed                         │
│ • repo_path: "Raju7d/Ops-timus-Prime"       │
│ • main_branch: "main"                        │
│ • file_contents: {                           │
│     "Function APP/infra/main.tf": "...",     │
│     "Function APP/infra/variables.tf": "..." │
│   }                                          │
└──────────────────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────┐
│ AGENT: CodeAnalysisAgent                     │
│ • Analyze current code                       │
│ • Map error to specific code lines           │
│ • Determine fix scope (1 or N files)         │
│ • Extract file sections needing changes      │
└──────────────────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────┐
│ STATE: code_analyzed                         │
│ • files_to_fix: ["main.tf"]                  │
│ • error_lines: [12, 45, 78]                  │
│ • fix_scope: "specific_sections"             │
│ • code_sections: { ... }                     │
└──────────────────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────┐
│ AGENT: AIFixGenerationAgent                  │
│ • Send to Gemini LLM:                        │
│   - Error analysis                           │
│   - Current code                             │
│   - Specific sections needing fixes          │
│ • Generate corrected code                    │
│ • Validate Terraform syntax                  │
└──────────────────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────┐
│ STATE: code_fixed                            │
│ • fixed_files: {                             │
│     "main.tf": "...[fixed code]..."          │
│   }                                          │
│ • fix_summary: "Updated SKU from v2 to v3"  │
└──────────────────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────┐
│ HUMAN APPROVAL GATE #1                       │
│ ┌──────────────────────────────────────────┐ │
│ │ Summary of Changes:                      │ │
│ │ ✓ File: main.tf                          │ │
│ │ ✓ Changes: Update app_service_plan SKU   │ │
│ │ ✓ Error Fixed: InvalidSKU validation     │ │
│ │                                          │ │
│ │ [APPROVE] [REJECT]                       │ │
│ └──────────────────────────────────────────┘ │
└──────────────────────────────────────────────┘
                    │              │
         ┌──────────┘              └─────────────┐
         │ (YES)                                 │ (NO)
         ▼                                       ▼
┌───────────────────────┐          ┌──────────────────┐
│ STATE: approved_branch│          │ STATE: rejected   │
│                       │          │ Exit Workflow     │
└───────┬───────────────┘          └──────────────────┘
        │
        ▼
┌──────────────────────────────────────────────┐
│ AGENT: GitHubBranchAgent                     │
│ • Connect to GitHub                          │
│ • Create new branch from main                │
│   - Branch name: raju_<uuid>                 │
│ • Commit fixed files                         │
│ • Message: "🤖 AI: Fix [error_type]"        │
└──────────────────────────────────────────────┘
        │
        ▼
┌──────────────────────────────────────────────┐
│ STATE: branch_created_committed              │
│ • new_branch: "raju_a1b2"                    │
│ • commit_sha: "abc123def456"                 │
│ • files_committed: ["main.tf"]               │
└──────────────────────────────────────────────┘
        │
        ▼
┌──────────────────────────────────────────────┐
│ HUMAN APPROVAL GATE #2                       │
│ ┌──────────────────────────────────────────┐ │
│ │ Branch: raju_a1b2                        │ │
│ │ Commit: abc123def456                     │ │
│ │ Files: main.tf (45 changes)              │ │
│ │                                          │ │
│ │ Ready to create Pull Request?             │ │
│ │ [APPROVE PR] [REJECT]                    │ │
│ └──────────────────────────────────────────┘ │
└──────────────────────────────────────────────┘
                    │              │
         ┌──────────┘              └─────────────┐
         │ (YES)                                 │ (NO)
         ▼                                       ▼
┌────────────────────┐              ┌──────────────────┐
│ STATE: approved_pr │              │ STATE: rejected   │
│                    │              │ Branch kept live  │
└────────┬───────────┘              │ Exit Workflow     │
         │                          └──────────────────┘
         ▼
┌──────────────────────────────────────────────┐
│ AGENT: GitHubPRAgent                         │
│ • Create Pull Request                        │
│ • Title: "🤖 AI: Fix [error_type]"           │
│ • Body: Includes:                            │
│   - Error analysis                           │
│   - Files changed                            │
│   - Validation results                       │
│   - Suggested reviewers                      │
│ • Set labels: ["ai-generated", "auto-fix"]   │
└──────────────────────────────────────────────┘
         │
         ▼
┌──────────────────────────────────────────────┐
│ STATE: pr_created                            │
│ • pr_url: "https://github.com/.../pull/123" │
│ • pr_number: 123                             │
│ • status: "open"                             │
└──────────────────────────────────────────────┘
         │
         ▼
    ✅ END
    (PR Ready for review & merge)
```

---

### Level 3: Component Interactions

```
┌────────────────────────────────────────────────────────────────┐
│                     USER / UPLOAD INTERFACE                    │
│  • File upload: error_logs.txt                                 │
│  • Dashboard: Show workflow progress                           │
│  • Approval UI: Accept/Reject at checkpoints                   │
└───────────────────────┬──────────────────────────────────────┘
                        │
                        ▼
┌────────────────────────────────────────────────────────────────┐
│                    LANGGRAPH ORCHESTRATOR                       │
│  • Manages workflow state                                      │
│  • Routes between agents                                       │
│  • Handles human interrupts                                    │
│  • Logs all actions                                            │
└────┬──────────────┬──────────────┬──────────────┬─────────────┘
     │              │              │              │
     ▼              ▼              ▼              ▼
┌─────────┐ ┌──────────────┐ ┌──────────┐ ┌──────────┐
│ Error   │ │ Repository  │ │ Code     │ │ AI Fix   │
│ Diag.   │ │ Search      │ │ Analysis │ │ Gen.     │
│ Agent   │ │ Agent       │ │ Agent    │ │ Agent    │
└─────────┘ └──────────────┘ └──────────┘ └──────────┘
     │              │              │              │
     └──────────────┴──────────────┴──────────────┘
                    │
    ┌───────────────┼───────────────┐
    │               │               │
    ▼               ▼               ▼
┌─────────┐ ┌──────────────┐ ┌──────────┐
│ GitHub  │ │ Human        │ │ LLM      │
│ Agent   │ │ Approval     │ │ Service  │
│         │ │ Manager      │ │ (Gemini) │
└─────────┘ └──────────────┘ └──────────┘
    │               │               │
    └───────────────┴───────────────┘
                    │
                    ▼
        ┌──────────────────────┐
        │ Logging & Monitoring │
        │ • Audit trail        │
        │ • Metrics            │
        │ • Error tracking     │
        └──────────────────────┘
```

---

## 🎯 Detailed Workflow Steps

### Phase 1: Error Analysis

**Step 1.1: Upload & Parse**
```
INPUT: pipeline_errors.txt (from CI/CD pipeline)
PROCESS:
  • Read file content
  • Tokenize error messages
  • Extract error codes & patterns
  • Identify service/module names
OUTPUT: Structured error data
```

**Step 1.2: Diagnosis Agent (ErrorDiagnosisAgent)**
```
AGENT LOGIC:
  1. Analyze error patterns
  2. Classify error type:
     - Version mismatch
     - Invalid SKU
     - Unsupported provider
     - Extension version conflict
     - Missing/invalid attributes
  3. Extract affected components
  4. Generate recommendations
OUTPUT: 
  error_type: str
  affected_services: list
  suggested_files: list
  priority: int (1-5)
```

---

### Phase 2: Repository Analysis

**Step 2.1: Repository Search Agent (RepositorySearchAgent)**
```
AGENT LOGIC:
  1. Connect to GitHub API
  2. Infer repo name from error context
     OR use hardcoded REPO_NAME
  3. Clone/fetch repository structure
  4. Identify file hierarchy
  5. Find deployment files:
     - main.tf
     - variables.tf
     - terraform.tfvars
     - other module files
  6. Extract current code content
OUTPUT:
  repo_path: str
  repo_structure: dict
  file_contents: dict
  commit_history: list
  current_branch: str
```

---

### Phase 3: Code Analysis

**Step 3.1: Code Analysis Agent (CodeAnalysisAgent)**
```
AGENT LOGIC:
  1. Compare error with code
  2. Identify affected lines:
     - Parse Terraform AST
     - Find error-causing definitions
     - Map error to code
  3. Determine fix scope:
     - Single file? (YES/NO)
     - Specific sections? (YES/NO)
  4. Extract code sections:
     - Context lines (5 before/after)
     - Variable definitions
     - Dependent resources
OUTPUT:
  files_to_fix: list
  error_line_numbers: list
  fix_scope: str (entire_file | specific_sections)
  code_sections: dict (section_name: code)
  dependencies: list
```

---

### Phase 4: AI-Powered Fix Generation

**Step 4.1: Fix Generation Agent (AIFixGenerationAgent)**
```
AGENT LOGIC:
  1. Build Gemini LLM prompt:
     {
       error_analysis: str,
       current_code: str,
       code_sections: str,
       error_context: str
     }
  2. Call Gemini API:
     "Fix these Terraform errors:"
     - Provide error details
     - Show current code
     - Ask for corrected code only
  3. Validate output:
     - Check Terraform syntax
     - Ensure no markdown wrapping
     - Verify all errors addressed
  4. Generate fix summary:
     - What was changed
     - Why it was changed
     - Validation results
OUTPUT:
  fixed_code: dict (filename: code)
  fix_summary: str
  changes_made: list
  validation_status: bool
```

**Gemini LLM Prompt Template:**
```
You are a Terraform expert AI fixing errors.

ERROR ANALYSIS:
{error_details}

CURRENT CODE:
{code_content}

ERROR LOCATION:
{line_numbers}

REQUIRED CHANGES:
{fix_requirements}

Your task:
1. Understand the error
2. Fix ONLY the necessary sections
3. Return corrected Terraform code
4. Do NOT wrap in markdown
5. Do NOT explain - code only
```

---

### Phase 5: Human Approval #1 (Branch Creation)

**Step 5.1: Present Changes to User**
```
┌─────────────────────────────────────────────┐
│ 🔍 FIX SUMMARY - READY FOR APPROVAL          │
├─────────────────────────────────────────────┤
│                                             │
│ ERROR TYPE: InvalidSKU                      │
│ PRIORITY: High                              │
│ AFFECTED FILES: main.tf                     │
│                                             │
│ CHANGES:                                    │
│ ├─ Line 12: app_service_plan.sku            │
│ │  OLD: "F1"  → NEW: "B1"                   │
│ │  Reason: F1 not available in region       │
│ │                                           │
│ ├─ Line 45: app_service_plan.sku_tier       │
│ │  OLD: undefined → NEW: "Free"             │
│ │  Reason: Required attribute               │
│ │                                           │
│ VALIDATION: ✅ All errors resolved          │
│                                             │
│ [👍 APPROVE & CREATE BRANCH]                │
│ [👎 REJECT & EXIT]                          │
│                                             │
└─────────────────────────────────────────────┘
```

---

### Phase 6: GitHub Operations #1 (Branch & Commit)

**Step 6.1: GitHub Branch Agent (GitHubBranchAgent)**
```
AGENT LOGIC:
  1. Authenticate with GitHub PAT
  2. Get repository object
  3. Get main branch SHA
  4. Create new branch:
     - Name: raju_<4-char-uuid>
     - Base: main branch SHA
  5. For each file to fix:
     a. Get current file SHA
     b. Update file content:
        - Path: file_path
        - Content: fixed_code
        - Message: "🤖 AI: Fix [error_type]"
        - Branch: new_branch
        - SHA: current_sha
  6. Return branch info
OUTPUT:
  branch_name: str
  branch_url: str
  commit_sha: str
  commit_url: str
  files_committed: list
  timestamp: datetime
```

---

### Phase 7: Human Approval #2 (Pull Request)

**Step 7.1: Present PR Plan to User**
```
┌─────────────────────────────────────────────┐
│ 🚀 PULL REQUEST - READY FOR APPROVAL        │
├─────────────────────────────────────────────┤
│                                             │
│ BRANCH: raju_a1b2                           │
│ COMMIT: abc123def456...                     │
│                                             │
│ FILES CHANGED:                              │
│ ├─ main.tf (+3, -2)                         │
│ │  ├─ Line 12: Updated app_service_plan    │
│ │  └─ Line 45: Added sku_tier attribute    │
│                                             │
│ PR TITLE:                                   │
│ "🤖 AI: Fix InvalidSKU Terraform Error"     │
│                                             │
│ PR DESCRIPTION:                             │
│ "Automated fix for Terraform errors:        │
│  - InvalidSKU: Changed F1 to B1             │
│  - Added required sku_tier attribute       │
│  - Validated all changes"                   │
│                                             │
│ LABELS: [ai-generated, auto-fix]            │
│ REVIEWERS: [@team-lead, @devops]            │
│                                             │
│ [✅ APPROVE & CREATE PR]                    │
│ [❌ REJECT - KEEP BRANCH]                   │
│                                             │
└─────────────────────────────────────────────┘
```

---

### Phase 8: GitHub Operations #2 (Create PR)

**Step 8.1: GitHub PR Agent (GitHubPRAgent)**
```
AGENT LOGIC:
  1. Authenticate with GitHub PAT
  2. Get repository object
  3. Create pull request:
     - Title: f"🤖 AI: Fix {error_type}"
     - Head: new_branch_name
     - Base: main_branch
     - Body: (detailed summary)
  4. Add labels:
     - "ai-generated"
     - "auto-fix"
     - error_type (e.g., "InvalidSKU")
  5. Assign reviewers (optional):
     - Team lead
     - DevOps team
  6. Return PR info
OUTPUT:
  pr_number: int
  pr_url: str
  pr_status: str ("open")
  creation_timestamp: datetime
  reviewers_assigned: list
```

---

## 🛠️ Technology Stack

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| **Orchestration** | LangGraph | Latest | State machine for multi-agent workflow |
| **Agents** | LangChain | Latest | Agent framework & tools |
| **LLM** | Gemini 2.5 Pro | Latest | Error diagnosis & code fixing |
| **VCS** | PyGithub | Latest | GitHub API integration |
| **Infrastructure** | Terraform | Latest | Code being fixed |
| **Language** | Python 3.10+ | Latest | Core implementation |
| **API Keys** | python-dotenv | Latest | Environment management |

---

## 📊 Data Models

### ErrorDiagnosisState
```python
@dataclass
class ErrorDiagnosisState:
    error_logs: str                    # Raw error text
    error_type: str                    # e.g., "InvalidSKU"
    affected_services: List[str]       # e.g., ["functionapp"]
    suggested_files: List[str]         # Files likely needing fixes
    priority: int                      # 1-5 priority level
    timestamp: datetime
```

### RepositoryAnalysisState
```python
@dataclass
class RepositoryAnalysisState:
    repo_name: str                     # e.g., "Raju7d/Ops-timus-Prime"
    repo_url: str
    file_structure: Dict[str, Any]     # Directory tree
    file_contents: Dict[str, str]      # filename: code
    current_branch: str                # Main branch name
    last_commit: str                   # SHA of latest commit
```

### CodeAnalysisState
```python
@dataclass
class CodeAnalysisState:
    files_to_fix: List[str]            # Which files need changes
    error_line_numbers: List[int]      # Specific lines with errors
    fix_scope: str                     # "entire_file" or "specific_sections"
    code_sections: Dict[str, str]      # section_name: code_content
    dependencies: List[str]            # Related files/modules
```

### AIFixState
```python
@dataclass
class AIFixState:
    fixed_code: Dict[str, str]         # filename: fixed_code
    fix_summary: str                   # What was changed
    changes_made: List[str]            # List of changes
    validation_status: bool            # Terraform syntax valid?
    gemini_response: str               # Full LLM response
```

### ApprovalGateState
```python
@dataclass
class ApprovalGateState:
    gate_number: int                   # 1 or 2
    prompt_text: str                   # What to ask user
    user_response: str                 # "yes" or "no"
    approved: bool
    timestamp: datetime
    reason_if_rejected: str            # Optional rejection reason
```

### GitHubOperationState
```python
@dataclass
class GitHubOperationState:
    operation_type: str                # "branch_create" or "pr_create"
    branch_name: str                   # e.g., "raju_a1b2"
    branch_url: str
    commit_sha: str
    commit_url: str
    files_committed: List[str]
    pr_number: int                     # For PR operations
    pr_url: str
    status: str                        # "success" or "failed"
    error_message: str                 # If failed
```

---

## 🔄 Agent Definitions (LangGraph)

### 1. ErrorDiagnosisAgent
```python
def error_diagnosis_agent(state: ErrorDiagnosisState) -> ErrorDiagnosisState:
    """
    Analyze error logs and classify the error type.
    Returns error classification and affected services.
    """
    # Implementation
    pass
```

### 2. RepositorySearchAgent
```python
def repository_search_agent(state: RepositoryAnalysisState) -> RepositoryAnalysisState:
    """
    Connect to GitHub, fetch repo structure and file contents.
    Returns file hierarchy and current code.
    """
    # Implementation
    pass
```

### 3. CodeAnalysisAgent
```python
def code_analysis_agent(state: CodeAnalysisState) -> CodeAnalysisState:
    """
    Analyze code and identify which files/sections need fixing.
    Returns fix scope and affected code sections.
    """
    # Implementation
    pass
```

### 4. AIFixGenerationAgent
```python
def ai_fix_generation_agent(state: AIFixState) -> AIFixState:
    """
    Send to Gemini LLM for code fixing.
    Returns fixed code and validation results.
    """
    # Implementation
    pass
```

### 5. HumanApprovalGate
```python
def human_approval_gate(state: ApprovalGateState) -> ApprovalGateState:
    """
    Interrupt workflow and ask for human approval.
    Returns user's decision.
    """
    # Implementation (with interrupt node)
    pass
```

### 6. GitHubBranchAgent
```python
def github_branch_agent(state: GitHubOperationState) -> GitHubOperationState:
    """
    Create branch and commit fixed files to GitHub.
    Returns branch info and commit SHA.
    """
    # Implementation
    pass
```

### 7. GitHubPRAgent
```python
def github_pr_agent(state: GitHubOperationState) -> GitHubOperationState:
    """
    Create pull request on GitHub.
    Returns PR URL and number.
    """
    # Implementation
    pass
```

---

## 📋 File Selection Logic

### Smart File Selection Algorithm

```python
def select_files_to_fix(error_type: str, repo_structure: Dict) -> List[str]:
    """
    Intelligently select which files need fixing based on error type.
    """
    
    FILE_MAPPINGS = {
        "InvalidSKU": [
            "*/main.tf",              # Main infrastructure
            "*/variables.tf",         # Variable definitions
            "*/terraform.tfvars"      # Variable values
        ],
        "VersionMismatch": [
            "versions.tf",            # Provider versions
            "*/terraform.tf"           # Terraform version
        ],
        "MissingAttribute": [
            "*/main.tf",              # Core resources
            "*/*.tf"                  # All TF files in module
        ],
        "InvalidProvider": [
            "versions.tf",            # Provider config
            "*/main.tf"
        ]
    }
    
    # Get suggested files for this error type
    patterns = FILE_MAPPINGS.get(error_type, ["*/main.tf"])
    
    # Find matching files in repo
    matching_files = []
    for pattern in patterns:
        matching_files.extend(find_files(repo_structure, pattern))
    
    # Deduplicate and sort by importance
    files = sorted(set(matching_files), 
                   key=lambda f: priority_score(f, error_type))
    
    return files
```

---

## 🎬 LangGraph Workflow Construction

```python
from langgraph.graph import StateGraph, END
from langgraph.checkpoint.memory import MemorySaver

# Create workflow graph
workflow = StateGraph(CompleteWorkflowState)

# Add nodes (agents)
workflow.add_node("error_diagnosis", error_diagnosis_agent)
workflow.add_node("repo_search", repository_search_agent)
workflow.add_node("code_analysis", code_analysis_agent)
workflow.add_node("ai_fix_generation", ai_fix_generation_agent)
workflow.add_node("human_approval_branch", human_approval_gate_1)
workflow.add_node("github_branch", github_branch_agent)
workflow.add_node("human_approval_pr", human_approval_gate_2)
workflow.add_node("github_pr", github_pr_agent)

# Add edges (transitions)
workflow.add_edge("error_diagnosis", "repo_search")
workflow.add_edge("repo_search", "code_analysis")
workflow.add_edge("code_analysis", "ai_fix_generation")
workflow.add_edge("ai_fix_generation", "human_approval_branch")

# Conditional edge based on approval
workflow.add_conditional_edges(
    "human_approval_branch",
    lambda state: "github_branch" if state.approved else END
)

workflow.add_edge("github_branch", "human_approval_pr")

workflow.add_conditional_edges(
    "human_approval_pr",
    lambda state: "github_pr" if state.approved else END
)

workflow.add_edge("github_pr", END)

# Set entry point
workflow.set_entry_point("error_diagnosis")

# Compile with memory
app = workflow.compile(checkpointer=MemorySaver())
```

---

## 🎯 Success Criteria

### Phase Completion Checkpoints

| Phase | Milestone | Status | Owner |
|-------|-----------|--------|-------|
| P1: Error Analysis | ErrorDiagnosisAgent working | ⏳ TODO | Backend |
| P1: Error Analysis | Error classification accurate | ⏳ TODO | Backend |
| P2: Repo Analysis | RepositorySearchAgent working | ⏳ TODO | Backend |
| P2: Repo Analysis | File fetching from GitHub | ⏳ TODO | Backend |
| P3: Code Analysis | CodeAnalysisAgent functional | ⏳ TODO | Backend |
| P3: Code Analysis | File/section selection logic | ⏳ TODO | Backend |
| P4: AI Fix Gen | AIFixGenerationAgent working | ✅ DONE | Backend |
| P4: AI Fix Gen | Gemini LLM integration | ✅ DONE | Backend |
| P5: Approval #1 | Human approval UI/mechanism | ⏳ TODO | Frontend |
| P5: Approval #1 | Summary presentation | ⏳ TODO | Frontend |
| P6: GitHub Ops #1 | GitHubBranchAgent working | ✅ PARTIAL | Backend |
| P6: GitHub Ops #1 | Branch creation & commit | ✅ PARTIAL | Backend |
| P7: Approval #2 | Human approval for PR | ⏳ TODO | Frontend |
| P7: Approval #2 | PR summary presentation | ⏳ TODO | Frontend |
| P8: GitHub Ops #2 | GitHubPRAgent working | ⏳ TODO | Backend |
| P8: GitHub Ops #2 | PR creation & labeling | ⏳ TODO | Backend |
| Integration | End-to-end workflow | ⏳ TODO | Full Team |
| Testing | Unit tests (70% coverage) | ⏳ TODO | QA |
| Testing | Integration tests | ⏳ TODO | QA |
| Deployment | Staging deployment | ⏳ TODO | DevOps |
| Deployment | Production deployment | ⏳ TODO | DevOps |

---

## 👥 Team Assignments

### Backend Team (2-3 Engineers)
- [ ] Implement ErrorDiagnosisAgent
- [ ] Implement RepositorySearchAgent
- [ ] Implement CodeAnalysisAgent
- [ ] Complete AIFixGenerationAgent
- [ ] Implement GitHubBranchAgent (file commit logic)
- [ ] Implement GitHubPRAgent
- [ ] LangGraph workflow integration
- [ ] Error handling & validation
- [ ] Unit tests for all agents

### Frontend Team (1-2 Engineers)
- [ ] File upload interface
- [ ] Display error analysis results
- [ ] Build approval gate UI (Human Approval #1)
- [ ] Build PR review UI (Human Approval #2)
- [ ] Dashboard/progress tracking
- [ ] Notification system

### DevOps Team (1 Engineer)
- [ ] Environment setup (.env configuration)
- [ ] API credentials management
- [ ] GitHub PAT setup and rotation
- [ ] Gemini API setup
- [ ] Staging/production deployment
- [ ] Monitoring & logging

### QA Team (1 Engineer)
- [ ] Test plan creation
- [ ] Unit test verification
- [ ] Integration testing
- [ ] End-to-end workflow testing
- [ ] Edge case testing
- [ ] Security testing

---

## 🚀 Implementation Timeline

```
Week 1: Foundation
├─ Day 1-2: LangGraph setup & state machine design
├─ Day 3-4: ErrorDiagnosisAgent implementation
└─ Day 5: RepositorySearchAgent implementation

Week 2: Core Agents
├─ Day 1-2: CodeAnalysisAgent implementation
├─ Day 3: AIFixGenerationAgent completion
├─ Day 4: GitHubBranchAgent (commit logic)
└─ Day 5: GitHubPRAgent implementation

Week 3: Integration & UI
├─ Day 1-2: LangGraph workflow integration
├─ Day 3: Frontend - File upload & results display
├─ Day 4: Frontend - Approval gate UI (both gates)
└─ Day 5: Testing & bug fixes

Week 4: Polish & Deploy
├─ Day 1-2: End-to-end testing
├─ Day 3: Security review
├─ Day 4: Documentation
└─ Day 5: Production deployment
```

---

## 📝 Documentation Needed

- [ ] Agent interface specifications
- [ ] LangGraph workflow diagram (detailed)
- [ ] API contracts (GitHub, Gemini)
- [ ] Error handling guide
- [ ] Deployment instructions
- [ ] User guide
- [ ] Developer guide
- [ ] Architecture decision records (ADRs)

---

## 🔒 Security Considerations

### Credential Management
```
✅ Use .env for local development
✅ Use GitHub Secrets for production
✅ Rotate credentials regularly
✅ Audit all GitHub API calls
✅ Log all AI operations
```

### Code Review
```
✅ All AI-generated code reviewed by human
✅ Approval gates prevent auto-merge
✅ Terraform validation before commit
✅ Syntax checking on generated code
```

### Audit Trail
```
✅ Log all workflow steps
✅ Track all user approvals/rejections
✅ Store all LLM prompts & responses
✅ Record all GitHub operations
✅ Timestamp all events
```

---

## 🐛 Known Limitations & Risks

| Issue | Impact | Mitigation |
|-------|--------|-----------|
| LLM hallucination | May generate incorrect fixes | Human approval + validation |
| File conflicts | Branch may fail if file changed | Rebase & retry logic |
| GitHub rate limits | API calls might throttle | Implement backoff strategy |
| Large code files | LLM context window limits | Split large files into sections |
| Concurrent workflows | Race conditions possible | Implement locking mechanism |
| Credential exposure | Security breach risk | Regular rotation & monitoring |

---

## 📞 Communication Plan

### Daily Standup
- 10 AM: 15-min sync on blockers
- Report: Completed tasks, current work, blockers

### Weekly Review
- Friday 3 PM: 30-min review
- Show working features, demo for stakeholders

### Escalation
- Critical issues: Immediate Slack notification
- Blockers: Escalate to tech lead within 1 hour
- Security issues: Report to security team immediately

---

## ✅ Definition of Done

Each task is complete when:
- [ ] Code written and reviewed
- [ ] Unit tests passing (>80% coverage)
- [ ] Integration tests passing
- [ ] Documentation updated
- [ ] No critical bugs remaining
- [ ] Peer reviewed by 2+ team members
- [ ] Deployed to staging successfully
- [ ] Demo'd to stakeholders

---

## 📌 Next Immediate Actions

1. **Backend Lead**: Review LangGraph architecture
2. **All Leads**: Assign team members to agents
3. **Tech Lead**: Set up development environment
4. **DevOps**: Prepare staging environment
5. **QA**: Create test plan
6. **Frontend**: Start file upload UI wireframes

---

## 📎 Appendix: LangGraph Basics

### What is LangGraph?
- Framework for building stateful, agentic applications
- Manages state transitions between agents
- Supports human-in-the-loop interrupts
- Provides persistence & checkpointing

### Key Concepts
- **StateGraph**: Defines workflow structure
- **Nodes**: Represent agents/operations
- **Edges**: Define transitions between nodes
- **Conditional Edges**: Route based on state
- **Interrupt Nodes**: Pause for human input
- **Checkpointer**: Save/restore workflow state

### Human Approval Pattern
```python
workflow.add_node("human_approval", human_approval_node)

# This creates an interrupt point
# Workflow pauses, waits for user input
# User provides approval/rejection
# Workflow resumes based on decision
```

---

**Document Version:** 1.0  
**Last Updated:** November 29, 2025  
**Next Review:** December 6, 2025
