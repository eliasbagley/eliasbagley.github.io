---
layout: post
title:  "Syncing a web API with UITableView using Core Data NSFetchedResultsController in Swift"
date:   2015-08-29 23:22:15
categories: JSON API CoreData UITableView NSFetchedResultsController
---

In the [last article][part1], we finished with a small app that could make a request to a web server, parse the `JSON` response into `Article` objects, and use the `Article` objects to populate a `UITableView`.

In order to do this, we needed to keep track of a local reference to an `Array` of `Article` objects, which creates synchronization and data binding issues down the road. In this article we will talk about getting `Core Data` set up, and synchronizing the web servers response with the data store.

Here's our outline:

1) Set up the `Core Data` stack
2) Create a `Core Data` data model
3) Add the syncing code to the `Alamofire` response


1)

There is tons of information on the Internet about setting up a `Core Data` stack, so I won't go into too much detail here. See the accompanying example code for the details. The easiest way to set up a stack is to use a [3rd party library][core-data-stack-link].

Lots of advice you see online will be to create the `NSManagedObjectContext` in the AppDelegate, but I think it's a better idea to create a separate `CoreDataManager` class to deal with everything `CoreData` related.


2) Create a new xcdatamodel object. Add a new `Article` entity. Add the `id`, `title`, and `body` fields. Make sure that the `id` field is Integer64 and use Int64 as the data type in the model. We can't just use Int, since the Int type is 64 bits on 64 bit devices, and 32 bits on 32 bit devices, and will cause core data to crash.

3) Add the following function to the global scope:

{% highlight swift %}

func sync<T: BaseModel>(key: String = "id", type: T.Type, objects: [JSONDictionary], context: NSManagedObjectContext) {

  context.performBlock {
    // sort the json dictionary by id
    var ids:[Int] = []

    // pull the ids out of the json array
    for json in objects {
      let id = json[key] as! Int
      ids += [id]
    }

    // sort the ids
    ids.sort { before, after in
      return before < after
    }

    // Sort the objects so they have the same sorting as the ids

    var sortedObjects = objects
    sortedObjects.sort { before, after in
      before[key]!.longLongValue < after[key]!.longLongValue
    }

    // fetch the set of events in the dictionary
    let predicate = NSPredicate(format: "(\(key) IN %@)", ids)
    let sortDescriptor =  NSSortDescriptor(key: key, ascending: true)

    let entityDescription = NSEntityDescription.entityForName(T.entityName, inManagedObjectContext: context)!
    var fetchRequest = FetchRequest<T>(entity: entityDescription)
    fetchRequest.predicate = predicate
    fetchRequest.sortDescriptors = [sortDescriptor]
    let result = fetch(request: fetchRequest, inContext: context)

    // walk the id list and create or update
    var i = 0; // index to the ids list
    var j = 0; // index to the resuts.obj list
    for (var i = 0; i < ids.count; i++) {
      let id = ids[i]
      let dict = sortedObjects[i]

      if (j >= result.objects.count) {
        let newObject = T.fromDictionary(dict, context: context)
        continue
      }

      let obj = result.objects[j] as T

      if Int(obj.id) == id {
        // exists in core data. update the object
        obj.populateWithJSON(dict, context: context)
        j++
      } else {
        // doesn't exist. Create it
        let newObject = T.fromDictionary(dict, context: context)
      }

    }

    // save context up parent chain recursively
    saveRecurse(context)
  }
}

{% endhighlight %}

Let's break down what is going on here. First, we are passed in an array of `JSONDictionary` objects, and a key to use (`id` by default) for uniquing the objects in the data store, and the type T: BaseModel, which constrains the type to be of `BaseModel` so that we have convenience methods for inserting an object into the `NSManagedObjectContext`, and for populating the objects from a `JSONDictoinary`.

Next, we sort the objects by their key.

We then construct a `NSFetchRequest` using the key as the sort descriptor, and a predicate that will only pull objects from `Core Data` if the id is in the list of ids.


We now have two lists: One list of objects from the web service, and one list of objects containing data that already exists in `Core Data`. Our task now is to walk down these two lists and either update the existing object in `Core Data`, or insert the object if it doesn't exist. This will be doable in a single pass, since both lists are already sorted.

We do all this work in a context.performBlock block so that it can all be done with a background child context if desired.


{% highlight swift %}

func saveRecurse(context: NSManagedObjectContext) {
  context.performBlock {
    saveContext(context) { result in
      if let context = context.parentContext {
        saveRecurse(context)
      }
    }
  }
}

{% endhighlight %}

This function will recursively save the `NSManagedObjectContext` asynchronously all the way up the parent chain. This way, you can save a child `NSManagedObjectContext` and all the parents will be saved as well. This is useful for syncing objects into `Core Data` in the background, so the UI thread is never locked.

[core-data-stack-link] www.google.com
[part1] eliasbagley.github.io
