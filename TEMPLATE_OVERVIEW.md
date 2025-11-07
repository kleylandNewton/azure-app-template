# Azure App Template - Architecture Overview

## 🎯 What This Template Does

This is a **proof-of-concept template repository** that allows users to deploy containerized applications to Azure with **zero Azure Portal interaction**. Everything is controlled centrally through configuration files and automated via GitHub Actions.

## 🏗️ Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                    USER CLICKS "USE TEMPLATE"                │
└─────────────────────┬────────────────────────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────────────────────────┐
│           USER'S NEW REPOSITORY (from template)              │
│                                                              │
│  Contains:                                                   │
│  • app-config.yml (user edits this ONLY)                    │
│  • terraform/ (centralized infrastructure)                  │
│  • .github/workflows/ (automated deployment)                │
│  • backend/ & frontend/ (sample Hello World apps)           │
└─────────────────────┬────────────────────────────────────────┘
                      │
                      │ User edits app-config.yml
                      │ and pushes to main
                      ▼
┌──────────────────────────────────────────────────────────────┐
│              GITHUB ACTIONS WORKFLOW                         │
│                                                              │
│  1. Parse app-config.yml                                    │
│  2. Generate Terraform variables automatically              │
│  3. Create Azure resources with unique names                │
│  4. Build Docker images                                     │
│  5. Push to Azure Container Registry                        │
│  6. Deploy containers                                       │
│  7. Run health checks                                       │
└─────────────────────┬────────────────────────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────────────────────────┐
│                    AZURE INFRASTRUCTURE                      │
│                                                              │
│  Auto-created with standardized naming:                     │
│  • rg-{appname}-{env}-{random}                             │
│  • acr{appname}{env}{random}                               │
│  • aci-{appname}-backend-{env}-{random}                    │
│  • aci-{appname}-frontend-{env}-{random}                   │
│  • psql-{appname}-{env}-{random} (if enabled)              │
└──────────────────────────────────────────────────────────────┘
```

## 📁 Template Structure

```
azure-app-template/
├── .github/
│   └── workflows/
│       ├── deploy.yml           # Main deployment workflow
│       └── destroy.yml          # Cleanup workflow
│
├── terraform/
│   ├── main.tf                  # Infrastructure definition
│   ├── variables.tf             # Input variables
│   └── outputs.tf               # URLs and connection strings
│
├── backend/                     # Sample backend app
│   ├── Dockerfile
│   ├── main.py                  # FastAPI Hello World
│   └── requirements.txt
│
├── frontend/                    # Sample frontend app
│   ├── Dockerfile
│   ├── server.js                # Express Hello World
│   └── package.json
│
├── app-config.yml              # ⭐ USER EDITS THIS ONLY
├── README.md                    # Full documentation
├── QUICKSTART.md               # 15-minute setup guide
└── .gitignore

```

## 🎛️ Centralized Control

### What Users Control (app-config.yml)

Users ONLY edit this simple configuration file:

```yaml
app:
  name: "myapp"          # App name (3-15 chars, lowercase)
  region: "uksouth"      # Azure region

components:
  backend:
    enabled: true        # Deploy backend?
    port: 8000
    cpu: 1.0
    memory: 1.5

  frontend:
    enabled: true        # Deploy frontend?
    port: 3000
    cpu: 0.5
    memory: 1.0

  database:
    enabled: true        # Deploy database?
    type: "postgresql"

environment: "dev"       # dev, staging, or prod
```

### What's Controlled Centrally (terraform/)

Everything else is **controlled by the template** in the Terraform files:

**Naming Conventions:**
- Resource Group: `rg-{appname}-{env}-{random6}`
- Container Registry: `acr{appname}{env}{random6}`
- Backend Container: `aci-{appname}-backend-{env}-{random6}`
- Frontend Container: `aci-{appname}-frontend-{env}-{random6}`
- Database: `psql-{appname}-{env}-{random6}`

**Infrastructure Patterns:**
- All Azure resources follow best practices
- Security groups configured automatically
- Networking setup handled centrally
- Database passwords auto-generated and secured
- Tags applied consistently

**Deployment Strategy:**
- Container Registry creation
- Docker image management
- Container orchestration
- Health checking
- Rollout strategy

## 🔄 Deployment Workflow

### 1. Configuration Parsing

```yaml
# GitHub Actions reads app-config.yml
parse-config:
  - Extract app.name
  - Extract app.region
  - Extract component flags
  - Validate configuration
  - Pass to next jobs
