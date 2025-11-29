# 🤖 Agentic Terraform Error Fixer - normal_flow.ipynb

## Overview

This notebook implements a complete **Agentic AI workflow** that automatically diagnoses Terraform errors, fixes them using Google Gemini AI, and commits the fixes to a GitHub repository with human approval.

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        INPUT SOURCES                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  📋 Error Logs                    📄 Broken Terraform Code       │
│  (Terraform validation errors)    (main.tf with issues)          │
│                                                                   │
└──────────────────────┬──────────────────────────────────────────┘
                       │
                       ▼
        ┌──────────────────────────────────┐
        │  Step 1: LOAD & PREPARE         │
        ├──────────────────────────────────┤
        │ • Load env variables            │
        │ • Initialize Gemini LLM         │
        │ • Read error logs               │
        │ • Read broken main.tf           │
        └──────────────────┬───────────────┘
                           │
                           ▼
        ┌──────────────────────────────────┐
        │  Step 2: AI DIAGNOSIS 🧠         │
        ├──────────────────────────────────┤
        │ • Send errors + code to Gemini  │
        │ • LLM analyzes issues           │
        │ • LLM generates fixes           │
        │ • Extract clean Terraform code  │
        └──────────────────┬───────────────┘
                           │
                           ▼
        ┌──────────────────────────────────┐
        │  Step 3: SAVE LOCALLY           │
        ├──────────────────────────────────┤
        │ • Save fixed code as            │
        │   fixed_main.tf file            │
        └──────────────────┬───────────────┘
                           │
                           ▼
        ┌──────────────────────────────────┐
        │  Step 4: HUMAN APPROVAL ✋       │
        ├──────────────────────────────────┤
        │ • Ask user: approve changes?    │
        │ • YES → Continue to GitHub      │
        │ • NO  → Exit safely             │
        └──────────────────┬───────────────┘
                           │
                ┌──────────┴──────────┐
                │ YES                 │ NO
                ▼                     ▼
   ┌──────────────────────┐    ✅ Exit
   │ Step 5: GITHUB OPS   │
   ├──────────────────────┤
   │ • Connect to GitHub  │
   │ • Create new branch  │
   │ • Commit fixed code  │
   │ • File path:         │
   │   Function APP/      │
   │   infra/main.tf      │
   └──────────┬───────────┘
              │
              ▼
   ┌──────────────────────┐
   │ ✅ COMPLETE          │
   ├──────────────────────┤
   │ • Branch created     │
   │ • Code committed     │
   │ • Ready for PR       │
   └──────────────────────┘
```

---

## 📋 Notebook Flow (Cell by Cell)

### Cell 1: Import Libraries
```python
from dotenv import load_dotenv
import os
from langchain_google_genai import ChatGoogleGenerativeAI
from langchain_core.messages import HumanMessage, SystemMessage, AIMessage
from github import Github
```
**What it does:** Imports required libraries for LLM, GitHub API, and environment management.

---

### Cell 2: Load Environment Variables
```python
load_dotenv()
GOOGLE_API_KEY = os.getenv("GOOGLE_API_KEY")
GITHUB_TOKEN = os.getenv("GITHUB_TOKEN")
REPO_NAME = os.getenv("REPO_NAME")
MAIN_BRANCH = os.getenv("MAIN_BRANCH", "main")
```
**What it does:** Loads credentials from `.env` file for Gemini AI and GitHub.

---

### Cell 3: Configure the LLM
```python
llm = ChatGoogleGenerativeAI(
    model="gemini-2.5-pro",
    temperature=0.0,
    max_retries=2,
)
```
**What it does:** Initializes Google Gemini LLM with deterministic settings (temperature=0.0 for consistent results).

---

### Cell 4: Create Diagnosis Agent
```python
def diagnosis_agent(error_text: str, main_tf: str) -> str:
    # Send to Gemini: fix these Terraform errors
    # Get back: corrected Terraform code
```
**What it does:**
- Takes error logs and broken code as input
- Sends to Gemini AI with specific prompt
- Cleans up markdown formatting from response
- Returns fixed Terraform code

**Key Features:**
- ✅ Handles markdown code blocks
- ✅ Extracts raw Terraform code
- ✅ Error handling for LLM failures

---

### Cell 5: Load Error Files
```python
error_file_path = r"C:\Users\virub\OneDrive\Desktop\Hackathon sat test\Errors\1. Terraform errors.txt"
main_tf_path = r"C:\Users\virub\OneDrive\Desktop\Hackathon sat test\main.tf"

