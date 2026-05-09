# Claude Code Instructions: Create a Fresh Umbraco 8 Project (NuGet)

## Goal
Create a working Umbraco 8.18.15 (latest stable v8) project using Visual Studio and NuGet.
This is a .NET Framework 4.7.2 project (NOT .NET Core — that starts at v9).

---

## Prerequisites (verify these exist before starting)
- Visual Studio 2017 (15.9.6 minimum) or Visual Studio 2019/2022
- .NET Framework 4.7.2 installed
- SQL Server (any edition) or SQL Server Express — note the server name
- IIS or IIS Express (IIS Express ships with Visual Studio)

---

## Step 1 — Create a new Visual Studio project

1. Open Visual Studio **as Administrator** (right-click → Run as administrator)
2. File → New → Project
3. Select **ASP.NET Web Application (.NET Framework)**
4. Configure the project:
   - **Name:** `UmbracoV8` (do NOT use `.Umbraco` anywhere in the name)
   - **Framework:** `.NET Framework 4.7.2` ← this is critical, v8 will not work on 4.6.x
   - Click **OK**
5. On the next screen, select **Empty** template
   - Leave all checkboxes (MVC, Web API etc.) **unchecked**
   - Click **OK**

---

## Step 2 — Install Umbraco via NuGet

Open the Package Manager Console:
**Tools → NuGet Package Manager → Package Manager Console**

Make sure the **Default project** dropdown at the top of the console
is set to your `UmbracoV8` project.

Run this exact command:

```powershell
Install-Package UmbracoCms -Version 8.18.15
```

This will:
- Download and install Umbraco 8.18.15 and all its dependencies
- Modify `web.config` automatically
- Add all required Umbraco folders and files
- Create a `packages.config` file tracking all installed packages

When prompted to overwrite `Global.asax` — answer **Yes to All (A)**.

Wait for it to complete. This may take several minutes as v8 has more dependencies than v7.

If you see a prompt about `Microsoft.CodeDom.Providers.DotNetCompilerPlatform`,
also run:
```powershell
Install-Package Microsoft.CodeDom.Providers.DotNetCompilerPlatform
```

---

## Step 3 — Build the solution

```
Build → Build Solution (Ctrl+Shift+B)
```

The build must succeed with **0 errors** before continuing.
If there are errors, report them here before proceeding.

---

## Step 4 — Create a SQL Server database

In SQL Server Management Studio (SSMS):
1. Connect to your SQL Server instance
2. Right-click **Databases** → **New Database**
3. Name it: `UmbracoV8`
4. Click **OK**

Note down:
- Server name (e.g. `.\SQLEXPRESS` or `localhost`)
- Database name: `UmbracoV8`
- Authentication method (Windows Auth recommended for local dev)

> ⚠️ Umbraco 8 does NOT support MySQL. Use SQL Server or SQL CE only.
> SQL CE (.sdf) is fine for local testing but not recommended for production.

---

## Step 5 — Run the Umbraco installer

1. Press **F5** (or Debug → Start Without Debugging) to launch the site
2. The browser will open the Umbraco installation wizard
3. Fill in:
   - **Name:** your name
   - **Email:** your email
   - **Password:** a strong password (remember this — it's the admin login)
4. Click **Customize** on the database screen
5. Select **Microsoft SQL Server**
6. Enter your server name and database name from Step 4
7. Select **Windows Authentication** (or enter SQL credentials if using SQL auth)
8. Click **Continue**
9. When asked about a starter kit — click **"No thanks, I just want a clean install"**
   unless you want a demo site to explore

The installer will create all database tables. This takes 1–3 minutes.

---

## Step 6 — Verify the installation

After the installer completes:
- You should be redirected to the Umbraco backoffice at `/umbraco`
- Log in with the email and password from Step 5
- Confirm the dashboard loads without errors
- Check that the Content, Media, and Settings sections are visible in the left nav

To view the front-end site, remove `/umbraco` from the URL.

---

## Step 7 — Set up Git (recommended)

Initialize a Git repository and apply the official Umbraco `.gitignore`
so generated files and binaries are not committed.

The key folders to exclude from Git:
```
/bin/
/obj/
/App_Data/TEMP/
/App_Data/Logs/
/umbraco/
/umbraco_client/
```

Download the official Umbraco .gitignore from:
https://github.com/github/gitignore/blob/main/Umbraco.gitignore

The `packages.config` file IS committed to Git — this allows NuGet to restore
all packages on any machine after a fresh clone.

---

## ⚠️ Important notes

- **Always run Visual Studio as Administrator**
- **.NET Framework 4.7.2 is required** — v8 will fail on 4.6.x with assembly errors
- **Never put `.Umbraco` in the project or solution name**
- **After any fresh Git clone:** always run
  1. Right-click solution → Restore NuGet Packages
  2. Build → Rebuild Solution
  This restores all DLLs to `/bin/` which Git does not track
- **Visual Studio 2017 minimum version is 15.9.6** — older 2017 builds may have issues

---

## Key differences from Umbraco 7

| Feature | Umbraco 7 | Umbraco 8 |
|---|---|---|
| .NET Framework | 4.5+ | 4.7.2 required |
| Models | `GetPropertyValue("alias")` | `Value("alias", culture)` |
| Templates inherit from | `UmbracoTemplatePage` | `UmbracoViewPage` |
| Published content namespace | `PublishedContentModels` | `PublishedModels` |
| Event handlers | `ApplicationEventHandler` | `IUserComposer` / `IComponent` |
| MySQL support | Yes | No |

---

## Troubleshooting

| Error | Fix |
|---|---|
| `Could not load file or assembly 'Umbraco.ModelsBuilder'` | Rebuild Solution — bin is empty |
| `Could not load DotNetCompilerPlatform` | Run `Install-Package Microsoft.CodeDom.Providers.DotNetCompilerPlatform` |
| `Update-Package` says package not installed | Check Default project dropdown in PM Console |
| Installer fails on database step | Verify .NET Framework is 4.7.2; check SQL Server is running |
| Assembly version mismatch errors | Delete `/bin` and `/obj` folders, then Rebuild Solution |
| Site shows IIS error after install | Run VS as Administrator |
