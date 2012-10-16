Edge Side Includes is a standard supported by Varnish and pricier CDNs (eg akami), which allows different parts of a page to have different cache lifetimes.

For example, consider the following HTML:

    <HTML><head><script src="/slow_to_load.js"></script></head><body>
    <h1> Static content </h1>
    <h2> Thanks for joining, John </h2>
    </body></html>

Because the "Thanks for joining" message is only shown if you joined in the last 24 hours, this page cannot be cached. 
Worse still, because we need to hit the application stack and load domain objects, the whole page takes 250ms to start sending.

However, with ESI this could look like:

    <HTML><head><script src="/slow_to_load.js"></script></head><body>
    <h1> Static content </h1>
    <esi:include src="/slow_loading_welcome_message" />
    </body></html>

Once the cache is hot, Varnish will respond instantly to a request by sending the first two lines.
Then it will ask the app server for `"/slow_loading_welcome_message"`.
While that resource is generated, the browser can parse the start of the page, start downloading `"/slow_to_load.js"` and even render the `<h1>` tag.
Once the loading message is generated, varnish sends it and the closing tags to the browser (on the same connection, which it has held open the whole time).

![ESI diagram](https://raw.github.com/DanielHeath/ESI_PRESENTATION/master/esi.png "ESI diagram")

99designs.com could (for instance) use this to load individual contest pages more quickly, 
by moving user-specific details (like which designs are visible) into ESI tags.
That way, the heading will be displayed quickly and resources like /analytics/ga.js and jquery-ui can start downloading early.

This results in the whole page loading faster because the static resources download while the page is generated.