with open(error_file_path, "r") as f:
    error_text = f.read()
with open(main_tf_path, "r") as f:
    main_tf = f.read()
```
**What it does:** Reads local error logs and broken Terraform files.

---

### Cell 6: Run Diagnosis Agent
```python
fixed_terraform_code = diagnosis_agent(error_text, main_tf)
```
**What it does:** Calls diagnosis agent to fix the Terraform code using Gemini AI.

---

### Cell 7: Save Fixed Code Locally
```python
with open("fixed_main.tf", "w") as tf_file:
    tf_file.write(fixed_terraform_code)
```
**What it does:** Saves the AI-generated fixed code to a local file for backup.

---

### Cell 8: Create GitHub Agent (Main Logic)
```python
def create_github_agent():
    # Step 1: Ask for human approval
    approval = input("Do you approve? (yes/no): ")
    if approval not in ["yes", "y"]:
        return
    
    # Step 2: Connect to GitHub
    gh = Github(GITHUB_TOKEN)
    repo = gh.get_repo(REPO_NAME)
    
    # Step 3: Create new branch from main
    base_ref = repo.get_git_ref(f"heads/{MAIN_BRANCH}")
    new_branch_name = f"raju_{uuid.uuid4().hex[:4]}"
    repo.create_git_ref(
        ref=f"refs/heads/{new_branch_name}", 
        sha=base_ref.object.sha
    )
    
    # Step 4: Commit fixed code to the branch
    file_path = "Function APP/infra/main.tf"
    contents = repo.get_contents(file_path, ref=new_branch_name)
    repo.update_file(
        path=file_path,
        message="🤖 AI: Fix Terraform errors (automated)",
        content=fixed_terraform_code,
        sha=contents.sha,
        branch=new_branch_name,
    )
```

**What it does:**
1. ✅ Asks for human approval (safety mechanism)
2. ✅ Connects to GitHub with credentials
3. ✅ Gets the main branch reference
4. ✅ Creates a new branch: `raju_XXXX`
5. ✅ Updates the file at `Function APP/infra/main.tf`
6. ✅ Commits with message: "🤖 AI: Fix Terraform errors (automated)"

---

### Cell 9: Execute GitHub Agent
```python
create_github_agent()
```
**What it does:** Runs the complete workflow.

---

## 🔄 Complete End-to-End Flow

```
1. USER LOADS NOTEBOOK
   ↓
2. LOAD CREDENTIALS (from .env)
   ↓
3. INITIALIZE GEMINI LLM
   ↓
4. READ ERROR LOGS & BROKEN CODE (local files)
   ↓
5. SEND TO GEMINI: "Fix these Terraform errors"
   ↓
6. GEMINI RESPONDS: "Here's the fixed code"
   ↓
7. SAVE FIXED CODE LOCALLY (backup)
   ↓
8. ASK USER: "Approve changes?" ✋
   ├─ YES → Continue
   └─ NO → Exit
   ↓
9. CONNECT TO GITHUB (REPO_NAME)
   ↓
10. GET MAIN BRANCH REFERENCE
   ↓
11. CREATE NEW BRANCH: raju_XXXX
   ↓
12. UPDATE FILE: Function APP/infra/main.tf
   ↓
13. COMMIT CHANGES (with message)
   ↓
14. COMPLETE ✅
   ↓
15. NEXT STEPS: Create Pull Request (manual or automated)
```

---

## 📊 Data Flow Diagram

```
┌─────────────────┐
│ Error Logs      │
│ (local file)    │
└────────┬────────┘
         │
         ├─────────────────────────────┐
         │                             │
         ▼                             ▼
    ┌─────────────────┐          ┌──────────────┐
    │ diagnosis_agent │◄────────►│ Gemini LLM   │
    └────────┬────────┘          └──────────────┘
             │
             ├─────────────────────────────┐
             │                             │
             ▼                             ▼
    ┌─────────────────┐          ┌──────────────┐
    │ Broken Code     │          │ Fixed Code   │
    │ (main.tf)       │          │ (from LLM)   │
    └────────┬────────┘          └──────┬───────┘
             │                          │
             └──────────┬───────────────┘
                        │
                        ▼
                ┌─────────────────────┐
                │ Human Approval      │
                │ (User decides)      │
                └────────┬────────────┘
                         │
                    ┌────┴────┐
                    │          │
                  YES          NO
                    │          │
                    ▼          ▼
            ┌───────────────┐ ✅ Exit
            │ GitHub        │
            │ • Connect     │
            │ • Create      │
            │   Branch      │
            │ • Commit Code │
            │ • Update File │
            └───────────────┘
                    │
                    ▼
            ┌───────────────┐
            │ ✅ COMPLETE   │
            │ Ready for PR  │
            └───────────────┘
