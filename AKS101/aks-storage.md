# AKS Storage

## Why do we need storage?

So far, we have been handling AKS without a storage component, and everything has been fine. So why do we need storage for AKS at all? Well, it is the same reason why Kubernetes or Docker storage exists. Based on your needs and the application you are running, you may need persistent storage. While memory storage is fast, it is not reliable, and the data stored within memory cannot be shared among pods. Additionally, if you were to run something like a database, then you would obviously need advanced storage capabilities, and in-memory storage would not be a feasible solution.

## Storage options

**Azure Volume Storage**: This is perhaps the most popular type of storage when it comes to Azure, regardless of whether you are using it with AKS, or with some other Azure service (such as ACI). Volumes storage is secure, highly available, scalable, and easy to integrate set up, and integrate with other Azure services such as AKS. You can read about this type of storage in-depth in the [official docs](https://docs.microsoft.com/en-us/azure/storage/common/storage-introduction).

**Azure Disks**: Azure disks are basically like disks you would get in your on-premise servers, but virtualized. Meaning that, unlike on-prem disks, Azure will manage them for you. You can change their size with a simple request, scale up and down, have access control, create disk backups, or even upload your own vhd to automate the setup process. You also get fine-grained access control and security, along with a heap of options that can all be studied in detail in the [official docs](https://docs.microsoft.com/en-us/azure/virtual-machines/managed-disks-overview)

**Azure Files**: Unlike the previous option, you don't get a disk here. Instead, what you get is a file share. This is obviously more geared towards containerization and can be helpful when used with AKS since it is significantly easier to use. You can also use it to extend existing disks (may they be on-prem or on an Azure disk). Since file shares are meant to be shared, it allows for better collaboration when creating the system, and is fully managed by Azure. More about this can be found in the [official docs](https://docs.microsoft.com/en-us/azure/storage/files/storage-files-introduction)

**Azure NetApp Files**: We are now heading on to more obscure types of storage that can be used with AKS. Azure NetApp files are not meant to be used by individual users or small teams, but rather, by huge organizations with demanding AKS clusters. As such, NetApp files are high-performance and cost a fair bit. If you want to learn more, the [official docs](https://docs.microsoft.com/en-us/azure/azure-netapp-files/azure-netapp-files-introduction) provide an introduction with a step-by-step guide.

**Azure Blob Storage**: Finally, we have Azure Blob Storage. This is somewhat popular, and you may have heard of it in a big data context. This is because Azure Blob Storage is specially designed to handle large amounts of unstructured data. Think in terms of data lakes that host billions of records. Accessing this blob storage can be done via the variety of clients provided by Azure. If you have an AKS cluster that is focused on data science, then Azure Blob Storage is the perfect solution for you. Read more about it in the [official docs](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blobs-overview)


## Other types of storage 

**Persistent Volumes (PV):** You may already know of persistent volumes and volume claims. These ensure that data isn't lost when pods disappear, and naturally, they are fully compatible with AKS. What's best is that PV's work well with already existing Azure services, and can dynamically resource Azure disks or File storage to provide persistent storage to a pod if the already allocated resources are insufficient. 

**Storage classes:**  Since Azure provides multiple tiers when it comes to storage space, it is important to select one that suits the application and the budget.  AKS clusters deal with tiers via Container Storage Interface (CSI) drivers. By using CSI's, AKS can perform all sorts of operations of existing data without having to modify the core Kubernetes code. this means a layer of abstraction is introduced which helps reduce dependencies on other code bases. More about CSI's are covered in the [official documentation](https://docs.microsoft.com/en-us/azure/aks/csi-storage-drivers).

**Persistent Volume Claims (PVC's):** PVC's are covered in detail in the [StatefulSets101](./../StatefulSets101/README.md) section of this course. As with PV's PVCs can be applied across AKS in the same way you would an ordinary Kubernetes cluster.

Next, let's talk about service meshes in AKS

[Next: Service Meshes](./aks-service-mesh.md)