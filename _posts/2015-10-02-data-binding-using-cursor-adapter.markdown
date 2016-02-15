---
layout: post
title:  "Android Data Binding and Persistence Using ActiveAndroid, CursorAdapter and ListView: a Poor Man's NSFetchedResultsController"
date:   2015-10-02 13:22:00
comments: true
categories: Android
---

Edit: I don't think this approach is that good anymore. Replace the `Loader` with `RxJava`.

A common paradigm in many Android applications is to load a list of objects from a webservice, and display it in in a `ListView`. The easiest and most common way to implement this is:

1) Launch the Activity which contains the `ListView` and a subclass of `BaseAdapter`, which holds a reference to a list in memory<br>
2) When the Activity finishes loading, initiate a network request to fetch a list of objects from the web service<br>
3) When the network request finishes, pass the list of objects back to the Activity<br>
4) The activity passes the list to the adapter<br>
5) The adapter calls `notifyDataSetChanged`, which causes the adapter to refresh and create the views<br>
6) The views in the list are displayed<br>

It probably looks something like this:

`MainActivity.java`

{% highlight java %}

...

private void initialize() {
  apiManager.fetchList(new Callback<List<Item>>() {
      @Override
      public void onSuccess(List<Item> items, Response response) {
        _adapter.setItems(items);
      }

      @Override
      public void onFailure(RetrofitError error) {
        // Handle error
      }
  });
}

...

{% endhighlight %}

`MyAdapter.java`

{% highlight java %}

...

private List<Item> _items;

public void setItems(List<Item> items) {
  _items = items;
  notifyDataSetChanged();
}

...

{% endhighlight %}

This approach works, but it has a couple problems.

1) The list isn't cached or persisted anywhere, so there will be a short delay after the activity loads before the data is shown<br>

2) The list of items is passed around from the network layer, to the Activity, to the adapter. Keeping the data model in memory means that we have to manually keep this in memory data model in sync with the API. If we know that the server's data model has changed (for example, because we created or updated an object), then we need to make sure to sync with the API again, and again pass the `List<Item>` from the network layer to the Activity to the Adapter.<br>

3) If we want to paginate the list, we have to do this same thing every time we request a new page<br>




I recently started working with `Core Data` and `NSFetchedResultsController` on iOS. `Core Data` is a little bit of a pain to get set up working correctly, but once it does it offers excellent binding between the data store and the `UITableViewController`. Any time a new object is saved into the data store, the `NSFetchedResultController` notices and automatically adds it into the list, without us having to store and manipulate a copy of the list in memory. The list items stay in the database, and the `NSFetchedResultsController` does the rest, with only a small amount of boilerplate code. For example, if I request page 2 of a paginated data set, the network layer can store the objects in core data, and the `UITableView` will automatically update to reflect these changes. Passing the list from the network layer to the UI is not necessary!

I wanted this same ability for Android, and after a little bit of research I discovered that a `CursorAdapter` fits the bill. A `CursorAdapter` is a subclass of `BaseAdapter` which is responsible for binding data between the UI and a database, using a `Cursor` object, which is basically an interface to access the database. A `Cursor` is pretty much what it sounds like: the database table is a 2 dimentional data structure consisting of rows of items, each with multiple columns. The `Cursor` iterates over this data structure, acting like a text editor cursor that can move up or down rows. It starts at index -1, and can iterate through each row of the table by calling `cursor.moveToNext()`, and can access columns with `int index = cursor.getColumnIndexOrThrow("columnName")`, and `int value = cursor.getInt(index)` or `String value = cursor.getString(index)` etc.

In order to implement this data binding solution, we'll need to tie a couple pieces together:

1) A network request to get a list of objects<br>
2) Something for data persistence, either a plain sqlite database or an ORM. In this case we'll use `ActiveAndroid`<br>
3) The `CursorAdapter`<br>
4) A `CursorLoader`

Let's look at how the pieces fit together.

