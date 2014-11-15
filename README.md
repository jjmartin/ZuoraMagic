ZuoraMagic
==========
This library is intended to be a simple, solid, and small library that encompasses and provides access to the SOAP, REST and Bulk API's for Zuora. The magic lies in the library's ability to dynamically generate ZOQL queries based on generic types and also the dynamic generation of the SOAP wsdl. You will never have to replace or update a wsdl again.

### Installation ###
[Nuget](https://www.nuget.org/packages/SalesforceMagic) => PM> Install-Package SalesforceMagic

### Currently Implemented Features ###
* Dynamic ZOQL query generation based on generic type interpretation
* Expression and predicate visitation allowing users to create both simple and complicated where conditions using powerful lambda expressions.
* SOAP Login and Session ID Retrieval
* SOAP Query based on generic types.
* Re-implementable storage for reusing session details.

### Example Usage ###

Start by setting up the configuration and Zuora client:

```csharp
public static Contact Find(string firstName, string lastName)
{
    var config = new ZuoraConfig
    {
        Username = "zuoraUsername",
        Password = "zuoraPassword",
        IsSandbox = true,
		InstanceUrl = "https://apisandbox.zuora.com/apps/services/a/62.1"
    }

 
	using (var zuoraClient = new ZuoraClient(config, true))
	{
		//... Zuora object Logic
	}

}
```

Let's grab set up a contact Object and do a search for it.

We can use the session id if we want, but the method automatically uses the last session id retrieved as long as it is not stale. For example, let's grab 5 vAttachments. We'll start by creating a C# class to represent out vAttachment. SalesforceMagic provides custom attributes that allow you to have pretty and simple classes while mapping to custom field names in Salesforce:

```csharp
	[ZuoraName("Contact")]
    public class Contact : ZObject
    {
        [ZuoraName("FirstName")]
        public string FirstName { get; set; }

        [ZuoraName("LastName")]
        public string LastName { get; set; }

        [ZuoraName("WorkEmail")]
        public string WorkEmailAddress { get; set; }
	}
```

Now let's actually perform the query, we'll find a contact based on firstName and lastName

```csharp
returnContact = zuoraClient.QuerySingle<Contact>(contact => contact.FirstName == firstName && contact.LastName == lastName);
```

Let's also go over the use of CRUD operations using both the SOAP and Bulk apis.
First let's create a list of objects we can use.

```csharp
vAttachment[] attachments = new []
{
    new vAttachment
    {
        FileName = "Test.pdf",
        S3Id = "123456789"
    },
    new vAttachment
    {
        FileName = "Test-2.pdf",
        S3Id = "abcdefghij"
    },
    ...
}
```

SOAP API: Using the SOAP api for CRUD operations is very simple. It can easily deal with individual sObjects or an array.

```csharp
SalesforceResponse response = client.PerformCrudOperation(attachments, CrudOperations.Insert);
```

BULK API: Interaction with the bulk api is slightly different. In order to use the bulk api you'll need to start a data load job:

```csharp
JobInfo jobInfo = client.CreateBulkJob<vAttachment>(new JobConfig
{
    ConcurrencyMode = ConcurrencyMode.Parallel,
    Operation = BulkOperations.Insert
});
```

Starting a job will return a JobInfo object:

```csharp
public class JobInfo
{
    public string Id { get; set; }
    public string Operation { get; set; }
    public string Object { get; set; }
    public string CreatedById { get; set; }
    public DateTime CreatedDate { get; set; }
    public JobState State { get; set; }
    public ConcurrencyMode ConcurrencyMode { get; set; }
    public string ContentType { get; set; }
    public int NumberBatchedQueued { get; set; }
    public int NumberBatchedInProgress { get; set; }
    public int NumberBatchedCompleted { get; set; }
    public int NumberBatchedFailed { get; set; }
    public int NumberBatchedTotal { get; set; }
    public int NumberBatchedProcessed { get; set; }
    public int NumberRetries { get; set; }
    public string ApiVersion { get; set; }
}