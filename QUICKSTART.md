# Quick Start Guide

Get your app running on Azure in 15 minutes!

## Step 1: Create Your Repository (2 minutes)

1. Click **"Use this template"** button at the top of this repository
2. Name your new repository (e.g., `my-awesome-app`)
3. Choose Public or Private
4. Click **"Create repository"**

## Step 2: Get Azure Credentials (5 minutes)

### Option A: Quick Setup (Recommended)

1. Go to [Azure Portal](https://portal.azure.com)
2. Click on **Cloud Shell** (top right, icon looks like `>_`)
3. Run this command:

```bash
# Create a service principal for GitHub Actions
az ad sp create-for-rbac \
  --name "github-actions-$(date +%s)" \
  --role contributor \
  --scopes /subscriptions/$(az account show --query id -o tsv) \
  --json-auth
```

4. Copy the JSON output - you'll need it in the next step!

### Option B: Manual Setup

If you prefer, find these in Azure Portal:
- **Subscription ID**: Portal → Subscriptions
- **Tenant ID**: Portal → Azure Active Directory → Properties

## Step 3: Add GitHub Secrets (3 minutes)

1. In your new GitHub repo, go to **Settings** → **Secrets and variables** → **Actions**
2. Click **"New repository secret"** and add these 4 secrets:

From the JSON output in Step 2:

| Secret Name | Value from JSON |
|------------|-----------------|
| `AZURE_CLIENT_ID` | Copy `clientId` |
| `AZURE_CLIENT_SECRET` | Copy `clientSecret` |
| `AZURE_SUBSCRIPTION_ID` | Copy `subscriptionId` |
| `AZURE_TENANT_ID` | Copy `tenantId` |

## Step 4: Configure Your App (2 minutes)

Edit `app-config.yml` in your repository:

```yaml
app:
  name: "myapp"  # Change this to your app name (lowercase, 3-15 chars)
  region: "uksouth"  # Change if you want different region

components:
  backend:
    enabled: true  # Keep as-is for now
  frontend:
    enabled: true  # Keep as-is for now
  database:
    enabled: false  # Set to true if you need a database

environment: "dev"
```

Commit and push this change.

## Step 5: Deploy! (3 minutes to trigger, 10-15 min to deploy)

**Option A: Automatic (when you push to main)**
- The workflow triggers automatically
- Just push your changes!

**Option B: Manual Trigger**
1. Go to **Actions** tab
2. Click **"Deploy to Azure"**
3. Click **"Run workflow"**
4. Click green **"Run workflow"** button

## Step 6: Access Your App!

After deployment completes (10-15 minutes):

1. Go to **Actions** tab
2. Click on the latest workflow run
3. Scroll to bottom - you'll see:
   - 🌐 **Backend URL** - Your API endpoint
   - 🌐 **Frontend URL** - Your web app

Click the Frontend URL to see your app running!

## What You Get

- ✅ Backend API (FastAPI/Python)
- ✅ Frontend Web App (Node/Express)
- ✅ Azure Container Registry (for Docker images)
- ✅ PostgreSQL Database (if enabled)
- ✅ Auto-generated unique resource names
- ✅ CI/CD pipeline for updates

## Next Steps

### Replace with Your App

1. **Replace Backend:**
   - Add your code to `backend/` directory
   - Update `backend/Dockerfile` if needed
   - Push to main branch

2. **Replace Frontend:**
   - Add your React/Vue/Next.js app to `frontend/`
   - Update `frontend/Dockerfile`
   - Push to main branch

3. **Enable Database:**
   - Set `database.enabled: true` in `app-config.yml`
   - Database connection details are auto-configured

### Update Your App

Just push to main:
```bash
git add .
git commit -m "Update feature"
git push origin main
```

The pipeline automatically:
- Builds new Docker images
- Pushes to Azure Container Registry
- Restarts containers with new images
- Runs health checks

### Monitor Costs

Azure resources cost money! Here's what to expect:

- **With Free Credits**: First 30 days are FREE ($200 credit)
- **After Free Credits**: ~$60-80/month
- **To Save Money**: Destroy resources when not using them

### Destroy Infrastructure (Save Costs)

When you're done testing:

1. Go to **Actions** tab
2. Select **"Destroy Azure Infrastructure"**
3. Click **"Run workflow"**
4. Type `DESTROY` to confirm
5. Click **"Run workflow"**

This deletes everything and stops charges!

## Troubleshooting

### Deployment Failed?

1. **Check workflow logs** in Actions tab for error messages
2. **Verify secrets** are added correctly
3. **Check app name** is unique and lowercase
4. **Ensure Azure subscription** has available quota

### Can't Access App URLs?

1. **Wait 2-3 minutes** after deployment completes
2. **Check health status** in workflow summary
3. **View logs** in Azure Portal → Resource Groups → Your App

### Need Help?

- Check the detailed [README.md](README.md)
- Review workflow logs in Actions tab
- Check Azure Portal for resource status

## Tips

1. **Use Azure Free Credits**: Sign up at [azure.microsoft.com/free](https://azure.microsoft.com/free) for $200 credit
2. **Start Small**: Disable database if you don't need it
3. **Monitor Costs**: Set up cost alerts in Azure Portal
4. **Destroy When Done**: Always destroy test environments to save money
5. **Version Control**: Keep your `app-config.yml` in git

---

**You're all set! Happy deploying! 🚀**