```

---

## 🔑 Key Features

### 1. **AI-Powered Diagnosis**
- Uses Google Gemini 2.5 Pro (latest model)
- Deterministic output (temperature=0.0)
- Automatic markdown cleanup
- Handles complex Terraform errors

### 2. **Human-in-the-Loop**
- User must approve before GitHub changes
- Safety mechanism to prevent accidental deployments
- Clear confirmation required

### 3. **Automated GitHub Operations**
- Connects with credentials from `.env`
- Creates unique branch names: `raju_XXXX`
- Updates file at: `Function APP/infra/main.tf`
- Automatic commit with descriptive message

### 4. **Error Handling**
- Try-except blocks throughout
- Clear error messages
- Graceful failure modes

### 5. **Local Backup**
- Saves fixed code as `fixed_main.tf` locally
- Can review before committing to GitHub

---

## 🚀 How to Use

### Prerequisites
```bash
pip install python-dotenv langchain-google-genai PyGithub
```

### Setup .env File
```
GOOGLE_API_KEY=your-gemini-api-key
GITHUB_TOKEN=your-github-pat
REPO_NAME=owner/repo-name
MAIN_BRANCH=main
```

### Run Notebook
1. Open `normal_flow.ipynb`
2. Run cells in order (1-9)
3. At Cell 9, you'll be asked to approve
4. Type `yes` to create branch and commit
5. Check GitHub for new branch

---

## 📈 Expected Output

```
Cell 6: LLM Diagnosis
✅ LLM Response received
✅ Fixed code extracted (3956 chars)

Cell 8: GitHub Operations
Connected as: Raju7d
Repository accessed: Raju7d/Ops-timus-Prime
Base ref: refs/heads/main
✅ Branch created: raju_a1b2
✅ Updated: Function APP/infra/main.tf
✅ Committed to branch: raju_a1b2
```

---

## 🔗 Next Steps After This Workflow

### Option 1: Manual PR Creation
1. Go to GitHub repository
2. Create Pull Request from `raju_XXXX` to `main`
3. Review the changes
4. Merge when ready

### Option 2: Automate PR Creation
Extend the notebook to automatically create PR:
```python
pr = repo.create_pull(
    title="🤖 AI: Automated Terraform Error Fix",
    body="Fixed Terraform errors automatically",
    head=new_branch_name,
    base=MAIN_BRANCH
)
print(f"PR created: {pr.html_url}")
```

---

## 🛠️ Troubleshooting

| Issue | Solution |
|-------|----------|
| "API key not set" | Check .env file exists and has GOOGLE_API_KEY |
| "Repository not found" | Verify REPO_NAME format: `owner/repo` |
| "File not found" | Ensure path `Function APP/infra/main.tf` exists in repo |
| LLM errors | Check GOOGLE_API_KEY is valid and active |
| GitHub auth fails | Regenerate GITHUB_TOKEN and verify scopes |

---

## 📝 System Components

| Component | Purpose | Status |
|-----------|---------|--------|
| Gemini LLM | Diagnose and fix errors | ✅ Working |
| Diagnosis Agent | Extract fixed code | ✅ Working |
| GitHub API | Branch creation & commits | ✅ Working |
| Human Approval | Safety gate | ✅ Working |
| File Operations | Local save/load | ✅ Working |

---

## 🎯 Summary

This notebook provides a **complete agentic workflow** for automatically fixing Terraform infrastructure errors:

- 🧠 **AI Diagnosis**: Gemini analyzes and fixes errors
- ✅ **Quality Gate**: Human approval before changes
- 🚀 **Automation**: Commits to GitHub automatically
- 📊 **Transparency**: Clear progress messages
- 🔒 **Safety**: Local backup and approval mechanisms

**Result:** Terraform errors fixed automatically with human oversight! 🎉

---

**Ready to use? Run the notebook!** ▶️
