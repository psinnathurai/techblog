+++
title = 'Certificate-Based Onboarding for Azure Arc Servers'
date = 2024-06-06T16:14:21+02:00
draft = false
+++


# Overview
Azure Arc version 1.41 introduces certificate-based authentication for connecting and disconnecting servers, replacing the old method of using passwords. This new feature makes managing servers easier and more secure.

In this article, I will explain how to set up and use certificates for Azure Arc-enabled servers. You will learn how to create a certificate using Active Directory Certificate Services, export the certificate, and use it for onboarding servers to Azure Arc. Additionally, I will cover common issues you might face and suggest ways to improve the process.

By the end of this guide, you will be able to use certificates to securely manage and onboard your servers to Azure Arc.

![Meme](/images/blog/20240606/meme.jpg#center)

# Why Use Certificates Instead of Secrets?
Previously, secrets were used for onboarding Azure Arc-enabled servers. However, certificates offer several advantages:
- **Simplified Management:** No need to remember or manage complex passwords.
- **Centralized Control:** Certificates can be centrally managed, and revocation can be used to disable them.
- **Enhanced Security:** Certificates support a zero-trust architecture by requiring verification at each step.

# Create a Certificate
## Active Directory Certificate Services
To create a certificate for onboarding, we will use an internal Active Directory Certificate Services (AD CS) infrastructure. AD CS is a Windows Server role for issuing and managing a public key infrastructure (PKI), which creates, manages, distributes, stores, and revokes digital certificates.

For more information, visit: [Active Directory Certificate Services Overview](https://learn.microsoft.com/en-us/windows-server/identity/ad-cs/active-directory-certificate-services-overview)

## Create a Certificate Template
With your Certificate Authority ready, open the Certification Authority Console.

1. **Manage Certificate Templates:**
   - Open up the Certificate Authority Management Console.
   - Click on "Certificates Templates" and then "Manage".

   ![](/images/blog/20240606/000004.jpg#center)

   ![](/images/blog/20240606/000005.jpg#center)

2. **Duplicate the Computer Template:**
   - Find the existing Computer Template, right-click on it, and select "Duplicate Template".

   ![](/images/blog/20240606/000006.jpg#center)

3. **Configure the New Template:**
   - Choose the highest available Certificate Authority.
   - Under Certificate Recipient, ensure it is set for the lowest Member of your Domain. If you still have Windows Server 2012 R2 in your Environment you need to choose Windows Server 2012 R2 and shame on you!

   ![](/images/blog/20240606/000007.jpg#center)

4. **Set General Properties:**
   - Name your template and publish the certificate in Active Directory.

   ![](/images/blog/20240606/000008.jpg#center)

5. **Request Handling:**
   - Select "Allow private key to be exported".

   ![](/images/blog/20240606/000009.jpg#center)

6. **Cryptography Settings:**
   - Choose Key Storage Provider as the Provider Category.
   - Set the Request Hash to SHA512.
   - Click "Apply".

   ![](/images/blog/20240606/000010.jpg#center)

7. **Issue the Certificate Template:**
   - In the Certificate Authority MMC, right-click on Certificate Template, select "New", then "Certificate Template to Issue".

   ![](/images/blog/20240606/000011.jpg#center)

   - Select your newly created Certificate Template and click "OK".

   ![](/images/blog/20240606/000013.jpg#center)

Now we have created a Certificate Template in our Active Directory Certificate Authority which we can use for creating a Certificate.

## Create a Certificate Using the Template
Use a server or client that is a member of your Active Directory domain.

1. **Open Certificate Console:**
   - Run `certlm.msc` to open the Certificate Console in Computer Context.

   ```bash
   certlm.msc
   ```
2. **Request a New Certificate:**
   - Navigate to Personal > Certificates > All Tasks > Request New Certificate.

   ![](/images/blog/20240606/000012.jpg#center)

3. **Certificate Enrollment:**
   - Click "Next".
   - Select "Active Directory Enrollment Policy" and click "Next".
   - Choose the template you created and click "Enroll".

   ![](/images/blog/20240606/000014.jpg#center)

   ![](/images/blog/20240606/000015.jpg#center)

   ![](/images/blog/20240606/000016.jpg#center)

## Exporting the Certificate
Return to the Personal Certificate Store of your computer.

1. **Identify and Export the Certificate:**
   - Right-click on the certificate, select "All Tasks", then "Export".

   ![](/images/blog/20240606/000017.jpg#center)

    ![](/images/blog/20240606/000018.jpg#center)

2. **Export Options:**
   - Click "Next".
   - Select "Yes, export the private key" and click "Next".
   - Choose "Personal Information Exchange - PKCS #12 (.PFX)" and click "Next".

    ![](/images/blog/20240606/000019.jpg#center)

    ![](/images/blog/20240606/000020.jpg#center)

    ![](/images/blog/20240606/000021.jpg#center)

3. **User Selection:**
   - Enter a Password of your Choice and select AES256-SHA256 as your Encryption and click "Next".

   ![](/images/blog/20240606/000022_a.jpg#center)

4. **Complete the Export:**
   - Choose a file name and click "Next".
   - Click "Finish".
   
    ![](/images/blog/20240606/000023.jpg#center)

    ![](/images/blog/20240606/000024.jpg#center)

5. **Repeat Without Exporting the Private Key:**
   - Perform the export again, but select "No, do not export the private key".

        ![](/images/blog/20240606/000025.jpg#center)


Now you should have both a .CER and a .PFX file.

![](/images/blog/20240606/000026.jpg#center)


## Service Principal for Onboarding
In this step, we will create a Service Principal for onboarding Azure Arc-enabled servers. A Service Principal is used for authentication during this process.

1. **Switch to the Azure Portal.**

2. **Navigate to Azure Arc:**
   - Click on "Service Principals" and then click "Add".

   ![](/images/blog/20240606/000027.jpg#center)

3. **Define the Service Principal:**
   - Enter a name for your Azure Arc Service Principal.
   - Select the scope for the Service Principal.

   ![](/images/blog/20240606/000028.jpg#center)

4. **Set Client Secret Details:**
   - Choose a description and an expiration date for the client secret (note: we will delete this later).
   - Select the role "Azure Connected Machine Onboarding".
   - Click "Create".

   ![](/images/blog/20240606/000029.jpg#center)

5. **Manage the Service Principal:**
   - Go to the created Service Principal, click on "Certificates & secrets," then click on "Client secrets."

   ![](/images/blog/20240606/000030.jpg#center)

   - Click on the trash icon to delete the client secret.

   ![](/images/blog/20240606/000031.jpg#center)

6. **Upload Certificate:**
   - Click on "Certificates" and then click "Upload Certificate."
   - Upload the certificate and click "Add."

   ![](/images/blog/20240606/000032.jpg#center)

   ![](/images/blog/20240606/000033.jpg#center)

## Onboarding the Server
Finally, we proceed to onboard the server But I failed 

1. **Convert the Certificate from .PFX to .PEM:**
   - Convert the .PFX File to a .PEM File. You can use openSSL for the conversion.

> **_INFO:_**  On a Windows Client you can use openSSL if you have Git installed on your Client.

2. **Copy the Certificate:**
   - Transfer the .PEM file to the Windows server you wish to onboard to Azure Arc.

    ```Bash
    cd 'C:\Program Files\Git\usr\bin\'
    .\openssl.exe pkcs12 -in "C:\Users\PratheepSinnathurai\Downloads\test.pfx" -out "C:\Users\PratheepSinnathurai\Downloads\test.pem" -nodes

    ```
3. **Install the Azure Arc Agent:**
   - Run the following PowerShell command to install the Azure Arc Agent on the Windows server:

   ```PowerShell
    # Download the installation package
    Invoke-WebRequest -UseBasicParsing -Uri "https://aka.ms/azcmagent-windows" -TimeoutSec 30 -OutFile "$env:TEMP\install_windows_azcmagent.ps1";

    # Install the hybrid agent
    & "$env:TEMP\install_windows_azcmagent.ps1";
    if ($LASTEXITCODE -ne 0) { exit 1; }

      ```

      ```PowerShell
   # Define parameters
   $servicePrincipalCertPath = "C:\temp\test.pfx"
   $servicePrincipalId = "befa049d-f87d-4362-95d2-a03728c80959"
   $tenantId = "5e6d7959-d83c-418e-bc9a-c1766178f93d"
   $location = "switzerlandnorth"
   $resourceGroup = "rg-azurearc-arclz-prd-szn-01"
   $subscriptionId = "58383638-826b-42fb-bc5b-e07f4ef489e5"
   $tags = "UpdateGroup=A1"

   # Construct the command
   $azcmagentConnectCmd = "azcmagent connect --service-principal-cert `"$servicePrincipalCertPath`" --service-principal-id $servicePrincipalId --tenant-id $tenantId --location $location --resource-group $resourceGroup --subscription-id $subscriptionId --tags $tags"

   # Execute the command
   Invoke-Expression $azcmagentConnectCmd

   ```

4. **Confirming**

At the end we see that the server was successfully onboarded.

![](/images/blog/20240606/000040.jpg#center)


## Troubleshooting
When I was testing this feature, I encountered some failures that I'd like to show you.

### .PFX File

In my first test I used the .PFX File instead of the .PEM File. 

In the documentation, it is mentioned that password-protected .PFX files were not working.

![Image 1](/images/blog/20240606/000035.jpg#center)

Therefore, I tested out the .PFX file with the permission "Everyone" (don't do that in production).

![Image 2](/images/blog/20240606/000037.jpg#center)

I received the following error message:

![Error Message](/images/blog/20240606/000036.jpg#center)


## Path to the .PFX File

Let's say we have our Certificate ready and I like to deploy that Certificate to multiple Servers.
I would use a Group Policy to deploy my Certificate into the Personal Certificate Store and would run the azcmagent connect Command refering to my Personal Certificate Store.

In my opininon we should be able to reference to the Certificates which are in the Certiciate Personal Store.

![Error Message](/images/blog/20240606/000041.jpg#center)


# Improvements
After testing certificate-based authentication for Azure Arc-enabled server onboarding, I recommend the following improvements for Microsoft:

1. It would be great if we could have more insights on how to create and manage the Certificate for Onboarding.
2. Enable linking certificates from the Certificate Store rather than relying on file storage. This would make the deployment of certificates across all Windows Servers for Azure Arc onboarding easier.
3. Simplify the creation of Azure Arc Service Principals in the Azure Portal by enabling direct use of a certificate, eliminating the current requirement to provide a client secret.

# Conclusion
In conclusion, Azure Arc version 1.41 brings a valuable new feature: certificate-based authentication for onboarding servers. This change makes the process more secure by using certificates instead of passwords, making it easier to manage and control access centrally.

This article provided a step-by-step guide on how to create and use certificates for Azure Arc. We covered setting up Active Directory Certificate Services, creating a certificate template, exporting the certificates, and creating a service principal for Azure Arc. We also discussed common problems and improvements needed to make the process smoother.

Overall, using certificates instead of passwords for Azure Arc onboarding is a big improvement. It simplifies the process and enhances security. With continued updates and user feedback, Microsoft can make Azure Arc even easier and more secure to use for managing servers.