I'm using [`Retrofit`][retrofit-link] for the network request, and [`ActiveAndroid`][active-android-link] for persistence. See the `ActiveAndroid` for instructions on how to set it up.

First lets add a couple of dependencies to make our life easier:

`build.gradle`

{% highlight groovy %}
compile 'com.google.code.gson:gson:2.2.+'
compile 'com.jakewharton:butterknife:5.1.+'
compile 'com.squareup.dagger:dagger:1.2.2'
compile 'com.squareup.retrofit:retrofit:1.9.0'
compile 'com.squareup.okhttp:okhttp-urlconnection:2.0.0'
compile 'com.squareup.okhttp:okhttp:2.0.0'
compile 'com.michaelpardo:activeandroid:3.1.0-SNAPSHOT'
provided 'com.squareup.dagger:dagger-compiler:1.2.2'
{% endhighlight %}

Next, we need a model to work with.

`Article.java`

{% highlight java %}

@Table(name = "Articles", id = BaseColumns._ID)
public class Article extends Model {

    public static final String ID = "id";
    public static final String TITLE = "title";
    public static final String BODY = "body";

    @SerializedName(ID)
    @Column(name = ID, unique = true, onUniqueConflict = Column.ConflictAction.REPLACE)
    public Integer id;

    @SerializedName(TITLE)
    @Column(name = TITLE)
    public String title;

    @SerializedName(BODY)
    @Column(name = BODY)
    public String body;

    public Article() {
    }

    public static Article fromCursor(Cursor cursor) {
        Article article = new Article();
        article.loadFromCursor(cursor);
        return article;
    }
}

{% endhighlight %}

The `@SerializedName` annotations are for `GSON` to serialize the `JSON` to our `Article` object. The `@Table` and `@Column` annotations are for `ActiveAndroid`. Pretty self explanatory.

The `fromCursor(Cursor cursor)` method is what will allow our `CursorAdapter` to turn a Cursor into an instance of our `Article` object.


Next, let's grab load a list of `Article` objects from out web server and store them in the database.

`ArticleAPI.java`

{% highlight java %}

public interface ArticleAPI {
    @GET("articles.json")
    void getArticles(Callback<ArticlesServiceResponse> callback);
}

{% endhighlight %}


`ArticleManager.java`

{% highlight java %}

public class ArticleManager {

    private ArticleAPI _articleAPI;

    @Inject
    public ArticleManager(ArticleAPI articleAPI) {
        _articleAPI = articleAPI;
    }

    public void getArticles(final Callback<ArticlesServiceResponse> callback) {
        _articleAPI.getArticles(new Callback<ArticlesServiceResponse>() {
            @Override
            public void success(final ArticlesServiceResponse articlesServiceResponse, Response response) {
                for (Article a : articlesServiceResponse.articles) {
                    a.save();
                }

                callback.success(articlesServiceResponse, response);

            }

            @Override
            public void failure(RetrofitError error) {
                callback.failure(error);
            }
        });
    }

}


class ArticlesServiceResponse {
    public List<Article> articles;
}

{% endhighlight %}

I'm using [`Dagger`][dagger-link] to inject the ArticleAPI into the ArticleManager.


Next, let's create a custom view which will be the cell in the ListView.

`ArticleCell.java`

{% highlight java %}

public class ArticleCell extends RelativeLayout {

    @InjectView(R.id.article_title) TextView _title;

    public ArticleCell(Context context) {
        super(context);
        initialize(context);
    }

    public ArticleCell(Context context, AttributeSet attrs) {
        super(context, attrs);
        initialize(context);
    }

    public ArticleCell(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initialize(context);
    }

    private void initialize(Context context) {
        LayoutInflater.from(context).inflate(R.layout.cell_article, this);
        ButterKnife.inject(this);
    }

    public void populate(Article article) {
        _title.setText(article.title);
    }
}

{% endhighlight %}


`cell_article.xml`

{% highlight xml %}

<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    >

    <TextView
        android:id="@+id/article_title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="15dp"
        android:layout_marginBottom="15dp"
        />

