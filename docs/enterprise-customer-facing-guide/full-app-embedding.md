## Embedding all of Metabase in your web app
The open-source edition of Metabase allows you to [embed standalone charts or dashboards](../administration-guide/13-embedding.md) in your own web applications for simple situations. But what if you want to provide your users with a more interactive, browsable experience? Metabase Enterprise Edition allows you to embed the entire Metabase app within your own web app, allowing you to provide drill-through for your embedded charts and dashboards, or even embed the graphical query builder, or collections of dashboards and charts.

You'll be putting the whole Metabase app into an iframe, and the SSO integration you've set up with Metabase will be used to make sure the embedded Metabase respects the collection and data permissions you've set up for your user groups. Clicking on charts and graphs in the embed will do just what they do in Metabase itself. The only difference is that Metabase's top nav bar and global search will not be rendered in your iframe.

### What you'll be doing
To get this going, you're going to need:
* An Enterprise Edition instance of Metabase that contains the dashboards and charts that you'd like to embed.
* A separate web application that you want to embed your dashboards and charts in.

### Enabling embedding in Metabase
First, let's enable embedding in your Metabase instance. Go to the Admin Panel, and under Settings, go to the “Embedding in other applications” tab. From there, click “Enable.”

Once you do, you'll see a set of options:

* **Embedding secret key:** You can ignore this setting, which is only for standalone chart or dashboard embeds.

* **Embedding the entire metabase app:** Here's where you'll enter the base URL of the web application that you want to allow to embed Metabase. Only include the protocol and the host. For example, `http://my-web-app.example.com/`. If you're a fancy person, you can specify this URL in the environment variable `MB_EMBEDDING_APP_ORIGIN`.

### Setting things up in your web app
To give you a picture of what you'll need to do in your app, we've created this [reference app](https://github.com/metabase/sso-examples/tree/master/app-embed-example). If you use React in your application, [this React component](https://github.com/metabase/sso-examples/blob/master/app-embed-example/src/MetabaseAppEmbed.js) may be helpful.

The main elements you'll need to embed Metabase in your app are:

* An SSO authentication endpoint in your application, typically JWT or SAML, and Metabase configured to use them.
* An `<iframe>` element in your application, with the `src` attribute set to either a URL in Metabase (e.x. `http://metabase.yourcompany.com/dashboard/1`), or directly to the authentication endpoint in your application, with a parameter specifying where to redirect back to (e.x. `http://yourcompany.com/api/auth?redirect=http%3A%2F%2Fmetabase.yourcompany.com%2Fdashboard%2F1`). The latter will avoid an extra redirect if the user has not been authenticated.
* Optional: JavaScript using [postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) in your application for enabling communication to and from the embedded Metabase. Here are the types of `postMessage` messages we currently support:

#### Supported `postMessage` messages *from* embedded Metabase:

* `frame` message with `normal` mode: the current page in the embedded Metabase will fill whatever size `iframe` it is displayed in, e.x. the `/question` pages:

    { "metabase": { "type": "frame", "frame": { "mode": "normal" }}}

* `frame` message with `fit` mode: the current page in the embedded Metabase expects a specific aspect ratio, e.x. `/dashboard` pages, so the `iframe` should be set to specified height if you don't want it to scroll:

    { "metabase": { "type": "frame", "frame": { "mode": "fit", height: HEIGHT_IN_PIXELS }}}

* `location` message: the embedded Metabase changed URLs. Use this for deep-linking, etc (`location` mirrors `window.location`):

    { "metabase": { "type": "location", "location": LOCATION_OBJECT }}

#### Supported `postMessage` messages *to* embedded Metabase:

* `location` message: change the URL of the embedded Metabase:

    { "metabase": { "type": "location", "location": LOCATION_OBJECT_OR_URL }}


### Choosing what to embed
The exact next steps will differ depending on your specific needs and goals, but the basic tool you have at hand now is that you can make any link in your web app render a particular page from your Metabase instance.

So if you have for example a "Stats" or "Analytics" page in your web app, you could have that page display one of your Metabase dashboards. What's powerful about this type of embedding vs. standalone chart or dashboard embeds though is that your users will be able to click on the individual charts in that dashboard to see them in more detail, and further explore them using drill-through, or even Metabase's graphical query builder.

You can even display a specific Metabase collection in an embed to allow your users to browse through all the dashboards and questions that you've made available to them.

### A note on drill-through and permissions
One of the main differences between embedding the full Metabase app vs. standalone embeds is that charts and graphs will have drill-down enabled. This lets your users click on charts to zoom in, pivot, and generally explore more.

#### What does drill-through let my users do exactly?
When clicking on any part of a chart — like a dot, bar, slice, or country — your users will see the drill-through action menu. This will let them do things like:
* See the unaggregated rows for that point on the chart.
* Zoom in on the clicked point on a time series
* Pivot or break out the clicked point by an available dimension to see a new chart
* Use the X-ray or Comparison actions, if you haven't turned X-rays off in the Admin Panel, which will display an automatic analysis of the clicked point.

Drill-through also allows users to click on the title of a chart in a dashboard to see the detail view of that question. From the detail view, if they have data permissions, they can use Metabase's graphical query editor to explore further. If they've been given SQL editor permissions, they can also view the SQL for the question and edit it to explore more.

Depending on the collections permissions you set, your users can also save their explorations into collections. If you want to allow them to find these saved explorations, make sure your web application implements a link to view the collections directory.

#### Using SSO to apply data or collection permissions to embeds
If you're using SSO to authenticate users in your web app and you've also connected your SSO to Metabase, users who authenticate into your web application will automatically have their Metabase group permissions applied when viewing the dashboards, charts, or collections you embed. This means that once you've set up sandboxes and data and collection permissions in Metabase, you don't need to think about what your web app users can see when exploring.

---

## Next: customizing drill-through
Tailor what happens when your customers click on charts or graphs by [customizing drill-through](customizing-drill-through.md).