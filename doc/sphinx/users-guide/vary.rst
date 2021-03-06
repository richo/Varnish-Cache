.. _users-guide-vary:

HTTP Vary
---------

*HTTP Vary is not a trivial concept. It is by far the most
misunderstood HTTP header.*

A lot of the response headers tell the client something about the HTTP
object being delivered. Clients can request different variants a an
HTTP object, based on their preference. Their preferences might cover
stuff like encoding or language. When a client prefers UK English this
is indicated through "Accept-Language: en-uk". Caches need to keep
these different variants apart and this is done through the HTTP
response header "Vary".

When a backend server issues a "Vary: Accept-Language" it tells
Varnish that its needs to cache a separate version for every different
Accept-Language that is coming from the clients.

If two clients say they accept the languages "en-us, en-uk" and "da,
de" respectively, Varnish will cache and serve two different versions
of the page if the backend indicated that Varnish needs to vary on the
Accept-Language header.

Please note that the headers that Vary refer to need to match
*exactly* for there to be a match. So Varnish will keep two copies of
a page if one of them was created for "en-us, en-uk" and the other for
"en-us,en-uk". Just the lack of space will force Varnish to cache
another version.

To achieve a high hitrate whilst using Vary is there therefor crucial
to normalize the headers the backends varies on. Remember, just a
difference in case can force different cache entries.

The following VCL code will normalize the Accept-Language headers, to
one of either "en","de" or "fr"::

    if (req.http.Accept-Language) {
        if (req.http.Accept-Language ~ "en") {
            set req.http.Accept-Language = "en";
        } elsif (req.http.Accept-Language ~ "de") {
            set req.http.Accept-Language = "de";
        } elsif (req.http.Accept-Language ~ "fr") {
            set req.http.Accept-Language = "fr";
        } else {
            # unknown language. Remove the accept-language header and 
	    # use the backend default.
            remove req.http.Accept-Language
        }
    }

The code sets the Accept-Encoding header from the client to either
gzip, deflate with a preference for gzip.

Vary parse errors
~~~~~~~~~~~~~~~~~

Varnish will return a 503 internal server error page when it fails to
parse the Vary server header, or if any of the client headers listed
in the Vary header exceeds the limit of 65k characters. An SLT_Error
log entry is added in these cases.

Pitfall - Vary: User-Agent
~~~~~~~~~~~~~~~~~~~~~~~~~~

Some applications or application servers send *Vary: User-Agent* along
with their content. This instructs Varnish to cache a separate copy
for every variation of User-Agent there is. There are plenty. Even a
single patchlevel of the same browser will generate at least 10
different User-Agent headers based just on what operating system they
are running. 

So if you *really* need to Vary based on User-Agent be sure to
normalize the header or your hit rate will suffer badly. Use the above
code as a template.

