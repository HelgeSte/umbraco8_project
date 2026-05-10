# Claude Code Instructions: Upgrade Umbraco 8.18.15 → Umbraco 9

## Context
- Source: Umbraco 8.18.15, ASP.NET (.NET Framework 4.7.2), Visual Studio solution
- Database: SQL Server
- Content: Official Umbraco Starter Kit 4.1.0 (Umbraco HQ)
- Target: Umbraco 9.5.4 (latest stable v9)

This is the most significant step in the entire upgrade journey.
Umbraco 9 moves from .NET Framework to .NET 5 / ASP.NET Core.
This means the v8 project CANNOT be upgraded in-place — a new v9 project
must be created and the database and content migrated across.

---

## Phase 0 — Reconnaissance (read-only, no changes yet)

### 0.1 — Read the existing v8 project structure
Scan the solution folder and report:
- The solution (.sln) file location
- The main web project folder and .csproj path
- Connection string from web.config (server name and database name)
- Umbraco version confirmed in web.config (umbracoConfigurationStatus)
- Any custom files in /Views/, /Controllers/, /Models/, /App_Plugins/
- The starter kit templates in /Views/

### 0.2 — Check .NET SDK
Run:
```
dotnet --version
```
Report the installed .NET SDK version.
Umbraco 9.5.4 requires .NET 5.0 SDK minimum.
If .NET 5 or higher is not installed, stop and report — it must be installed
before continuing. Download from: https://dotnet.microsoft.com/download/dotnet/5.0

### 0.3 — Back up the database
Using the connection string from web.config, create a full SQL Server backup:
```sql
BACKUP DATABASE [YourDatabaseName]
TO DISK = 'C:\Backups\umbraco_v8_preupgrade_backup.bak'
WITH FORMAT, COMPRESSION, STATS = 10;
```
Confirm the backup completed successfully before proceeding.

---

## Phase 1 — Create a new Umbraco 9 project

### 1.1 — Install the Umbraco .NET templates
```
dotnet new install Umbraco.Templates::9.5.4
```

### 1.2 — Create the new project
Create the new project as a sibling folder to the v8 project:
```
dotnet new umbraco -n UmbracoV9 --friendly-name "Administrator" --email "admin@example.com" --password "UmbracoPassword123!" --connection-string "Server=YOUR_SERVER;Database=YOUR_DATABASE;Integrated Security=true;" --connection-string-provider-name "Microsoft.Data.SqlClient"
```

Replace YOUR_SERVER and YOUR_DATABASE with the values from the v8 web.config connection string.

This creates a ready-to-run Umbraco 9 project pointing at the existing v8 database.
Umbraco 9 will automatically run the database migration on first startup.

### 1.3 — Move into the new project folder
```
cd UmbracoV9
```

---

## Phase 2 — Run the database migration

### 2.1 — Start the site to trigger migration
```
dotnet run
```

Umbraco 9 will detect the v8 database schema and run all migration steps automatically.
This may take several minutes. Watch the console output for errors.

When you see output like:
`Now listening on: https://localhost:44391`
the migration is complete.

### 2.2 — Verify in the backoffice
Open the URL shown in the console output and navigate to /umbraco.
Log in and confirm:
- Content tree is intact (starter kit content is present)
- Media library is intact
- Document types are present
- Settings look correct

---

## Phase 3 — Migrate starter kit assets

The official Umbraco starter kit assets (views, CSS, JS) need to be
brought across from the v8 project into the v9 structure.

### 3.1 — Copy views
In Umbraco 9 (ASP.NET Core), views live in /Views/ just like v8,
but the syntax has changed slightly.

Copy all .cshtml files from:
`[v8 project]/Views/`
to:
`UmbracoV9/Views/`

Then update each view file:
- Remove `@inherits Umbraco.Web.Mvc.UmbracoViewPage<ContentModels.X>`
  Replace with `@inherits Umbraco.Cms.Web.Common.Views.UmbracoViewPage<ContentModels.X>`
- Remove `@using ContentModels = Umbraco.Web.PublishedModels;`
  Replace with `@using ContentModels = Umbraco.Cms.Web.Common.PublishedModels;`
- Replace any `@Umbraco.AssignedContentItem` with `@Model`
- Replace `UmbracoContext.Current` with injected `IUmbracoContextAccessor`

### 3.2 — Copy static assets
Copy from v8 to v9 into the /wwwroot/ folder (new in ASP.NET Core):

| v8 source | v9 destination |
|---|---|
| /css/ | /wwwroot/css/ |
| /scripts/ | /wwwroot/scripts/ |
| /fonts/ | /wwwroot/fonts/ |
| /images/ | /wwwroot/images/ |
| /media/ | /wwwroot/media/ |

### 3.3 — Copy App_Plugins
If any App_Plugins exist from the starter kit:
Copy from `[v8]/App_Plugins/` to `UmbracoV9/App_Plugins/`

---

## Phase 4 — Update configuration

### 4.1 — appsettings.json
In Umbraco 9, configuration moves from web.config XML to appsettings.json.
The new project already has a generated appsettings.json with the connection string.
Verify the connection string is correct and the database name matches the v8 database.

### 4.2 — Check Umbraco:CMS settings
In appsettings.json, verify these settings are present:
```json
{
  "Umbraco": {
    "CMS": {
      "Global": {
        "MainDomLock": "FileSystemMainDomLock"
      }
    }
  }
}
```

---

## Phase 5 — Build and verify

### 5.1 — Build the project
```
dotnet build
```
Fix any compilation errors before proceeding. Common issues:
- Namespace changes (Umbraco.Web.* → Umbraco.Cms.*)
- Removed APIs (UmbracoHelper methods that changed)
- Startup.cs / Program.cs differences

### 5.2 — Run and test
```
dotnet run
```

Test the following:
- Front-end renders correctly
- All starter kit pages load
- Navigation works
- Images and media load from /wwwroot/media/
- Backoffice login works
- Content tree matches v8

### 5.3 — Check logs
Review /umbraco/Logs/ for any warnings or errors after startup.

---

## Phase 6 — Git

### 6.1 — Initialise Git for the new project
```
cd E:\claude-ai\UmbracoV9
git init
git checkout -b main
copy E:\claude-ai\umbraco-v9-plus.gitignore .gitignore
git add .
git commit -m "Umbraco 9.5.4 — migrated from v8 with starter kit"
git remote add origin git@github.com:HelgeSte/umbraco9_project.git
git push -u origin main
```

Use the umbraco-v9-plus.gitignore file as .gitignore before committing.

---

## Key namespace changes reference (v8 → v9)

| v8 | v9 |
|---|---|
| `Umbraco.Web` | `Umbraco.Cms.Web.Common` |
| `Umbraco.Core` | `Umbraco.Cms.Core` |
| `Umbraco.Web.Mvc.UmbracoViewPage` | `Umbraco.Cms.Web.Common.Views.UmbracoViewPage` |
| `Umbraco.Web.PublishedModels` | `Umbraco.Cms.Web.Common.PublishedModels` |
| `ApplicationEventHandler` | `IComposer` |
| `web.config` | `appsettings.json` |
| `/wwwroot/` didn't exist | All static files go in `/wwwroot/` |

---

## Notes
- If dotnet run fails with a certificate error, run: `dotnet dev-certs https --trust`
- If the database migration fails, restore from the backup created in Phase 0.3 and report the error
- The starter kit v8 package (4.1.0) does not have a v9 equivalent on NuGet —
  its content and templates are migrated manually as described in Phase 3
