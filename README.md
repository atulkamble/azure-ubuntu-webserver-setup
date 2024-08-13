# Deploy a Full Website on Azure Ubuntu VM 
# Clone repo
```
git clone https://github.com/atulkamble/azure-ubuntu-webserver-setup.git
cd azure-ubuntu-webserver-setup
```

Detailed step-by-step guide to creating an Azure Linux VM with Ubuntu, installing a web server, and deploying a full website using Azure CLI and shell scripting.

### Clone this Project
```
git clone https://github.com/atulkamble/AzureWebsiteProject.git
cd AzureWebsiteProject
```

### Prerequisites

1. **Install Azure CLI**: If you haven't installed the Azure CLI, follow the instructions [here](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).
2. **Login to Azure**: Open your terminal and run:
   ```sh
   az login
   ```

### Step 1: Create Resource Group

First, create a resource group to hold your resources:

```sh
RESOURCE_GROUP="MyResourceGroup"
LOCATION="eastus"

az group create --name $RESOURCE_GROUP --location $LOCATION
```

### Step 2: Create a Virtual Machine

1. **Create VM:**
   ```sh
   VM_NAME="MyUbuntuVM"
   IMAGE="UbuntuLTS"
   ADMIN_USERNAME="azureuser"
   SSH_KEY_PATH="$HOME/.ssh/id_rsa.pub"

   az vm create \
     --resource-group $RESOURCE_GROUP \
     --name $VM_NAME \
     --image $IMAGE \
     --admin-username $ADMIN_USERNAME \
     --ssh-key-value $SSH_KEY_PATH \
     --output json \
     --verbose
   ```

2. **Open Port 80 for Web Traffic:**
   ```sh
   az vm open-port --resource-group $RESOURCE_GROUP --name $VM_NAME --port 80
   ```

### Step 3: Install Apache Web Server and Deploy Website

1. **Get Public IP Address:**
   ```sh
   PUBLIC_IP=$(az vm show --show-details --resource-group $RESOURCE_GROUP --name $VM_NAME --query publicIps -o tsv)
   ```

2. **Create Website Files:**

   Create a directory for your website files:

   ```sh
   mkdir mywebsite
   cd mywebsite
   ```

   Create an `index.html` file:

   ```html
   <!DOCTYPE html>
   <html>
   <head>
       <title>My Azure VM Website</title>
   </head>
   <body>
       <h1>Hello from Azure VM!</h1>
       <p>Welcome to my website hosted on an Azure VM with Ubuntu and Apache.</p>
   </body>
   </html>
   ```

3. **Deploy Website to VM:**

   Upload your website files to the VM:

   ```sh
   scp -o "StrictHostKeyChecking no" -r * $ADMIN_USERNAME@$PUBLIC_IP:/tmp/
   ```

4. **Install Apache and Move Website Files:**

   Connect to the VM and install Apache, then move the website files to the Apache directory:

   ```sh
   ssh -o "StrictHostKeyChecking no" $ADMIN_USERNAME@$PUBLIC_IP <<EOF
   sudo apt update
   sudo apt install apache2 -y
   sudo systemctl start apache2
   sudo systemctl enable apache2
   sudo mv /tmp/* /var/www/html/
   EOF
   ```

### Step 4: Verify the Website

1. **Open a Web Browser:**
   - Navigate to the public IP address of your VM: `http://<PUBLIC_IP>`
   - You should see the website content you created in `index.html`.

### Step 5: Secure Your Website with HTTPS (Optional)

1. **Install Certbot and Obtain SSL Certificate:**
   ```sh
   ssh $ADMIN_USERNAME@$PUBLIC_IP <<EOF
   sudo apt install certbot python3-certbot-apache -y
   sudo certbot --apache --non-interactive --agree-tos --email your-email@example.com -d your-domain.com
   EOF
   ```

2. **Update DNS Records:**
   - Ensure your domain's DNS records point to your VM's public IP address.

### Complete Script

Here’s a complete script that automates the above steps:

```sh
#!/bin/bash

# Variables
RESOURCE_GROUP="MyResourceGroup"
LOCATION="eastus"
VM_NAME="MyUbuntuVM"
IMAGE="UbuntuLTS"
ADMIN_USERNAME="azureuser"
SSH_KEY_PATH="$HOME/.ssh/id_rsa.pub"
DOMAIN="your-domain.com"
EMAIL="your-email@example.com"

# Create Resource Group
az group create --name $RESOURCE_GROUP --location $LOCATION

# Create VM
az vm create \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --image $IMAGE \
  --admin-username $ADMIN_USERNAME \
  --ssh-key-value $SSH_KEY_PATH \
  --output json \
  --verbose

# Open Port 80
az vm open-port --resource-group $RESOURCE_GROUP --name $VM_NAME --port 80

# Get Public IP
PUBLIC_IP=$(az vm show --show-details --resource-group $RESOURCE_GROUP --name $VM_NAME --query publicIps -o tsv)

# Create website files
mkdir mywebsite
cd mywebsite
echo '<!DOCTYPE html>
<html>
<head>
    <title>My Azure VM Website</title>
</head>
<body>
    <h1>Hello from Azure VM!</h1>
    <p>Welcome to my website hosted on an Azure VM with Ubuntu and Apache.</p>
</body>
</html>' > index.html

# Deploy website to VM
scp -o "StrictHostKeyChecking no" -r * $ADMIN_USERNAME@$PUBLIC_IP:/tmp/

# Install Apache and move website files
ssh -o "StrictHostKeyChecking no" $ADMIN_USERNAME@$PUBLIC_IP <<EOF
sudo apt update
sudo apt install apache2 -y
sudo systemctl start apache2
sudo systemctl enable apache2
sudo mv /tmp/* /var/www/html/
EOF

# Optionally secure the website with HTTPS
ssh -o "StrictHostKeyChecking no" $ADMIN_USERNAME@$PUBLIC_IP <<EOF
sudo apt install certbot python3-certbot-apache -y
sudo certbot --apache --non-interactive --agree-tos --email $EMAIL -d $DOMAIN
EOF

echo "Website is available at http://$PUBLIC_IP"
```

### Running the Script

1. **Save the script to a file, e.g., `create-azure-vm-with-website.sh`.**
2. **Make the script executable:**
   ```sh
   chmod +x create-azure-vm-with-website.sh
   ```
3. **Run the script:**
   ```sh
   ./create-azure-vm-with-website.sh
   ```

By following these steps and running the provided script, you will have an Ubuntu server running on an Azure VM with Apache installed, hosting a complete website. You can further customize the script and your website as needed.
