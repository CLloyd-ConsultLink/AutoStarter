# Workspace Manager

Workspace Manager is a Windows 10/11 x64 desktop app for defining reusable groups of applications, websites, and files, then opening them across Windows virtual desktops. It supports configurable workspace profiles and a separate, ordered launch sequence.

## Install the app

Download and run `WorkspaceLauncher-Setup-win-x64.exe`. The setup wizard installs the app for the current Windows user, creates a **Workspace Manager** Start Menu shortcut, and offers an optional desktop shortcut. Administrator access and the .NET SDK are not required. The setup filename and internal installation/configuration paths retain their original `WorkspaceLauncher` names to support in-place upgrades and preserve existing workspace settings.

The installer is for Windows x64.

Workspace Manager checks GitHub Releases for a newer stable version when it starts. If one is available, the app asks whether to download and start the installer. Choosing **No** postpones the update until the next time the app starts. Choosing **Yes** downloads the installer, checks its SHA-256 digest against GitHub's release metadata, closes Workspace Manager, and opens the normal setup wizard. The check needs an internet connection; if GitHub cannot be reached, you can continue using the installed version and the app will check again next time. You can also update manually by running a newer setup file and closing the app first.

The SHA-256 check detects a download that differs from the file GitHub published, but it does not independently verify the publisher's identity. The setup program is not currently code-signed, so Windows SmartScreen may show a warning.

To uninstall, use **Settings > Apps > Installed apps** (or **Apps & features**) and uninstall **Workspace Manager**. Uninstalling removes the installed app and shortcuts, but leaves your workspace configuration in your user profile.

## Use the app

On first run, the app starts with no workspaces, launch items, or websites. Add only the profiles and launch targets you want. Updating an existing installation preserves its saved configuration; users who already have the previous sample profiles can delete them in the app, including the last profile, to return to a clean slate.

### Edit workspaces

- Choose **+ New** to create a workspace; edit its name and choose a virtual desktop.
- Choose **+ Add launch item** to open one dialog and select **Application or shortcut**, **Website**, or **File**. Use **Browse...** to select an `.exe` or `.lnk` for an application, or any existing file for a file item. File items open with the default Windows app for that file type, including PDFs, Office documents, and other registered file types.
- The launch-item table is read-only. Double-click any row to edit it in the same launch-item dialog. For applications that start their actual window in a child process, set the process name to that child app (for example, `javaw` for a Java application). For files, Workspace Manager attempts to determine the default app process from the file association and uses the file name to locate the document window. If detection is not correct, set **Process name** and **Window title contains** in the item settings.
- Select an item and use **Move up** or **Move down** to change its launch order. Use **Remove** to delete the selected item, then choose **Save changes**.
- Choose **Delete workspace** to remove the selected profile. The workspace list may be left empty.

Hover over controls for brief tips. When there are no workspaces or the selected workspace has no launch items, the app shows a getting-started hint.

### Configure and launch a sequence

Choose **Configure sequence** to move profiles into or out of the sequence, arrange their order, and choose the desktop to leave active when the sequence finishes. The sequence can contain any subset of your workspaces. Choose **Save sequence** to save it, then **Launch sequence** to run each selected workspace in order. Each workspace switches to its assigned desktop before opening its items. By default the app remains on the last workspace's desktop; select a specific desktop in the sequence settings to end there instead.

You can also choose **Launch workspace** in the editor to run just the selected workspace.

During launch, websites open in the default browser. Applications with an existing matching visible window are skipped; background-only processes are not treated as already open. After starting an app with a detectable process name, the launcher waits up to 120 seconds for its visible window and moves it to the workspace's assigned desktop, even if you switched desktops while it was starting. It prefers windows belonging to the launched process tree and its children, but can also find a window opened by another instance of the configured process (for example, when an app reuses a background process). If the window is not detected or cannot be moved, Workspace Activity reports a warning. Applications with no resolvable process name and websites opened in the default browser cannot be moved reliably. Missing application files and invalid websites are also reported in Workspace Activity.

Choose **Close workspace** to select which configured applications and files to close on that workspace's assigned desktop. All application and file entries are checked by default; uncheck any to leave it open. Website entries are not included because the app cannot safely target an individual browser tab. For file items, the app matches the default app process and the file name (or your optional window-title filter), then requests a normal close for matching visible document windows. If a file is open in a browser, this can close its entire containing browser window, including other tabs. The app does not force-terminate applications or remove the virtual desktop. Apps may show their own unsaved-work prompts. If Windows does not expose a usable file association, or if the document window title does not contain the file name, placement or targeted close may not be available; set the process name or window title manually when needed. Matching windows on other virtual desktops are left alone.

## Virtual desktop behavior

The app detects the current number of Windows virtual desktops when it starts and refreshes the desktop choices whenever you open the desktop dropdown. If the count has decreased since a workspace was configured, its desktop is adjusted to the last available desktop and the configuration is saved. You do not need to restart the app after adding or removing virtual desktops. For apps launched by Workspace Manager, late-opening windows are moved to the assigned desktop after they appear; this does not affect apps launched manually outside the app.

Desktop detection and switching use Windows shell interfaces plus keyboard input. Moving a window between processes uses an internal Windows shell interface that may vary between Windows builds. The installed helper is `workspace_desktop.ps1`, stored beside the app executable. If detection, switching, or window relocation is unavailable on a particular Windows build, the app reports an error or warning rather than silently claiming success.

## Developer: run locally

Requirements: Windows 10/11 x64 and the .NET 10 SDK.

From the repository root, run:

```powershell
dotnet run --project WorkspaceLauncher\WorkspaceLauncher.csproj
```

## Developer: build the installer

Requirements: Windows, the .NET 10 SDK, and Inno Setup 6.

Build the self-contained x64 setup program from the repository root:

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\Publish-WorkspaceLauncher.ps1
```

The script publishes the app, includes its PowerShell helper, and builds:

```text
artifacts\WorkspaceLauncher-Setup-win-x64.exe
```

Share that single setup file with users. Intermediate publish files are stored in `artifacts\WorkspaceLauncher-win-x64`; the `artifacts` directory is excluded from Git.

### Publish an update

The app version in `WorkspaceLauncher\WorkspaceLauncher.csproj` is the source of truth for both the app and installer. Increase it using `major.minor.patch` format, build the installer, and publish a stable GitHub Release in `CLloyd-ConsultLink/Workspace-Launcher` with a matching `v`-prefixed tag (for example, version `1.4.0` uses tag `v1.4.0`). Attach the generated `WorkspaceLauncher-Setup-win-x64.exe` file. Keep this asset filename so existing installations can download updates. The app checks the latest stable release only; drafts and prereleases are not offered.

## Configuration and legacy launcher

Workspace settings are stored per user at:

```text
%LOCALAPPDATA%\WorkspaceLauncher\workspaces.json
```

The `workspace_launcher.bat` remains in the repository as a legacy command-line launcher with its original fixed app groups. Its console title uses the Workspace Manager brand. Its GitHub link is generic, and its YouTube Music shortcut lookup uses the current user's `%APPDATA%` path. The WPF app is the recommended way to configure profiles and sequences.
