# Azure App Template - One-Click Deployment

Deploy your containerized application to Azure with zero Azure Portal interaction!

## 🚀 Quick Start

1. **Click "Use this template"** to create your own repository
2. **Add Azure credentials** to GitHub Secrets (one-time setup)
3. **Configure your app** in `app-config.yml`
4. **Push to main** or manually trigger the workflow
5. **Your app is live!** ✅

## What Gets Created

This template automatically provisions:

- ✅ Resource Group with unique naming
- ✅ Azure Container Registry (ACR)
- ✅ PostgreSQL Database (optional)
- ✅ Backend Container Instance
- ✅ Frontend Container Instance (optional)
- ✅ All networking and security configurations
- ✅ CI/CD pipeline for continuous deployment

## Prerequisites

- Azure Subscription (free tier works!)
- GitHub account
- 10 minutes of your time

## Setup Instructions

### Step 1: Add GitHub Secrets

Go to your repository Settings → Secrets → Actions and add:

| Secret Name | Description | How to Get |
|------------|-------------|------------|
| `AZURE_SUBSCRIPTION_ID` | Your Azure subscription ID | [Azure Portal](https://portal.azure.com) → Subscriptions |
| `AZURE_TENANT_ID` | Your Azure AD tenant ID | Azure Portal → Azure Active Directory → Properties |

**Note:** The workflow will create a service principal automatically on first run, or you can create one manually.

### Step 2: Configure Your App

Edit `app-config.yml`:

```yaml
# Application Configuration
app:
  name: my-app  # Letters and numbers only, will be made unique
  region: uksouth  # Azure region

# Components to deploy
components:
  backend:
    enabled: true
    port: 8000
  frontend:
    enabled: true
    port: 3000
  database:
    enabled: true
    type: postgresql

# Environment
environment: dev  # dev, staging, or prod
```

### Step 3: Add Your Application Code

Place your code in these directories:

```
backend/
  ├── Dockerfile
  ├── requirements.txt (Python) or package.json (Node)
  └── your-app-code/

frontend/  (optional)
  ├── Dockerfile
  ├── package.json
  └── your-app-code/
```

### Step 4: Deploy!

**Option A: Push to main**
```bash
git add .
git commit -m "Deploy my app"
git push origin main
```

**Option B: Manual trigger**
- Go to Actions tab
- Click "Deploy to Azure"
- Click "Run workflow"

### Step 5: Get Your URLs

After deployment (10-15 minutes), check the workflow summary for:
- 🌐 Backend URL
- 🌐 Frontend URL
- 📊 Database connection string (in secrets)

## What Happens During Deployment

```
┌─────────────────────────────────────────────┐
│  1. Validate Configuration                   │
│     ✓ Check app-config.yml                  │
│     ✓ Verify Azure credentials              │
└─────────────────────────────────────────────┘
                    ▼
┌─────────────────────────────────────────────┐
│  2. Provision Infrastructure (Terraform)     │
│     ✓ Create resource group                 │
│     ✓ Create container registry             │
│     ✓ Create database (if enabled)          │
│     ✓ Setup networking                      │
└─────────────────────────────────────────────┘
                    ▼
┌─────────────────────────────────────────────┐
│  3. Build & Deploy Application               │
│     ✓ Build Docker images                   │
│     ✓ Push to ACR                            │
│     ✓ Deploy containers                     │
│     ✓ Run health checks                     │
└─────────────────────────────────────────────┘
                    ▼
            🎉 App is Live!
```

## Project Structure

```
azure-app-template/
├── .github/
│   └── workflows/
│       ├── deploy.yml           # Main deployment workflow
│       └── destroy.yml          # Cleanup workflow
├── terraform/
│   ├── main.tf                  # Infrastructure definition
│   ├── variables.tf             # Configuration inputs
│   └── outputs.tf               # URLs and connection strings
├── backend/                     # Your backend code
│   ├── Dockerfile
│   └── ...
├── frontend/                    # Your frontend code (optional)
│   ├── Dockerfile
│   └── ...
├── app-config.yml              # Application configuration
└── README.md                    # This file
```

## Managing Your Deployment

### View Deployment Status
- Go to the **Actions** tab in GitHub
- Click on the latest workflow run
- View detailed logs and deployment summary

### Update Your App
Just push changes to main:
```bash
git add .
git commit -m "Update feature"
git push
```

The pipeline automatically:
- Tests your code
- Builds new images
- Deploys updates
- Runs health checks

### Destroy Infrastructure
When you're done (to save costs):

1. Go to Actions tab
2. Select "Destroy Azure Infrastructure"
3. Click "Run workflow"
4. Confirm destruction

**Warning:** This deletes everything including the database!

## Cost Estimates

Based on Azure pricing (as of 2025):

| Resource | Cost (per month) |
|----------|------------------|
| Container Instances (2x) | ~$30-50 |
| Container Registry | ~$5 |
| PostgreSQL Basic | ~$25 |
| **Total** | **~$60-80/month** |

**Free tier:** Use Azure free credits ($200 for 30 days) for testing!

## Customization

### Different Regions
Edit `app-config.yml`:
```yaml
app:
  region: eastus  # or westeurope, southeastasia, etc.
```

### Resource Sizing
Modify `terraform/variables.tf` to change CPU/memory allocations.

### Different Database
```yaml
components:
  database:
    type: mysql  # or postgresql, none
```

## Troubleshooting

### Deployment Failed?
1. Check Actions logs for error messages
2. Verify Azure secrets are correct
3. Check Azure subscription has available quota
4. Ensure resource names are unique

### App Not Responding?
1. Check container logs in Azure Portal
2. Verify Dockerfile builds locally
3. Check health check endpoints

### Need Help?
- Check the Actions workflow logs
- Review Terraform plan output
- Azure Portal → Resource Groups → [your-app-name]

## Advanced Features

### Environment Variables
Add to `app-config.yml`:
```yaml
environment_variables:
  API_KEY: "secret-key"
  DEBUG: "true"
```

### Custom Domain
After deployment, configure in Azure:
1. Azure Portal → Container Instance
2. Add custom domain
3. Configure DNS

## Security

- ✅ Service principal has minimal permissions
- ✅ Secrets stored in GitHub (encrypted)
- ✅ Database passwords auto-generated
- ✅ Network security groups configured
- ✅ HTTPS support ready

## Next Steps

1. ⭐ Star this template if it helps!
2. 🔧 Customize for your needs
3. 🚀 Deploy your app
4. 📊 Monitor in Azure Portal
5. 💰 Set up cost alerts

## License

MIT - Feel free to use and modify!

---

**Built with ❤️ using Terraform and GitHub Actions**
