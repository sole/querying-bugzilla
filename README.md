# Querying Bugzilla

The queries start with `https://bugzilla.mozilla.org/buglist.cgi?` and then all the search parameters you need to use, using pairs of `parameter=value` joined by `&`, as in for example:

`https://bugzilla.mozilla.org/buglist.cgi?bug_status=RESOLVED&bug_status=VERIFIED&bug_status=CLOSED&chfield=resolution&chfieldfrom=-8d&chfieldto=Now&chfieldvalue=FIXED&product=DevTools`

Here is [more documentation](https://bugzilla.mozilla.org/page.cgi?id=quicksearch.html) on querying Bugzilla.

## Interesting fields and data types

### `bug_status`, `chfield`, `chfieldvalue` (for finding "done" bugs)

A bug is considered to be "closed" (as in "done", for our purposes) when this happens:

`bug_status = (RESOLVED,VERIFIED,CLOSED) + chfield=resolution + chfieldvalue = FIXED`

Bugzilla will try an "OR" search, so to add these three multiple possible values of `bug_status`, you specify the parameter three times in the url query: `bug_status=RESOLVED&bug_status=VERIFIED&bug_status=CLOSED`

### Times and dates

When querying, you can use specific dates, in the format: `YYYY-MM-DD`

Or you can also use relative values: -8d is 8 days ago. Now is... *now!*

## Useful searches

### Bugs resolved in the last week in the whole of DevTools

https://bugzilla.mozilla.org/buglist.cgi?bug_status=RESOLVED&bug_status=VERIFIED&bug_status=CLOSED&chfield=resolution&chfieldfrom=-8d&chfieldto=Now&chfieldvalue=FIXED&product=DevTools

### Bugs resolved in the last week in a specific component of DevTools

Append the component parameter to the previous DevTools product search. For example, to search for `General` bugs, we append `&component=General`.

https://bugzilla.mozilla.org/buglist.cgi?bug_status=RESOLVED&bug_status=VERIFIED&bug_status=CLOSED&chfield=resolution&chfieldfrom=-8d&chfieldto=Now&chfieldvalue=FIXED&product=DevTools&component=General

### Bugs created in the last week

Sort of similar to the previous query, but looking at bugs whose `creation_ts` field changed in the last week:

https://bugzilla.mozilla.org/buglist.cgi?product=DevTools&chfield=creation_ts&chfieldfrom=-8d&chfieldto=Now&include_fields=creation_ts


### Bugs that block another bug

Append the `blocked=<bug id>`. Useful if you want to track work happening on a meta bug.

For example, the bugs that block DevTools fission work:

https://bugzilla.mozilla.org/buglist.cgi?blocked=1451861

Note that it DOES NOT work with its alias: https://bugzilla.mozilla.org/buglist.cgi?blocked=dt-fission will return *Zarro boogs*. Always use the bug id to avoid disappointments.

The `blocked` search does not extend to child bugs, so if you meta bug has bugs which have more child bugs, you won't have full visibility on these.

If you know the bug ids of each meta bug in your project, you could specify multiple values for the `blocked` field, and Bugzilla will aggregate the results (it's an "OR" style of search).

For example, this search looks for bugs that block the main `dt-fission` bug, and one of its metas:

https://bugzilla.mozilla.org/buglist.cgi?blocked=1451861&blocked=1450150


### Keyword searches

These are special 'tags' that can be added to bugs. Some teams watch for bugs containing those tags, and then do something about it. For example, the documentation team watch for appearances of `dev-doc-needed` to signify a feature needs to be documented.

For example, all the bugs in DevTools which are `meta` bugs:

https://bugzilla.mozilla.org/buglist.cgi?product=DevTools&keywords=meta

(By the way, there are some truly old bugs here, if you want to have a go at retriaging them and possibly close them if they're not relevant anymore).

Or the DevTools bugs for documentation work:

https://bugzilla.mozilla.org/buglist.cgi?product=DevTools&keywords=dev-doc-needed

Or good first bugs!

https://bugzilla.mozilla.org/buglist.cgi?product=DevTools&keywords=good-first-bug

### Whiteboard searches

Very similar to the keyword searches, except you don't need a Bugzilla admin to add keywords to the database before you can use them; the whiteboard is a free space!

Note that the query parameter is called `status_whiteboard`, not just `whiteboard`.

For example, DevTools bugs marked with `dt-fission`:

https://bugzilla.mozilla.org/buglist.cgi?status_whiteboard=dt-fission

### Assigned bugs

This is a bit tricky and there seem to be two different cases, depending on whether we want to find assigned or not bugs.

#### Bugs which are NOT assigned

To find all the DevTools bugs without an assignee, we use the `assigned_to` field and search for the default value, `nobody@mozilla.org`:

https://bugzilla.mozilla.org/buglist.cgi?product=DevTools&assigned_to=nobody@mozilla.org

#### Bugs that ARE assigned

But to search for bugs where the assignee is *not* the default, we can only use the `NOT` (`!` or negation) operator on the `quicksearch` parameter, which involves building an slightly different query:

https://bugzilla.mozilla.org/buglist.cgi?product=DevTools&quicksearch=assignee%21%3Dnobody%40mozilla.org

Note the exclamation mark to say: find bugs where `assignee` is `!nobody@mozilla.org` i.e. NOT `nobody@mozilla.org`.

Bugzilla seems to be doing some internal field conversion when using quicksearch, because when you use the `REST` or `CSV` links at the bottom of the page, you will see it generates three parameters for the assignee field, to create a `NOT` query. For example, this REST URL:

https://bugzilla.mozilla.org/rest/bug?include_fields=id,summary,status&bug_status=UNCONFIRMED&bug_status=NEW&bug_status=ASSIGNED&bug_status=REOPENED&field0-0-0=assigned_to&product=DevTools&type0-0-0=notequals&value0-0-0=nobody%40mozilla.org

## Importing data into other systems

When importing data it can be very useful to restrict the *amount* of data you get---maybe you only care about some columns, or you want additional columns other than the default ones.
If you don't specify which fields you want, it will return the defaults: id, status, summary.

You can specify which fields to include using the `include_fields` parameter, using comma separated values. For example, if you also want to show the priority you'd use: `include_fields=id,status,summary,priority`.

### JSON

The responses have a very permissive CORS header, so you can import Bugzilla searches into your websites.

You can build a REST query URL by replacing `https://bugzilla.mozilla.org/buglist.cgi?` with `https://bugzilla.mozilla.org/rest/bug?` in your existing search.

Please note there's a rate limit and if you do too many requests per minute you will be banned temporarily.

### CSV

Useful for importing live data into Google Spreadsheets, for example.

Append `&ctype=csv&human=1` to your query URL.

For example, closed bugs for `General` in the last week, in CSV format:

https://bugzilla.mozilla.org/buglist.cgi?bug_status=RESOLVED&bug_status=VERIFIED&bug_status=CLOSED&chfield=resolution&chfieldfrom=-8d&chfieldto=Now&chfieldvalue=FIXED&product=DevTools&component=General&ctype=csv&human=1

Note: if you enter that URL in a browser, it will try to download the file instead of displaying it.

To import the data into Spreadsheets, enter this formula in the cell where you want to start seeing results:

```spreadsheets
=IMPORTDATA(<CSV URL>)
```

where `<CSV URL>` is the query that generates the search results in CSV.

It's advisable to do one import per sheet to avoid results overlapping each other.

Note there's a limit to the amount of times you can use IMPORTDATA in a given Spreadsheet. I think it's 20.
