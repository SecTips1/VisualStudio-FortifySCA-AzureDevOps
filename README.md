# Integrating Microsoft Visual Studio & Fortify SCA with Azure DevOps

This document provides a step-by-step guide on how to integrate **Microsoft Visual Studio** with **Fortify Static Code Analyzer (SCA)** in an Azure DevOps repository.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Install and Configure Fortify SCA](#install-and-configure-fortify-sca)
3. [Optional: Install the Fortify Extension for Visual Studio](#optional-install-the-fortify-extension-for-visual-studio)
4. [Connect Visual Studio to Azure DevOps Repository](#connect-visual-studio-to-azure-devops-repository)
5. [Run a Local Scan with Fortify SCA](#run-a-local-scan-with-fortify-sca)
6. [Create or Update an Azure DevOps Pipeline for Fortify Scans](#create-or-update-an-azure-devops-pipeline-for-fortify-scans)
7. [View and Manage Fortify Results](#view-and-manage-fortify-results)
8. [Additional Tips and Best Practices](#additional-tips-and-best-practices)

---

## Prerequisites

Before you begin, ensure you have:

- **Microsoft Visual Studio** (2019 or later)  
- **Fortify Static Code Analyzer (SCA)** installed on your local machine  
- **(Optional) Fortify extension/plugin for Visual Studio**  
- **Azure DevOps** organization with a Git repository (and local clone)  
- **Azure DevOps Build Agent** (hosted or self-hosted) for pipeline runs  
- **(Recommended) Fortify Software Security Center (SSC)** for enterprise use

---

## Install and Configure Fortify SCA

1. **Download and Install Fortify SCA**  
   - Obtain the installer from your Fortify (OpenText) portal or your security team.  
   - Follow the on-screen instructions to complete the installation.

2. **Add Fortify to the System PATH**  
   - Locate the **bin** folder (e.g., `C:\Program Files\Fortify\Fortify_SCA\<version>\bin`).  
   - Add this folder to your system’s `PATH` environment variable.

3. **Verify Installation**  
   - Open **Command Prompt** or **PowerShell** and run:  
     ```bash
     sourceanalyzer -h
     ```
   - If it displays the Fortify SCA help menu, you are set.

---

## Optional: Install the Fortify Extension for Visual Studio

1. **Extensions Manager**  
   - In Visual Studio, go to **Extensions > Manage Extensions**.

2. **Search for Fortify**  
   - Look for **Micro Focus Fortify** or “Fortify” and install the extension.

3. **Restart Visual Studio**  
   - After installation, restart Visual Studio to activate the extension.

4. **Configure Extension (Optional)**  
   - Navigate to **Tools > Fortify > Options** to set advanced properties like memory usage or translation settings.

---

## Connect Visual Studio to Azure DevOps Repository

1. **Clone the Repository**  
   - In Visual Studio, select **Clone a repository** (or use **Team Explorer**) and enter your **Azure DevOps** repository URL.

2. **Build and Test the Solution**  
   - Open the solution and ensure it builds successfully (no errors).

3. **Verify Git Connectivity**  
   - Make a small code change, commit, and push to confirm proper Git permissions.

---

## Run a Local Scan with Fortify SCA

1. **Open a Command Prompt**  
   - Navigate to your solution’s root folder (where the `.sln` file is located).

2. **Translation Phase**  
   - Translate the code with a Fortify build ID (e.g., `MyProjectBuild`):  
     ```bash
     sourceanalyzer -b MyProjectBuild msbuild MySolution.sln
     ```

3. **Scan Phase**  
   - After translation completes, run:  
     ```bash
     sourceanalyzer -b MyProjectBuild -scan -f MyProjectResults.fpr
     ```

4. **Review Findings**  
   - Open the resulting `MyProjectResults.fpr` in **Fortify Audit Workbench** (installed with Fortify SCA) or in Visual Studio (via the **Tools > Fortify > Open FPR** option).

---

## Create or Update an Azure DevOps Pipeline for Fortify Scans

### Install the Fortify Extension on Azure DevOps (Optional)

1. **Azure DevOps Marketplace**  
   - Go to **Organization Settings > Extensions** in Azure DevOps or directly to the Marketplace.

2. **Install Fortify Tasks**  
   - Search for **Fortify** or **Micro Focus Fortify** and install the extension.

3. **Confirm Installation**  
   - Check that new Fortify tasks (e.g., “Fortify Static Code Analyzer - Scan”) appear in your pipeline editor.

### Example YAML Pipeline

Add the following to a file named `azure-pipelines.yml` in your repository:

```yaml
trigger:
  - main

pool:
  vmImage: 'windows-latest'

steps:
- task: NuGetToolInstaller@1
  inputs:
    checkLatest: true

- task: NuGetCommand@2
  inputs:
    command: 'restore'
    feedsToUse: 'select'
    projects: '**/*.sln'

- task: VSBuild@1
  inputs:
    solution: '**/*.sln'
    configuration: 'Release'

- task: PowerShell@2
  displayName: 'Fortify Translation'
  inputs:
    targetType: 'inline'
    script: |
      cd "$(Build.SourcesDirectory)"
      sourceanalyzer -b MyProjectBuild msbuild **/*.sln

- task: PowerShell@2
  displayName: 'Fortify Scan'
  inputs:
    targetType: 'inline'
    script: |
      sourceanalyzer -b MyProjectBuild -scan -f "$(Build.ArtifactStagingDirectory)\MyProjectResults.fpr"

- task: PublishBuildArtifacts@1
  inputs:
    pathToPublish: '$(Build.ArtifactStagingDirectory)'
    artifactName: 'FortifyResults'