```

### 2. Infrastructure Provisioning

```yaml
# Terraform creates Azure resources
terraform-apply:
  - Generate unique suffix (6 random chars)
  - Apply naming convention to all resources
  - Create resource group
  - Create container registry
  - Create database (if enabled)
  - Create container instances
  - Output URLs and credentials
```

### 3. Application Deployment

```yaml
# Build and deploy containers
push-to-acr:
  - Build backend Docker image
  - Build frontend Docker image
  - Push to ACR with standardized tags
  - Restart containers to pull new images
  - Run health checks
```

### 4. Output Generation

```yaml
# Provide user with access information
outputs:
  - Backend URL: http://{backend-fqdn}:8000
  - Frontend URL: http://{frontend-fqdn}:3000
  - Resource Group name
  - Database connection string (if enabled)
```

## 🔐 Security Model

### Secrets Management

**What users provide:**
- `AZURE_SUBSCRIPTION_ID` - Their Azure subscription
- `AZURE_TENANT_ID` - Their Azure AD tenant
- `AZURE_CLIENT_ID` - Service principal ID
- `AZURE_CLIENT_SECRET` - Service principal secret

**What's auto-generated:**
- Database passwords (random 32 chars)
- Container registry credentials
- Resource naming (includes random suffix)

### Least Privilege Access

Service principal only needs:
- Contributor role on subscription (or specific resource group)
- AcrPush role on container registry

### No Hard-Coded Credentials

- All secrets in GitHub Secrets (encrypted)
- Database passwords generated at deploy time
- ACR credentials from Terraform outputs
- Never committed to repository

## 🎨 Customization Points

### For Template Maintainers (You)

**Modify centralized patterns in:**
- `terraform/main.tf` - Infrastructure definition
- `terraform/variables.tf` - Available options
- `.github/workflows/deploy.yml` - Deployment logic

**Change naming conventions:**
```hcl
# In terraform/main.tf locals block
resource_group_name = "rg-${var.app_name}-${local.unique_suffix}"
# Change pattern here to affect ALL deployments
```

**Adjust resource defaults:**
```hcl
# In terraform/variables.tf
variable "backend_cpu" {
  default = 1.0  # Change default CPU allocation
}
```

### For Template Users

**Only edit:**
- `app-config.yml` - Simple configuration
- `backend/` - Replace with their code
- `frontend/` - Replace with their code

## 🧪 Testing the Template

### Test with Current Infrastructure

You can test this with your existing `azure-simulation-app`:

1. **Create new GitHub repo** from this template
2. **Add your Azure secrets** (same ones you have)
3. **Edit app-config.yml**:
   ```yaml
   app:
     name: "simtest"
     region: "uksouth"
   ```
4. **Push to main**
5. **Watch it create everything automatically**

### Test with Hello World

The template includes simple Hello World apps:
- Backend: FastAPI with 3 endpoints
- Frontend: Express serving interactive HTML

This lets users test deployment before adding their code.

### Test Destroy Workflow

```yaml
# .github/workflows/destroy.yml
# Safely removes all resources
# Requires typing "DESTROY" to confirm
```

## 💡 Key Design Decisions

### 1. Why Centralized Naming?

**Pros:**
- ✅ Consistent across all deployments
- ✅ No naming conflicts
- ✅ Easy to identify resources
- ✅ Automated uniqueness via random suffix

**Cons:**
- ❌ Users can't choose exact resource names
- ❌ Names might be long

**Decision:** Consistency and automation > user control

### 2. Why app-config.yml instead of Terraform vars?

**Pros:**
- ✅ Simple YAML format (non-technical friendly)
- ✅ Single source of truth
- ✅ Easy to understand
- ✅ No Terraform knowledge required

**Cons:**
- ❌ Workflow must parse and convert to Terraform

**Decision:** User-friendliness > technical efficiency

### 3. Why Include Sample Apps?

**Pros:**
- ✅ Users can test deployment immediately
- ✅ Provides working Docker examples
- ✅ Shows how to structure code
- ✅ Validates infrastructure works

**Cons:**
- ❌ Extra files in template

**Decision:** Better onboarding > minimal template

### 4. Why Separate Deploy/Destroy Workflows?

**Pros:**
- ✅ Prevents accidental deletion
- ✅ Requires explicit confirmation
- ✅ Clear cost management

**Cons:**
- ❌ Two workflows to maintain

**Decision:** Safety > convenience

## 🚀 How Users Deploy Their Real Apps

### Step 1: Use Template
Click "Use this template" → New repo created

### Step 2: Configure
Edit `app-config.yml`:
```yaml
app:
  name: "coolapp"
  region: "uksouth"
