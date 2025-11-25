# Novu Source Code Management & Build Guide

This guide explains how to manage our custom version of Novu (with branding removed) and how to build it from source.

## 🏗️ Architecture Overview

We use a **Submodule + Fork** architecture to maintain our custom changes while keeping the ability to pull updates from the official Novu project.

- **Official Repo:** `https://github.com/novuhq/novu` (Read-Only upstream)
- **Our Fork:** `https://github.com/brigandx/novu` (Write access, contains our "No Branding" changes)
- **Main Project:** `MidyeciHub2` tracks our fork as a submodule in the `novu/` directory.

---

## 🔄 Workflow 1: Updating Novu (Merging Upstream Changes)

When Novu releases a new version, follow these steps to update our fork while keeping our changes.

1.  **Enter the Submodule:**
    ```bash
    cd novu
    ```

2.  **Pull from Official Upstream:**
    ```bash
    git pull https://github.com/novuhq/novu.git master
    ```
    *Resolve any merge conflicts if they occur (usually unlikely unless they changed the Footer component).*

3.  **Push to Our Fork:**
    ```bash
    git push origin next
    ```

4.  **Update Main Project:**
    ```bash
    cd ..
    git add novu
    git commit -m "Update Novu submodule to latest version"
    ```

---

## 🛠️ Workflow 2: Building from Source (WSL Required)

Because of file locking issues between Windows/OneDrive and WSL, we **cannot** build directly in the mounted `c:\Users\...` directory. We must build in the Linux filesystem.

### Prerequisites
- WSL2 (Ubuntu) installed.
- `pnpm` installed in WSL (`npm install -g pnpm`).

### Build Steps

1.  **Open WSL Terminal:**
    ```powershell
    wsl -d Ubuntu
    ```

2.  **Copy Source to Linux Filesystem:**
    *This bypasses the slow/locking Windows file system.*
    ```bash
    # Create directory if not exists
    mkdir -p ~/novu

    # Sync source code (excluding heavy node_modules)
    rsync -av --exclude='node_modules' --exclude='.git' /mnt/c/Users/hakan/OneDrive/Documents/GitHub/MidyeciHub2/novu/ ~/novu/
    ```

3.  **Install Dependencies & Build:**
    ```bash
    cd ~/novu
    pnpm install
    
    # Build the specific packages we need
    cd packages/js
    pnpm run build
    
    cd ../react
    pnpm run build
    ```

4.  **Copy Artifacts Back to Windows:**
    ```bash
    # Copy the 'dist' folders back to your project
    cp -r ~/novu/packages/js/dist /mnt/c/Users/hakan/OneDrive/Documents/GitHub/MidyeciHub2/novu/packages/js/
    cp -r ~/novu/packages/react/dist /mnt/c/Users/hakan/OneDrive/Documents/GitHub/MidyeciHub2/novu/packages/react/
    ```

5.  **Done!** Your client app will automatically pick up the new built files because of the link in `package.json`.

---

## 🔗 Client Configuration

The client is configured to use the local built packages instead of downloading them from npm.

**`client/package.json`:**
```json
"dependencies": {
  "@novu/js": "file:../novu/packages/js",
  "@novu/react": "file:../novu/packages/react"
}
```

If you ever want to revert to the official (branded) version, just change these lines back to version numbers (e.g., `^2.0.0`) and run `npm install`.
