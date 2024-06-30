---
layout: post
title: 'Leader Election using etcd from C#'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Having a service registry can be very useful when you have multiple instances of the same application running. Let's take a look at how to elect one of the instances to be the leader.

<!--more-->

### The Problem

In a previous post I explored how to use `etcd` as a service registry for .NET applications. We created a service registry class that allowed each instance of a .NET application to register itself and to have it automatically be removed fro the registry if it didn't let the know it was still alike periodically.

But what if we have a special task that we don't want all of them to be performing? An example is polling a database for new work to queue up. Let's say we have a legacy application that writes events into a database table. We'd prefer to process those events using a queue (like RabbitMQ, or Azure Service Bus). We have multiple instances of a worker application that processes the events in the queue. But we only want one of them responsible for polling the database for new events to add to the queue. For that to work we need to elect one of the instances to be the the one responsible for this task.

### Electing a Leader

Electing a leader could be done via a configuration setting. Let's say you have 2 instances, each running on a separate server (or in a container). You could deploy them with settings that says instance 1 should be the leader, and it will poll the database for new events to queue. But if this instance of the application goes offline, then the polling stops and new new work is being queued. You would have to update the settings for instance 2 to now designate it as the leader. A better solution would be to have the instances determine who is the leader automatically.

Many service registry have the ability to elect a leader. Etcd uses leases to perform this function. However, a simple mechanism would simply to query the registry for all active instances for a service. Whichever instances was registered first is the leader. Each of the services will check periodically if it is the one registered first. If so, it is now considered the leader. I have used systems like this and they work well. But I'd like to explore the lease features in etcd and see how we can use them to elect a leader.

### Using Etcd Leases

Each lease in etcd is identified using a key. Let's create a key that represents the leader of a service: `/leaders/{ServiceName}`

Each of the service instances will attempt to create a lease using this key. The first one to successfully create this key is the leader. All the others will fail and will need to try again in a few seconds. The instance that created the lease will keep it alive by sending heartbeats to etcd. If those heartbeats ever stop the lease will be deleted. Once deleted, the instances will all try to re-elect a leader.

### Transaction Request

To create the leader key, we will use a transaction that will compare if the key in etcd has a version equal zero. If the version is zero, that means the key doesn't exists. The transaction will then either run a success operation or failure operation. If the version is zero, then the transaction will send a PUT request for the service instance using the same lease that it used to register itself with etcd (which keeps the key alive). Otherwise, on failure, it will return the value of the current version of the key. This value is the instance that is currently the leader.

The transaction is an atomic "If/Then/Else" construct. The request consists of 3 parts:
- Compare: This is the "IF" clause. It returns a true/false result.
- Success: This operation is executed if the "Compare" operation returns true.
- Failure: This operation is executed if the "Compare" operation returns false.

Here is what the transaction request looks like for our situation:

```csharp
var txnRequest = new TxnRequest
{
    Compare =
    {
        // Does The Key Exist
    },
    Success =
    {
        // Make This Instance The Leader
    },
    Failure =
    {
        // Get The Elected Leader
    }
};
```

Using the transaction request means that any other request to perform the same operation will have to wait until this one is completed.

The Compare operation needs to check if the key exists:

```csharp
Compare = new Compare
        {
            Key = ByteString.CopyFromUtf8(key),
            Result = Compare.Types.CompareResult.Equal,
            Target = Compare.Types.CompareTarget.Version,
            Version = 0 // version is 0 if key does not exist
        };
```

The Success operation will send a PUT operation which adds the key to the etcd key/value store:

```csharp
Success = new RequestOp
        {
            RequestPut = new PutRequest
            {
                Key = ByteString.CopyFromUtf8(key),
                Value = ByteString.CopyFromUtf8(instanceId),
                Lease = leaseId
            }
        }
```

The Failure operation returns the value of the current key:

```csharp
Failure = new RequestOp
        {
            RequestRange = new RangeRequest
            {
                Key = ByteString.CopyFromUtf8(key),
            }
        };
```

In our application we can now create an "IsLeader" method. The code will send the transaction request to etcd which will return success or failure. If successful, then the current instance has just become the leader. If it fails, we need to check which instance is currently stored as the leader. If the instance stored as the leader is the instance being run then it is considered the leader. 

```csharp
var isLeader = false;

var txnRequest = new TransactionRequest{ ....... };

var txnResponse = await _client.TransactionAsync(txnRequest, null, null, token);
if (txnResponse.Succeeded)
{
    isLeader = true;
}
else
{
    var responses = txnResponse.Responses.FirstOrDefault();
    var keyValues = responses?.ResponseRange.Kvs.FirstOrDefault();
    var electedLeaderInstanceId = keyValues?.Value?.ToStringUtf8();

    isLeader = instanceId.Equals(electedLeaderInstanceId, StringComparison.OrdinalIgnoreCase);
}
```

### Summary

Etcd is a powerful tool when orchestrating distributed services. I've demonstrated how to use it to act as a service registry and to use it to elect a leader among multiple instances of the same service. This basic infrastructure provides an excellent foundation for operating multiple instances of the same service.
