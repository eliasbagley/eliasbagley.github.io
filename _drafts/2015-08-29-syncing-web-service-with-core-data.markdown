---
layout: post
title:  "Syncing a web API with UITableView using Core Data NSFetchedResultsController in Swift"
date:   2015-08-29 23:22:15
categories: JSON API CoreData UITableView NSFetchedResultsController
---

The problem: I want to show data from a paged JSON API endpoint in a UITableView

The solution consists of 3 parts:

1) Load the data from the API endpoint into an array of JSON objects

2) Sync the JSON object array with core data. For each item in the list, add the item if it's new, or update the existing object if it's already managed by core data

3) Use NSFetchedResultsController to watch for changes to core data, and update the UITableView to reflect the core data updates


1) Load the dat afrom the API endpoint into an array of JSON objects

We will use alamofire for this





You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

To add new posts, simply add a file in the `_posts` directory that follows the convention `YYYY-MM-DD-name-of-post.ext` and includes the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll’s dedicated Help repository][jekyll-help].

[jekyll]:      http://jekyllrb.com
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help
