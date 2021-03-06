---
layout: post
title:  "Making Network Requests with Alamofire"
date:   2015-09-25 23:22:15
comments: true
categories: JSON API Alamofire
---

Almost every iOS app needs to interact with a remote API in some form or another. In this post I want to detail how I set up my networking layer using Alamofire in Swift. Enhancing this approach to work with Core Data will come in a future article.

Our approach will be:

1) Use an enum to model the API Endpoints <br>
2) Use router objects to encapsulate the details of an NSURLRequest for each API endpoint case <br>
3) Use a manager singleton, which kicks off the request using Alamofire with a Router object and endpoint <br>
4) Add a completion closure so the initiating view controller can be informed when the request completes <br>

We will use enums to model API endpoints, Router objects to manage details of making requests to the endpoint, a manager object singleton to perform the request, and a callback closure to get the results of the API call back to the view controller that initiated the request.

First, let's install some 3rd party libraries we'll be using

Cartfile

{% highlight swift %}

github "Alamofire/Alamofire" == 1.3.0
github "Thomvis/SwiftyJSON" == 2.2.1
github "swisspol/GCDWebServer" ~> 3.2.5

{% endhighlight %}

run:

{% highlight swift %}

> carthage update

{% endhighlight %}



Link in .framework files into the xcode project following the instructions at the [Carthage][carthage-github] site


Use the following typdef to represent a JSON dictionary:

{% highlight swift %}

public typealias JSONDictionary = [String: AnyObject]

{% endhighlight %}


We'll be working with an Article model, which has properties for an id, title, and body, and a function which parses a JSONDictionary into an instance of the Article class

{% highlight swift %}

import Foundation

class Article {
  var id: Int?
  var title: String?
  var body: String?


  init(json: JSONDictionary) {
    populateWithJSON(json)
  }

  func populateWithJSON(dictionary: JSONDictionary) {
    if let id = dictionary["id"] as? Int {
      self.id = id
    }
    if let title = dictionary["title"] as? String {
      self.title = title
    }
    if let body = dictionary["body"] as? String {
      self.body = body
    }
  }

  func toJsonDictionary() -> JSONDictionary {
    var dict: JSONDictionary = [:]

    if let id = id { dict["id"] = id }
    if let title = title { dict["title"] = title }
    if let body = body { dict["body"] = body }

    return dict
  }
}

{% endhighlight %}

I use this approach for JSON parsing over something nicer like ObjectMapper, because it allows more flexibility in the way objects are initialized. For example, when we switch to use core data, we don't have the ability to init() our objects ourselves - core data does that for us, therefore we need a way to populate objects that have already been initialized. Hence, populateWithJson

Next lets add the base router class:

{% highlight swift %}

import Foundation
import Alamofire

public typealias JSONDictionary = [String: AnyObject]
typealias APIParams = [String : AnyObject]?

protocol APIConfiguration {
  var method: Alamofire.Method { get }
  var encoding: Alamofire.ParameterEncoding? { get }
  var path: String { get }
  var parameters: APIParams { get }
  var baseUrl: String { get }
}

class BaseRouter : URLRequestConvertible, APIConfiguration {

  init() {}

  var method: Alamofire.Method {
    fatalError("[\(reflect(self).summary) - \(__FUNCTION__))] Must be overridden in subclass")
  }

  var encoding: Alamofire.ParameterEncoding? {
    fatalError("[\(reflect(self).summary) - \(__FUNCTION__))] Must be overridden in subclass")
  }

  var path: String {
    fatalError("[\(reflect(self).summary) - \(__FUNCTION__))] Must be overridden in subclass")
  }

  var parameters: APIParams {
    fatalError("[\(reflect(self).summary) - \(__FUNCTION__))] Must be overridden in subclass")
  }

  var baseUrl: String {
    let baseUrl = NSBundle.mainBundle().stringForKey("API_BASE_URL")!
    return baseUrl
  }

  var URLRequest: NSURLRequest {
    let baseURL = NSURL(string: baseUrl);
    let URLRequest = NSURLRequest(URL: baseURL!.URLByAppendingPathComponent(path))
    let mutableURLRequest = NSMutableURLRequest(URL: baseURL!.URLByAppendingPathComponent(path))
    mutableURLRequest.HTTPMethod = method.rawValue
    if let encoding = encoding {
      return encoding.encode(mutableURLRequest, parameters: parameters).0
    }
    return mutableURLRequest
  }

}


{% endhighlight %}