</RelativeLayout>

{% endhighlight %}

The `populate(Article article)` method is what binds our Article model to the UI elements.

Next, our `CursorAdapter`

`ArticleAdapter.java`

{% highlight java %}

public class ArticleAdapter extends CursorAdapter {
    public ArticleAdapter(Context context, Cursor c, boolean autoRequery) {
        super(context, c, autoRequery);
    }

    public ArticleAdapter(Context context, Cursor c, int flags) {
        super(context, c, flags);
    }

    @Override
    public View newView(Context context, Cursor cursor, ViewGroup viewGroup) {
        return new ArticleCell(context);
    }

    @Override
    public void bindView(View view, Context context, Cursor cursor) {
        ArticleCell cell = (ArticleCell) view;
        cell.populate(Article.fromCursor(cursor));
    }
}

{% endhighlight %}

All we need is the two inherited constructors, and a `newView` method and `bindView method`. `newView` instantiates the view, and `bindView` creates an Article object from the Cursor, and then populates the `ArticleCell`. The `CursorAdapter` has the added benefit of handling the [ViewHolder][view-holder-link] pattern for us.

Now let's tie it all together with the Activity:

`MainActivity.java`

{% highlight java %}

public class MainActivity extends ActionBarActivity {

    @InjectView(R.id.main_list_view) ListView _listView;
    private ArticleAdapter _adapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.inject(this);

        _adapter = new ArticleAdapter(this, null, 0);
        _listView.setAdapter(_adapter);

        setup();
    }

    private void setup() {
        getSupportLoaderManager().initLoader(0, null, new LoaderManager.LoaderCallbacks<Cursor>() {
            @Override
            public Loader<Cursor> onCreateLoader(int id, Bundle args) {
                return new CursorLoader(MainActivity.this,
                        ContentProvider.createUri(Article.class, null),
                        null, null, null, null);
            }

            @Override
            public void onLoadFinished(Loader<Cursor> loader, Cursor cursor) {
                _adapter.swapCursor(cursor);
            }

            @Override
            public void onLoaderReset(Loader<Cursor> loader) {
                _adapter.swapCursor(null);
            }
        });

    }
}

{% endhighlight %}

We use the Activity's `LoaderManager` and a `CursorLoader` so that we don't read from the database on the UI thread, which can make our UI laggy or cause an ANR crash.

The `ContentProvider` in the `onCreateLoader` activity can be customized to provide filtering and sorting by passing in extra parameters. See the [`ContentProvider`][content-provider-doc-link] docs for more information.

To get `ContentProvider` to work properly, you'll need the following in `AndroidManifest.xml`:

{% highlight xml %}
<application>
    ...
      <provider android:authorities="${applicationId}" android:exported="false" android:name="com.activeandroid.content.ContentProvider" />
    ...
</application>
{% endhighlight %}

Make sure to replace `com.eliasbagley.example` with your app's package.

Great! Now anytime we save a new `Article` object into the database, the `CursorAdapter` will automatically display it in the `ListView`.

In summary:

1) Create all the models and views<br>
2) Fetch a list of objects from the web service, and save into the database<br>
3) Use a `CursorLoader` and `CursorAdapter` to bind the database to the UI<br>

One nice thing that `UITableView` and `NSFetchedResultsController` can do that we don't currently have is insertion, deletion, and move animations. Stay tuned for part 2 where we will replace the `ListView` with the new [`RecyclerView`][recycler-view-link], which can support these animations.

You can find the source code for the examples [here][example-code-link].

[retrofit-link]: https://github.com/square/retrofit
[active-android-link]: https://github.com/pardom/ActiveAndroid
[dagger-link]: https://github.com/square/dagger
[content-provider-doc-link]: http://developer.android.com/reference/android/content/ContentProvider.html
[view-holder-link]: http://developer.android.com/training/improving-layouts/smooth-scrolling.html#ViewHolder
[recycler-view-link]: https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html
[example-code-link]: https://github.com/eliasbagley/android-data-binding-1
