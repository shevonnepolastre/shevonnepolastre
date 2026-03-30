# Minecraft Dashboard App

A full-stack Node.js web application deployed on Azure App Service that connects a
web form to a Notion database via the Notion API. Built to explore AZ-104 concepts
including App Services, infrastructure as code, and deployment automation.

---

## What It Does

Users submit data through a web form. The Express.js backend receives the submission
and writes it to a Notion database via the Notion API. The entire stack is deployed
to Azure App Service (Linux) using GitHub Actions and provisioned with Bicep.

---

## Stack

| Layer | Technology |
|-------|------------|
| Backend | Node.js + Express.js |
| Data | Notion API |
| Infrastructure | Azure App Service (Linux, 64-bit) |
| IaC | Azure Bicep |
| CI/CD | GitHub Actions + self-hosted runner |
| Auth | Service principal (JSON secret) |

---

## Setup

### 1. Clone and install dependencies

```bash
git clone https://github.com/shevonnepolastre/minecraft-dashboard-app.git
cd minecraft-dashboard-app
npm install
```

### 2. Configure environment variables

Copy `.env.example` to `.env` and fill in your values:

```bash
cp .env.example .env
```

Required variables:

```
NOTION_KEY=your-notion-api-key
NOTION_DATABASE_ID=your-notion-database-id
```

For Azure authentication, use a service principal JSON secret rather than
four separate secrets:

```
AZURE_CREDENTIALS={"clientId":"...","clientSecret":"...","tenantId":"...","subscriptionId":"..."}
```

**Do not commit `.env` to version control.** It is already in `.gitignore`.
Add all variables to Azure App Service Environment Variables before deploying.

In `server.js`, reference them as:

```javascript
const notion = new Client({ auth: process.env.NOTION_KEY });
const databaseId = process.env.NOTION_DATABASE_ID;
```

### 3. Create the Notion integration

Go to [Notion Developers](https://developers.notion.com/docs/getting-started) and
create an integration. Use **Internal** for personal use or **Public** if others
will access the form. Copy the API key into your `.env`.

### 4. Provision Azure App Service

Bicep template is in `infrastructure/appservice.bicep`. Before deploying, confirm:

- App Service tier is **Basic or higher** (Free tier does not support 64-bit)
- OS is set to **Linux**
- Node.js runtime is specified

Deploy via CLI:

```bash
az deployment group create \
  --resource-group <your-rg> \
  --template-file infrastructure/appservice.bicep
```

### 5. Configure the GitHub Actions pipeline

The workflow file is at `.github/workflows/appservice.yml`.

Add required secrets to the repository:

| Secret | Value |
|--------|-------|
| `AZURE_CREDENTIALS` | Service principal JSON |
| `NOTION_KEY` | Notion API key |
| `NOTION_DATABASE_ID` | Notion database ID |

### 6. Add a self-hosted runner

Follow the [GitHub guide for self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/adding-self-hosted-runners).

### 7. Test locally before deploying

```bash
node server.js
```

Open `localhost:[port]` in your browser and verify the form submits to Notion
before pushing to Azure.

### 8. Deploy

Push to `main` to trigger the GitHub Actions workflow. The pipeline builds and
deploys the app to Azure App Service automatically.

### 9. Troubleshoot

If the deployed app has issues, use two tools in the Azure portal:

- **Log Stream** — live output from the running app
- **Debug Console** — shell access to the App Service container

---

## Lessons Learned

- Linux App Service is significantly easier to configure than Windows for Node.js
- Free tier does not support 64-bit — use Basic or higher
- Deployment Slots require Standard tier or above — Basic does not include them
- Always add `.env` variables to Azure Environment Variables before deploying —
  the app will fail silently if they're missing
- Starting from an existing sample (Notion's Express example) and adapting it is
  a legitimate engineering approach

---

## Reference Resources

- [Notion API Getting Started](https://developers.notion.com/docs/getting-started)
- [Notion SDK Express web form example](https://github.com/makenotion/notion-sdk-js/tree/main/examples/web-form-with-express)
- [Deploy Node.js to Azure App Service via GitHub Actions](https://docs.github.com/en/actions/use-cases-and-examples/deploying/deploying-nodejs-to-azure-app-service)
- [Provision App Service with Bicep (Linux)](https://learn.microsoft.com/en-us/azure/app-service/provision-resource-bicep?pivots=app-service-bicep-linux)
- [Configure Azure App Service for 64-bit Node.js](https://devblogs.microsoft.com/premier-developer/configure-azure-app-service-for-64-bit-platform-and-node-js/)

---

## Related Projects

- [azure-infrastructure-labs](https://github.com/shevonnepolastre/minecraft-azure-quests) — AZ-104 lab environment covering identity, storage, compute, networking, and monitoring
- [minecraft-azure-server](https://github.com/shevonnepolastre/minecraft-azure-server) — automated Minecraft server deployment on Azure using GitHub Actions and Bicep
- [intune-assistant-chatbot](https://github.com/shevonnepolastre/intune-assistant-chatbot) — RAG-based chatbot using Azure AI Foundry and Azure Cognitive Search
