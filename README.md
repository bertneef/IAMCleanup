# Azure Permissions Viewer & Cleanup Tool

A self-contained HTML tool for analyzing and managing Azure RBAC and Entra ID role assignments. This tool allows you to view all users with permissions in your Azure environment and generate cleanup scripts to remove those permissions.

## Original Request

> I am trying to build a self contained tool, preferably in the format of a single HTML file with CSS and Javascript which allows a user to login to Azure/Entra ID and get a list of users with permissions in Azure, directly or indirectly, permanent or on-demand or permissions in Entra ID. I want to show the user a list of users/permissions (permissions may be summarized, e.g. if the user has owner on multiple resource groups just say so with the number of resource groups and no need to specify which resource groups) and allow the user to select users which should be cleaned up (which consists of two actions: remove any role assignments and optionally: remove them from Entra ID), this should be in the form of a script they can copy/paste into something like an Azure cloud shell, so powershell or Azure CLI.

## 🚀 Quick Start

### Prerequisites

Before using this tool, you need to register an Azure AD application. This is a one-time setup that takes about 5 minutes.

### Step 1: Register Azure AD Application

1. **Go to Azure Portal**: Navigate to [Azure Active Directory → App registrations](https://portal.azure.com/#view/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/~/RegisteredApps)

2. **Create New Registration**:
   - Click **"New registration"**
   - **Name**: `Azure Permissions Viewer` (or any name you prefer)
   - **Supported account types**: Select "Accounts in this organizational directory only" (single tenant)
   - **Redirect URI**:
     - Select **"Single-page application (SPA)"** from the dropdown
     - Enter the URL where you'll host this file
     - If running locally, use: `file:///C:/path/to/azure-permissions-viewer.html` (Windows) or `file:///Users/path/to/azure-permissions-viewer.html` (Mac/Linux)
     - Or use: `http://localhost:8000/azure-permissions-viewer.html` if serving locally
   - Click **"Register"**

3. **Copy Application Details**:
   - On the Overview page, copy the **"Application (client) ID"** - you'll need this
   - Copy the **"Directory (tenant) ID"** - you'll need this too

4. **Configure API Permissions** (Two Options):

   **Option A: User-Delegated Permissions Only (No Admin Consent Required)** ⭐ Recommended

   Skip adding any API permissions in the Azure Portal. The app will request permissions when users sign in, and users will only be able to access data they already have permissions to view through their existing Azure roles.

   - ✅ No admin consent needed
   - ✅ Each user consents for themselves
   - ✅ Users can only see what they already have access to
   - ⚠️ Requires users to have appropriate directory roles (e.g., Global Reader, Directory Readers, Privileged Role Administrator)
   - ⚠️ If a user lacks permissions, they'll see errors when the tool tries to fetch data

   **Option B: Pre-Approved App Permissions (Admin Consent)**

   If you want to pre-approve permissions for all users:
   - Go to **"API permissions"** in the left menu
   - Click **"Add a permission"**
   - Select **"Microsoft Graph"** → **"Delegated permissions"**
   - Search for and add these permissions:
     - ✅ `Directory.Read.All`
     - ✅ `User.Read.All`
     - ✅ `RoleManagement.Read.Directory`
   - Click **"Add permissions"**
   - Now click **"Add a permission"** again
   - Select **"Azure Service Management"** → **"Delegated permissions"**
   - Add: ✅ `user_impersonation`
   - Click **"Add permissions"**
   - Finally, click **"Grant admin consent for [Your Organization]"** (requires admin privileges)
   - Confirm by clicking **"Yes"**

   With this option, users won't see permission consent prompts, but the app can still only access data the user has permissions to view.

5. **Verify Authentication Settings**:
   - Go to **"Authentication"** in the left menu
   - Under "Implicit grant and hybrid flows", ensure **nothing is checked** (we use PKCE flow)
   - Under "Advanced settings" → "Allow public client flows", ensure this is set to **"No"**

### Step 2: Configure the Tool

1. Open `azure-permissions-viewer.html` in your web browser
2. Click **"⚙️ Configure App"**
3. Enter your **Client ID** (Application ID from step 3)
4. Enter your **Tenant ID** (Directory ID from step 3), or leave as "common" for multi-tenant
5. Click **"Save & Sign In"**

The tool will save your configuration in browser local storage, so you only need to do this once per browser.

## Features

- **Single HTML File**: No installation required, just open the HTML file in a browser
- **Configurable Authentication**: Use your own Azure AD application for secure authentication
- **Azure AD Authentication**: Secure login using Microsoft Authentication Library (MSAL.js)
- **Comprehensive Permission Discovery**:
  - Azure RBAC role assignments across all subscriptions
  - Entra ID directory roles (permanent assignments)
  - PIM (Privileged Identity Management) eligible roles
- **Permission Summary**: Intelligently summarizes permissions (e.g., "Owner (5 Resource Groups)")
- **User Selection**: Easy checkbox interface to select users for cleanup
- **Script Generation**: Generates PowerShell or Azure CLI scripts for cleanup
- **Flexible Options**: Choose to remove role assignments only or also delete users from Entra ID

## How to Use

### Step 1: Open the Tool

1. Download or save `azure-permissions-viewer.html` to your computer
2. Open the file in a modern web browser (Chrome, Edge, Firefox, Safari)

### Step 2: Configure (First Time Only)

If this is your first time using the tool:
1. Click **"⚙️ Configure App"**
2. Follow the on-screen instructions to register an Azure AD app (see Quick Start section above)
3. Enter your Client ID and Tenant ID
4. Click **"Save & Sign In"**

### Step 3: Sign In

1. Click **"Sign In with Microsoft"**
2. Authenticate with your Azure account
3. Grant the requested permissions when prompted (if not already granted via admin consent)

### Step 4: Review Permissions

1. The tool will automatically load all users with permissions
2. Use the search box to filter users by name, email, or permission type
3. Review the Azure RBAC and Entra ID roles for each user

### Step 5: Select Users for Cleanup

1. Check the boxes next to users you want to remove
2. Or use "Select All" to select all users

### Step 6: Configure Cleanup Options

1. Choose script type:
   - **PowerShell (Az module)**: For use with Azure PowerShell
   - **Azure CLI (bash)**: For use with Azure Cloud Shell or bash
2. Options:
   - **Delete users from Entra ID**: Check this to permanently delete users (not just remove permissions)
   - **Include comments**: Add explanatory comments to the script

### Step 7: Copy and Execute Script

1. Click "Copy Script to Clipboard"
2. Open Azure Cloud Shell (https://shell.azure.com) or your local terminal
3. Paste and review the script carefully
4. Execute the script to perform the cleanup

## Required Permissions

### To View Permissions (Read-Only)

Your Azure account needs the following permissions to use this tool:

**For Entra ID Data:**
- **Global Reader** (recommended) - Can read all directory data
- **Directory Readers** - Can read basic directory information
- **Privileged Role Administrator** - Can read role assignments
- Or any role with these Microsoft Graph permissions:
  - `Directory.Read.All`
  - `User.Read.All`
  - `RoleManagement.Read.Directory`

**For Azure RBAC Data:**
- **Reader** role at the subscription or management group level
- This allows reading role assignments across your Azure resources

**Important Notes:**
- ✅ With **Option A (User-Delegated)**, the tool uses YOUR existing permissions - no admin approval needed
- ✅ With **Option B (Pre-Approved)**, an admin grants permissions once for all users
- ⚠️ Regular users without directory read roles won't be able to see Entra ID permissions (but can still see their Azure RBAC access)

### To Execute Cleanup Scripts

To actually remove permissions and delete users, you need:

- **Azure RBAC**: `User Access Administrator` or `Owner` role (to remove role assignments)
- **Entra ID**: `User Administrator` or `Global Administrator` role (to delete users)

## Security Considerations

### Authentication

- Uses your own Azure AD application registration (you control the client ID)
- All authentication happens through Microsoft's official MSAL.js library (MSAL.js 2.38.1)
- Uses Authorization Code Flow with PKCE (Proof Key for Code Exchange) for enhanced security
- Tokens are stored in session storage and cleared when you close the browser
- Configuration (Client ID and Tenant ID) is stored in browser local storage
- The tool runs entirely in your browser - no data is sent to third-party servers
- All API calls go directly to Microsoft Graph and Azure Resource Manager APIs

### Permissions

- The tool only requests read permissions
- It cannot make any changes to your Azure environment
- All modifications happen through the scripts you generate and execute manually
- Review generated scripts carefully before executing

### Best Practices

1. **Test in a non-production environment first**
2. **Review the generated script** before executing
3. **Backup important data** before deleting users
4. **Verify user selections** are correct
5. **Consider disabling users first** instead of immediate deletion
6. **Document why each user is being removed**

## Troubleshooting

### "AADSTS65002: Consent between first party application..." error

This error occurs when trying to use a first-party Microsoft client ID (like the Azure CLI client ID). **Solution**: You must register your own Azure AD application following the Quick Start guide above.

### "Failed to acquire tokens" or "Please configure your Azure AD application first"

- Ensure you have registered an Azure AD application (see Quick Start section)
- Click "⚙️ Configure App" and enter your Client ID and Tenant ID
- Verify the redirect URI in your Azure AD app registration matches the URL you're using
- Clear browser cache and local storage, then try again
- Check that your account has the required permissions

### "Failed to fetch subscriptions"

- Verify you have at least Reader access to Azure subscriptions
- Check that your subscriptions are active

### "Failed to fetch Entra ID roles"

- Ensure you have Directory.Read.All or equivalent permissions
- Some roles may require Global Reader or higher

### Script execution fails

- Verify you have the required permissions to remove role assignments
- For Entra ID user deletion, you need User Administrator or Global Administrator
- Check that the Az PowerShell module or Azure CLI is installed and up to date

## Limitations

- **PIM Eligible Roles**: The tool can detect PIM eligible roles but the generated scripts cannot remove them automatically. These must be removed through the Azure Portal or Microsoft Graph API.
- **Group Memberships**: The tool does not show permissions granted through group membership. Users may still have access through security groups or Azure AD groups.
- **Application Permissions**: Service principals and managed identities are not included.
- **Custom Roles**: Custom role definitions are shown as "Custom Role" without detailed permissions.

## Technical Details

### APIs Used

- **Microsoft Graph API v1.0**:
  - `/users` - List all users
  - `/directoryRoles` - List directory role assignments
  - `/roleManagement/directory/roleEligibilityScheduleInstances` - List PIM eligible roles

- **Azure Resource Manager API**:
  - `/subscriptions` - List subscriptions
  - `/providers/Microsoft.Authorization/roleAssignments` - List RBAC role assignments

### Browser Compatibility

- Chrome 90+
- Edge 90+
- Firefox 88+
- Safari 14+

Requires a modern browser with support for:
- ES6 JavaScript
- Fetch API
- Async/await
- CSS Grid/Flexbox

## Example Output

The tool generates scripts like this:

### PowerShell Example

```powershell
# Azure Permissions Cleanup Script (PowerShell)
# Generated: 2026-01-11T12:00:00.000Z
# Selected users: 2

# ===== John Doe (john.doe@example.com) =====
# Remove Azure RBAC assignments (3 assignments)
Remove-AzRoleAssignment -ObjectId "user-id-1" -Scope "/subscriptions/sub-id/resourceGroups/rg1" -RoleDefinitionId "/subscriptions/sub-id/providers/Microsoft.Authorization/roleDefinitions/role-id"
# Remove Entra ID roles (1 role)
Remove-AzureADDirectoryRoleMember -ObjectId "role-id" -MemberId "user-id-1"
```

### Azure CLI Example

```bash
#!/bin/bash
# Azure Permissions Cleanup Script (Azure CLI)
# Generated: 2026-01-11T12:00:00.000Z
# Selected users: 2

# ===== John Doe (john.doe@example.com) =====
# Remove Azure RBAC assignments (3 assignments)
az role assignment delete --ids "/subscriptions/sub-id/providers/Microsoft.Authorization/roleAssignments/assignment-id"
# Remove Entra ID roles (1 role)
az rest --method DELETE --url "https://graph.microsoft.com/v1.0/directoryRoles/role-id/members/user-id/$ref"
```

## Contributing

This is a self-contained tool. To modify:

1. Open `azure-permissions-viewer.html` in a text editor
2. Make your changes to the HTML, CSS, or JavaScript
3. Test thoroughly before use
4. Consider the security implications of any changes

## License

This tool is provided as-is for use in managing Azure environments. Use at your own risk.

## Support

For issues or questions:
1. Review the Troubleshooting section above
2. Check Azure documentation for API-specific issues
3. Verify your permissions in the Azure Portal

---

**WARNING**: This tool generates scripts that can permanently delete users and remove access. Always review generated scripts carefully and test in a non-production environment first.