```

### Step 3: Add Their Code

**Backend:**
```bash
# Remove sample code
rm -rf backend/*

# Add their code
cp -r ~/my-python-api/* backend/

# Ensure Dockerfile exists
# backend/Dockerfile should build their app
```

**Frontend:**
```bash
# Remove sample code
rm -rf frontend/*

# Add their Next.js app
cp -r ~/my-nextjs-app/* frontend/

# Ensure Dockerfile exists
# frontend/Dockerfile should build their app
```

### Step 4: Push to Deploy
```bash
git add .
git commit -m "Deploy my app"
git push origin main
```

### Step 5: Access App
Workflow outputs:
- Backend URL
- Frontend URL
- Resource Group name

## 📊 Cost Management

### Estimated Costs

Based on default configuration:
- Container Instances (2x): ~$30-50/month
- Container Registry: ~$5/month
- PostgreSQL (if enabled): ~$25/month
- **Total: ~$60-80/month**

### Cost Saving Tips

1. **Use Azure Free Credits**: $200 for 30 days
2. **Destroy when not using**: Run destroy workflow
3. **Disable database**: Set `database.enabled: false`
4. **Lower resources**: Reduce CPU/memory in config

### Auto-Tagging for Cost Tracking

All resources tagged with:
```hcl
tags = {
  Environment = var.environment
  ManagedBy   = "Terraform"
  AppName     = var.app_name
  DeployedAt  = timestamp()
}
```

## 🔮 Future Enhancements

### Potential Improvements

1. **Multi-Environment Support**
   - Separate dev/staging/prod branches
   - Environment-specific configurations
   - Promotion workflows

2. **Database Options**
   - MySQL support
   - Cosmos DB option
   - Database migration scripts

3. **Advanced Features**
   - Custom domains
   - SSL certificates
   - Azure Front Door
   - Application Insights
   - Log Analytics

4. **Terraform State Management**
   - Remote state in Azure Storage
   - State locking
   - Workspace management

5. **Testing Integration**
   - Unit tests before deploy
   - Integration tests after deploy
   - Smoke tests

6. **Notifications**
   - Slack/Teams webhooks
   - Email notifications
   - Deployment summaries

## 📝 Next Steps

### To Use This Template

1. **Push to GitHub**:
   ```bash
   cd /Users/kitleyland/Documents/Coding_projects/azure-app-template
   gh repo create azure-app-template --public --source=. --push
   ```

2. **Make it a Template**:
   - Go to repo Settings
   - Check "Template repository"

3. **Test It**:
   - Click "Use this template"
   - Add Azure secrets
   - Push to main
   - Watch it deploy!

### To Improve This Template

1. Review and test Terraform configurations
2. Add more database options
3. Enhance error handling
4. Add monitoring/logging
5. Create video walkthrough
6. Write blog post

---

**This template demonstrates:**
- ✅ Infrastructure as Code
- ✅ GitOps workflow
- ✅ Centralized control
- ✅ User-friendly configuration
- ✅ Automated deployment
- ✅ Cost management
- ✅ Security best practices

**Perfect for:**
- Development teams wanting standardized deployments
- Platform teams providing self-service infrastructure
- Hackathons and quick prototypes
- Learning Azure + Terraform + GitHub Actions