Alamofire needs an NSURLRequest to be able to make a request, which is what is being constructed in the URLRequest computed property. We add the URLRequestConvertible protocol to this object so that the router object itself can be treated as a NSURLRequest, and passed to any method (such as alamofire's request method) which expects one.

APIConfiguration is a protocol which allows us to configure each endpoint in the enum with the information we'll need for each endpoint, such as the encoding method, base url, HTTP method, and path. Lets take a look at an example:

ArticleRouter.swift
{% highlight swift %}

import Foundation
import Alamofire

enum ArticleEndpoint {
  case GetArticles(page: Int)
  case GetArticle(id: Int)
  case UpdateArticle(article: Article)
  case DeleteArticle(id: Int)
}

class ArticleRouter : BaseRouter {

  var endpoint: ArticleEndpoint
  init(endpoint: ArticleEndpoint) {
    self.endpoint = endpoint
  }

  override var method: Alamofire.Method {
    switch endpoint {
    case .GetArticles: return .GET
    case .GetArticle: return .GET
    case .UpdateArticle: return .PUT
    case .DeleteArticle: return .DELETE
    }
  }

  override var path: String {
    switch endpoint {
    case .GetArticles: return "articles.json"
    case .GetArticle(let id): return "/\(id)/article.json"
    case .UpdateArticle(let article): return "/\(article.id)/article.json"
    case .DeleteArticle(let id): return "/\(id)/article.json"
    }
  }

  override var parameters: APIParams {
    switch endpoint {
    case .GetArticles(let page):
      return ["page" : page]

    case .UpdateArticle(let article):
      return article.toJsonDictionary()

    default: return nil
    }
  }

  override var encoding: Alamofire.ParameterEncoding? {
    switch endpoint {
    case .GetArticles: return .URL
    case .GetArticle: return .URL
    default: return .JSON
    }
  }

}

{% endhighlight %}

Notice the ArticleEndpoint enum: this enum represents the set of actions we can do with articles on our server. In this case, we can get a list of articles, get the details for a single article, update an article, and delete an article.

For each of the methods in the BaseRouter, we override them in the ArticleRouter and switch on the ArticleEndpoint to configure the information for that endpoint. The method, path, parameters, and encoding fully specify the request, and is enough information for Alamofire to make the request to the server.

Next, we can create a manager object which will make the requests for us.

{% highlight swift %}

import Foundation
import Alamofire

public class APIManager {

  public class var sharedInstance: APIManager {
    struct Singleton {
      static let instance : APIManager = APIManager()
    }
    return Singleton.instance
  }

  let manager = Manager()

  init() {
  }

  //MARK: methods

  func getArticles(page: Int = 1, completion: (articles: [Article]) -> ()) -> Request {
    let router = ArticleRouter(endpoint: .GetArticles(page: page))

    return manager.request(router)
      .validate()
      .responseJSON { (request, response, object, error) -> Void in
        if let error = error {
          println(error)
          return;
        }

        let articleJson = ((object as! JSONDictionary)["articles"] as? [JSONDictionary])!

        var objs: [Article] = []

        for json in articleJson {
          objs += [Article(json: json)]
        }

        completion(articles: objs)
    }
  }
}


{% endhighlight %}

This manager uses the singleton pattern to hold onto a reference of the Alamofire `Manager` class to make network requests. We make a request in the function `getArticles()`, which constructs a router object using the parameters provided, and uses the router object as an NSURLRequest to make a network request with Alamofire. The results are returned in the `responseJSON` block, and `Article` objects are created from the JSON response, and passed back to the completion handler.

Next we need a view controller to initiate the request and handle the response. Our example view controller will look like this:

{% highlight swift %}

import UIKit
import Alamofire

class ViewController: UIViewController {

  var request: Request?

  required init(coder aDecoder: NSCoder) {
    super.init(coder: aDecoder)
  }

  override func viewDidLoad() {
    super.viewDidLoad()
    loadData()
  }

  deinit {
    request?.cancel()
  }

  func loadData() {
    request = APIManager.sharedInstance.getArticles { [unowned self] articles in
      self.request = nil
      println("received articles: \(articles)")
    }
  }

}

{% endhighlight %}

This `UIViewController` will initiate a network request when the view controller loads, and then print out the resulting list of objects to the console once the network requests completes.

Now we have the ability to make a request to a web server and receive a list of objects back, as a list of `Article` objects that can be used in our app. The only difficulty with the current approach is that we have to keep a local copy of `Article` objects inside of the view controller, in order to present them in the UI of our app. This means that we need to cate care of all the data binding and syncing manually, which can be a pain. For example, if the user edits an object in a detail page, those changes will need to be sent back to the list in order to keep the list of data and detail view controller for that object in sync. We can prevent the need to have a local copy of the `Article` list in the view controller by first putting the `Articles` into Core Data, and using an `NSFetchedResultsController` to bind the data in core data to the UI. Stay tuned for the next article.





An executable is worth a thousand words, so you can find a working example of the code on [Github][source-github]. The example code will host a local test server so you can make network requests to a mocked Artilce API.


[carthage-github]: https://github.com/Carthage/Carthage
[source-github]: https://github.com/eliasbagley/alamofire-example-1
