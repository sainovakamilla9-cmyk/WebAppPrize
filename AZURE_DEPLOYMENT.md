# Nasazení na Azure App Service

Tato dokumentace popisuje, jak nasadit aplikaci WebAppPrize na Azure App Service.

## Příslušné soubory

- `.github/workflows/azure-app-deploy.yml` - GitHub Actions workflow pro automatické nasazení
- `web.config` - Konfigurace pro Azure App Service (IIS)

## Možnost 1: Automatické nasazení přes GitHub Actions (Doporučeno)

### Kroky:

1. **Přihlaste se na Azure Portal:**
   ```
   https://portal.azure.com
   ```

2. **Vytvořte App Service (pokud neexistuje):**
   - Resource Group: vaše RG
   - Name: např. `webapp-prize-app`
   - Publish: Code
   - Runtime stack: Node 18 LTS
   - Region: vaše preferovaná

3. **Získejte Publish Profile:**
   - V Azure Portalu jděte na App Service
   - Klikněte na "Download publish profile"
   - Uložte tento soubor

4. **Přidejte GitHub Secret:**
   - Jděte do Settings → Secrets and variables → Actions
   - Klikněte "New repository secret"
   - Name: `AZURE_PUBLISH_PROFILE`
   - Value: Obsah publikačního profilu (obsah souboru)
   - Klikněte "Add secret"

5. **Přidejte další GitHub Secret:**
   - Name: `AZURE_APP_NAME`
   - Value: Název vaší App Service (bez .azurewebsites.net)
   - Klikněte "Add secret"

6. **Push ke GitHub:**
   ```bash
   git add .
   git commit -m "Add Azure deployment configuration"
   git push origin main
   ```

7. **Monitorujte deployment:**
   - Jděte na záložku Actions v GitHub
   - Sledujte běh workflow
   - Po úspěšném nasazení bude aplikace dostupná na:
     `https://[AZURE_APP_NAME].azurewebsites.net`

## Možnost 2: Ruční nasazení přes Azure CLI

```bash
# Přihlášení
az login

# Vytvoření Resource Group (pokud neexistuje)
az group create --name myResourceGroup --location eastus

# Vytvoření App Service Plan
az appservice plan create \
  --name myAppServicePlan \
  --resource-group myResourceGroup \
  --sku B1 \
  --is-linux

# Vytvoření Web App
az webapp create \
  --resource-group myResourceGroup \
  --plan myAppServicePlan \
  --name webapp-prize-app \
  --runtime "node|18-lts"

# Nasazení kódu
az webapp deployment source config-zip \
  --resource-group myResourceGroup \
  --name webapp-prize-app \
  --src webappcode.zip
```

## Možnost 3: Nasazení přes Git

```bash
# Přihlášení k Azure
az loginaz webapp deployment user set --user-name <username> --password <password>

# Přidejte Azure Git remote
az webapp deployment source config-local-git \
  --name webapp-prize-app \
  --resource-group myResourceGroup

# Přidejte Azure jako remote
git remote add azure https://<username>@<app-name>.scm.azurewebsites.net:443/<app-name>.git

# Push do Azure
git push azure main
```

## Proměnné prostředí

Pokud aplikace potřebuje proměnné (např. databázové kredence), přidejte je v Azure Portalu:
- App Service → Configuration → Application settings
- Přidejte klíč-hodnota páry dle potřeby

## Monitoring

Po nasazení můžete sledovat:
- Azure Portal → App Service → Logs
- Live Metrics Stream
- Application Insights

## Řešení problémů

1. **Deployment se nezdařil v GitHub Actions:**
   - Zkontrolujte GitHub Secrets
   - Ověřte, že Publish Profile je správný

2. **Aplikace vykazuje chyby:**
   - Podívejte se na Log Stream v Azure Portalu
   - Zkontrolujte Application Insights

3. **Port:**
   - Azure App Service automaticky směruje PORT 3000 na port 80
   - Aplikace poslouchá na portu 3000 (správně)

## Aktualizace aplikace

Pokaždé, když pushujete změny do `main` branch, GitHub Actions workflow:
1. Nainstaluje dependencies (`npm ci`)
2. Nasadí aplikaci na Azure
3. Restartuje App Service

Není potřeba nic dalšího!
