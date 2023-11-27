# Azure SDK Sample6 Create FilesShare

## 1. Libraries reference

```csharp
using System;
using System.Threading.Tasks;
using Azure.Identity;
using Azure.ResourceManager.Storage.Models;
using Azure.ResourceManager.Resources;
using NUnit.Framework;
using Azure.Core;
```

## 2. Create ARM (Azure Resources Management) client.

When you first create your ARM client, choose the subscription you're going to work in. 

There's a convenient DefaultSubscription property that returns the default subscription configured for your user:

```csharp
ArmClient armClient = new ArmClient(new DefaultAzureCredential());
SubscriptionResource subscription = await armClient.GetDefaultSubscriptionAsync();
```

This is a scoped operations object, and any operations you perform will be done under that subscription. 

From this object, you have access to all children via collection objects. Or you can access individual children by ID.

## 3. Create or update the Azure ResourceGroup 

```csharp
string rgName = "myRgName";
AzureLocation location = AzureLocation.WestUS2;
ArmOperation<ResourceGroupResource> operation = await subscription.GetResourceGroups().CreateOrUpdateAsync(WaitUntil.Completed, rgName, new ResourceGroupData(location));
ResourceGroupResource resourceGroup = operation.Value;
```

## 4. Create an Azure Storage Account

```csharp
//first we need to define the StorageAccountCreateParameters
StorageSku sku = new StorageSku(StorageSkuName.StandardGrs);
StorageKind kind = StorageKind.Storage;
string location = "westus2";
StorageAccountCreateOrUpdateContent parameters = new StorageAccountCreateOrUpdateContent(sku, kind, location);
//now we can create a storage account with defined account name and parameters
StorageAccountCollection accountCollection = resourceGroup.GetStorageAccounts();
string accountName = "myAccount";
ArmOperation<StorageAccountResource> accountCreateOperation = await accountCollection.CreateOrUpdateAsync(WaitUntil.Completed, accountName, parameters);
StorageAccountResource storageAccount = accountCreateOperation.Value;
```

## 5. Create the Azure Fileshare 

Then we need to get the **file service**, which is a singleton resource and the name is "default"

```csharp
FileServiceResource fileService = await storageAccount.GetFileService().GetAsync();
```

Now that we have the file service, we can manage the file shares inside this storage account.

Create a file share

```csharp
FileShareCollection fileShareCollection = fileService.GetFileShares();
string fileShareName = "myFileShare";
FileShareData fileShareData = new FileShareData();
ArmOperation<FileShareResource> fileShareCreateOperation = await fileShareCollection.CreateOrUpdateAsync(WaitUntil.Started, fileShareName, fileShareData);
FileShareResource fileShare =await fileShareCreateOperation.WaitForCompletionAsync();
```

## 6. List all file shares

```csharp
FileShareCollection fileShareCollection = fileService.GetFileShares();
AsyncPageable<FileShareResource> response = fileShareCollection.GetAllAsync();
await foreach (FileShareResource fileShare in response)
{
    Console.WriteLine(fileShare.Id.Name);
}
```

## 7. Get a file share

```csharp
FileShareCollection fileShareCollection = fileService.GetFileShares();
FileShareResource fileShare = await fileShareCollection.GetAsync("myFileShare");
Console.WriteLine(fileShare.Id.Name);
```

## 8. Delete a file share

```csharp
FileShareCollection fileShareCollection = fileService.GetFileShares();
FileShareResource fileShare = await fileShareCollection.GetAsync("myFileShare");
await fileShare.DeleteAsync(WaitUntil.Completed);
```
