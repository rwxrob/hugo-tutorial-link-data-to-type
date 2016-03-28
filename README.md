
*Note: this tutorial content is taken from the [Hugo
tutorial](//gohugo.io/tutorial/link-data-to-type) of
the same title*

# Linking Data to Hugo Types

For many the most powerful part of Hugo is its amazing support for
structured [data files](//gohugo.io/extras/datafiles). This feature allows
Hugo to truly replace sites dependent on low-volume databases and
gain all the advantages of hosting your site on a full CDN hosting
provider.

Many beginners, however, do not realize that to use data files
effectively still requires the minimal use of
[`content`](//gohugo.io/content/organization) files.

This tutorial suggests one way to minimize the use of `content`
files by linking your data files to Hugo [content
types](//gohugo.io/content/types).  We use
[substr](//gohugo.io/templates/functions#substr) on a lesser known—but
very useful—`.File.LogicalName` template variable in the
`layouts/TYPE/single.html` template to link to a matching `.Site.Data`
object that has the same base name as the nearly-empty content file
(ex: `content/student/steve.md` <-> `data/student/steve.toml`).
This way you don't have to put anything in the front-matter of the
content file. Yeah, I thought I saw you smile.

## Why?

Often the easiest answer is just to not use the `data` directory
and put whatever was in your TOML/JSON/YAML files into the front
matter of your content documents. (If all of that sounds confusing
you might want to go back and [read about
it](//gohugo.io/content/front-matter).)

Just using the `content` directory has the advantage of keeping all
the content in one place but starts to fall down as the complexity
of the data on which your site is built increases—particularly if
you are sharing your site data externally or deriving it from other
data sources before Hugo even touches it. 

### Data-centric Over Content-centric

This data-centric (over content-centric) approach is becoming more
common as static sites continue to efficiently and effectively
replace the overly complex and expensive dynamic database + caching
server architectures out there. Indeed, this is the same reasoning
behind the increase in JSON Web and REST APIs in general.

### Some Must Use .Site.Data

As we've [read on the
forums](https://discuss.gohugo.io/t/pulling-a-data-file-based-on-filename/967),
some developers of complex Hugo sites, (which we likely will never
see because of their enterprise-ness), are actually forced to design
their Hugo site structure entirely from the `data` directory because
the precious, primary source of the data is behind an API or secure
data store that requires it to be "dumped" and validated before
Hugo builds it.

This not only requires the `data` directory be used but also a
structure to the data that can be easily validated. In a sense,
this data dump-and-validate step covers more robust data validation
constraints that are normally implemented in the database itself or
application business logic.

### Automated Data Views

You can probably see where this is going. Most data is read and
viewed way more often than it is written. Hugo workflows are being
combined with watchdogs and cron jobs that auto dump to `data`
triggering a live data validation process followed by a Hugo compile
and push deployment.

In this sense Hugo sites are just what traditional database people
would see as "views" of the data. This concept of "views" to organize
and simplify large amounts of data is a long-standing best-practice
and architecture. Hugo just happens to be perfect for the job.

## Stuff You Should Already Grok

You should have a firm grasp on Hugo [source
organization](//gohugo.io/overview/source-directory) and how [data
files](//gohugo.io/extras/datafiles) work. This tutorial uses the
beautiful [TOML](https://github.com/toml-lang/toml), which was
designed for structured data files that are maintained by humans.
If you are tying into other data sources you might want to use JSON,
but maybe not.

## Tutorial

Ok, let's get on with it. If you prefer you can [fork the
tutorial site](//github.com/skilstak/hugo-tutorial-link-data-to-type)
we are making.

### Create the Empty Content Files

In this example, we'll create a few logical `person`s in different
roles: `student`, `admin`, `creator`, and `user`. Start by making some
nearly empty `content/person/` files:

* `data/person/spf13.md`
* `data/person/bep.md`
* `data/person/robmuh.md`
* `data/person/benmuh.md`

All you need in each one is the empty front-matter section:

```markdown
+++
+++
```

You can obviously make these with a simple script:

```bash
for i in spf13 bep robm benmuh;do printf "+++\n+++\n" > $i.md; done
```

Although content files can also end in `html` we stick with `.md`
so we have a consistent suffix we can chop off later with
[substr](//gohugo.io/templates/functions#substr).

That's it for the content for the individual `person` types. The rest
will come from the `data/person` files.

### Create the Data Files

Now we need the `data/person` TOML files to match:

* `data/person/spf13.toml`
* `data/person/bep.toml`
* `data/person/robmuh.toml`
* `data/person/benmuh.toml`

We'll have to do these by hand, as we would if building up a real
site. But thankfully this is the only place we need to remember to
make any changes. Here's one file as an example:

```toml
name = "Rob Muhlestein"
github = "robmuh"
roles = ["student","user"]
```

This is enough of a data file to illustrate the point. Make sure you
understand the [data file](/extras/data-files) concept fully to get
the most out of it. It is really amazing.

### Create a Single Person Partial

We use [partials](/templates/partials) instead of [content
views](/templates/views), (which require `.Render` because they are
almost always preferred for their flexibility by allowing the passing
of context, which `.Render` infers instead causing it to fail when
using `.Site.Data` and not `.Site.Pages`).

First create the partials directory to keep things organized:

```bash
mkdir `partials/person`
```

Now create a partial that just contains the markup for the default
person view. We will pull this in from the content layout
`single.html` next. So `partials/person/block.html` would contain:

```html
<ul>
  <li>Name: {{ .name }}</li>
  <li>GitHub: <a href="//github.com/{{ .github }}">{{ .github }}</a>
  <li>Roles: {{ delimit .roles ", " }}</a>
</ul>
```

Nothing fancy really just to make the point. Notice how clean all
the references are.

### Create the Single Content Type View

Here's where the magic happens and we link it all together.  Now
is a good time to make sure you understand what a [Hugo Content
Type](//gohugo.io/content/types) is.

First, get into the `layouts/person` directory. If you haven't made
one yet, make it. Now add something like the following template
HTML to the `single.html` file:

```html
{{ with (index .Site.Data.person (substr $.File.LogicalName 0 -3)) }}
<!doctype html>
<html>
  <head>
    <title>{{ .name }}</title>
    <link rel="stylesheet" href="/styles.css">
  </head>
  <body>
    {{ partial "person/block" . }}
  </body>
</html>
{{ end }}
```

There are obviously lots of variations of this but you get the idea.
The first line is the key. This line looks up the `.Site.Data.person`
logical data object and matches the base file name (aka
`.File.LogicalName`) of the content file being used to generate the
HTML that will be served up from `/person/robmu/` for example. If
everything is set you should be able to run `hugo server` and pull
it up at `http://localhost:1313/person/robmuh`. Keep in mind that
`http://localhost:1313/person` will not yet work. We'll do that
next.

## Viewing Collections

Hugo calls these `lists` and they refer to any time you want to
view a single web page that has all of the items of a certain type
together, either at once or paginated, (which we won't get into).

There are [many places Hugo will look](//gohugo.io/templates/list/)
for these special collection view templates, which originally were
only known as `lists` but have come to be called `sections` as well,
although there are connotations to each. In our example we will use
only the `layout/section` directory since this is the only one
supported by themes (and yes you really do want to consider starting
with a theme in the first place even though we don't here).

### Create the List Item Partial

Before we create the `layout/section` page let's make a
`layouts/partials/person/li.html` that we can easily use in it:

```html
<li><a href="/{{ .github }}/">{{ .name }}</a></li>

```

That's it. Short and sweet.

### 



### Turn On the Collection View 

Like everything else in Hugo, nothing gets built without something
in the `content` directory. In the case of our collection view all
we need directory that matches the name with a single empty file
in it that we'll call `on.md`:




Sometimes setting `pluralizelisttitles` to `false` isn't enough and
you want to control the entire look of your collection view.

## Students Only

To isolate our larger data set of `person`s we add the following:







## Use Data Views with Partials Instead of Content Views

[Content views](http://gohugo.io/templates/functions/#content-views),
which require `.Render` to have a context only work with content
types. Partials, which accept a context as the first argument,  work
with anything and are therefore much easier to work with in general
and essential when creating what we'll call *data views* in contrast.
Instead of a `/layouts/student/li.html` we have a
`layouts/partials/student/li.html` and follow that convention when
adding different views of our data. Many agree this is better anyway
because of the flexibility it affords.

