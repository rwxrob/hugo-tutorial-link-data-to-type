
*Note: this tutorial content is taken from the [Hugo
tutorial](https://gohugo.io/tutorial/link-data-to-type/) of
the same title*

# Linking Data to Hugo Types

For many the most powerful part of Hugo is its amazing support for
structured [data files](https://gohugo.io/extras/datafiles/). This feature allows
Hugo to truly replace sites dependent on low-volume databases and
gain all the advantages of hosting your site on a full CDN hosting
provider.

Many beginners, however, do not realize that to use data files
effectively still requires the minimal use of
[`content`](https://gohugo.io/content/organization/) files.

This tutorial suggests one way to minimize the use of `content`
files by linking your data files to Hugo [content
types](https://gohugo.io/content/types/).  We use
[substr](https://gohugo.io/templates/functions#substr/) on a lesser known—but
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
it](https://gohugo.io/content/front-matter/).)

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
organization](https://gohugo.io/overview/source-directory/) and how [data
files](https://gohugo.io/extras/datafiles/) work. This tutorial uses the
beautiful [TOML](https://github.com/toml-lang/toml), which was
designed for structured data files that are maintained by humans.
If you are tying into other data sources you might want to use JSON,
but maybe not.

## Tutorial

Ok, let's get on with it. If you prefer you can [fork the
tutorial site](https://github.com/skilstak/hugo-tutorial-link-data-to-type)
we are making.

### Create the Empty Content Files

In this example, we'll create a few logical `person`s in different
roles: `student`, `admin`, `creator`, and `user`. Start by making some
nearly empty `content/person/` files:

* `data/person/spf13.md`
* `data/person/bep.md`
* `data/person/robmuh.md`
* `data/person/betropper.md`

All you need in each one is the empty front-matter section:

```markdown
+++
+++
```

You can obviously make these with a simple script:

```bash
for i in spf13 bep robmuh betropper;do printf "+++\n+++\n" > $i.md; done
```

Although content files can also end in `.html` we stick with `.md`
so we have a consistent suffix we can chop off later with
[substr](https://gohugo.io/templates/function/#substr).

That's it for the content for the individual `person` types. The rest
will come from the `data/person` files.

### Create the Data Files

Now we need the `data/person` TOML files to match:

* `data/person/spf13.toml`
* `data/person/bep.toml`
* `data/person/robmuh.toml`
* `data/person/betropper.toml`

We'll have to do these by hand, as we would if building up a real
site. But thankfully this is the only place we need to remember to
make any changes. Here's one file as an example:

```toml
name = "Rob Muhlestein"
github = "robmuh"
roles = ["student","user"]
```

This is enough of a data file to illustrate the point. Make sure
you understand the [data file](https://gohugo.io/extras/datafiles/) concept
fully to get the most out of it. It is really amazing.

### Create a Person Block Partial

We use [partials](https://gohugo.io/templates/partials/) instead
of [content views](https://gohugo.io/templates/views/), which are
older than partials and require
[`.Render`](https://gohugo.io/templates/views/#rendering-view-inside-of-a-list).
These days partials are almost always preferred for their flexibility
by allowing the passing of context, (which `.Render` infers instead
causing it to fail when using `.Site.Data` and not `.Data.Pages`).

First create the partials directory to keep things organized:

```bash
mkdir layouts/partials/person
```

Now create a partial that just contains the markup for the default
person view. We will later pull this in from the content layout
`single.html`. We'll name our partial "block" to imply the use of
[BEM methodology](http://getbem.com/introduction/) So
`layouts/partials/person/block.html` would contain:

```html
<ul>
  <li>Name: {{ .name }}</li>
  <li>GitHub: <a href="https://github.com/{{ .github }}">{{ .github }}</a>
  <li>Roles: {{ delimit .roles ", " }}</a>
</ul>
```

Nothing fancy really just to make the point. Notice how clean all
the references are.

### Create the Single Content Type View

Here's where the magic happens and we link it all together.  Now
is a good time to make sure you understand what a [Hugo Content
Type](https://gohugo.io/content/types/) is.

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
The first line is the key, literally. This [`index` Go text template
function](https://golang.org/pkg/text/template/#hdr-Functions) looks up
the item in the `.Site.Data.person` map that has the index key
matching the base file name (aka `substr .File.LogicalName 0 -3`) of the content
file that triggers the generation of the HTML that will be served
up from `http://localhost:1313/person/robmuh` for example. 

Notice `substr` chops the last three characters off of `.File.LogicalName`,
which is why we name all our `content/person` files with `.md` at
the end.

### Try It Out

If everything is set you should be able to run `hugo server` and
pull it up at `http://localhost:1313/person/robmuh`. Keep in mind
that `http://localhost:1313/person` will not yet work. We'll do
that next.

## Viewing Collections

Hugo calls collections `lists` and there are [many places Hugo will
look](https://gohugo.io/templates/list/) for these special collection
view templates, which originally were only known as `lists` but
have come to be called `sections` as well. Sections seems to be the
preferred name now since themes only support a
`themes/THEME/layout/section` directory. *(While we are not using
themes in this tutorial you really should start with a theme because
it doesn't hurt.)*

### Create the List Item Partial

Before we create the `layout/section` page let's make a
`layouts/partials/person/li.html` that we can easily use in it:

```html
<li><a href="/person/{{ .github }}/">{{ .name }}</a></li>

```

That's it. Short and sweet. Notice we linked to a relative link
matching the `.github` user name. We'll use `github` as our unique
identifier for "persons", but that sounds weird so let's fix it with
a permalink and configuration addition.

### Add Person Permalinks

This is where we run into a limitation in the theme system somewhat.
There is something of a debate about whether permalinks should be able
to be dictated by a theme or not. Currently the theme does not have
this ability so you need to add them to your `config.toml` file. While
we are at it let's add a `.Site.Title`:

```toml
title = "Testing Hugo Data Linked To Content Type"
[permalinks]
person = "/:filename"
```

Since a `person` is a fundamental data type we don't have a problem
with permalinking its unique identifier.

Now you can remove the `person` portion of the link in the `li.html`
partial view:

```html
<li><a href="/{{ .github }}/">{{ .name }}</a></li>
```

### Add in Some CSS

Throw the following into your `/static/styles.css` if you like:

```css
body {
  margin: 5rem;
  font-family: "Helvetica", "Arial", sans-serif;
  background: lightgrey;
}

li {
  padding .2rem;
  font-weight: bold;
  list-style-type: none;
}

a {
  text-decoration: none;
  color: grey;
}

a:hover {
  color: goldenrod;
}
```

Ah, much better, well at least a little.


## Adding Another Collection View

The logical idea of a `person` looks good in UML and data design
diagrams but no one really uses it. What they really want is to see
what role that person has in the organization. In our tutorial there
are `admins`, `creators`, `users`, and `students`, each potentially
with extra data related to that role inside their individual
`data/person` files. But how do we create collection views for these
different roles? This is where [content list
templates](https://gohugo.io/templates/list/), which some prefer to call
section templates, come in.

### Turn On the Collection Section View 

Like everything else in Hugo, nothing gets built without something
being in the `content` directory. In the case of our collection
section view all we need is a directory that matches the name of
the section we want containing a single empty file that we'll call
`on.md`:

```bash
mkdir content/students
touch content/students/on.md
```

Notice we named our section students instead of student. Try
`http://localhost:1313/person` in comparison. Notice the title is
"persons". This is a case for setting `pluralizelisttitles` to
`false` in your `config.toml`. If this isn't enough you can add
another `http://localhost:1313/people` view using the same steps as
those we are doing for `students` now.

### Create the Students Section View

This is the same as earlier just with a logical filter to only include
those `person` data objects that have `student` in their roles. To
save time just copy the `layout/sections/person.html` file to
`layout/sections/students.html`, which will look like this when you
are done with it:

```html
<!doctype html>
<html>
  <head>
    <title>Students</title>
    <link rel="stylesheet" href="/styles.css">
  </head>
  <body>
    <h1>Students</h1>
    <ul>
      {{ range .Site.Data.person }}
        {{ if in .roles "student" }}{{ partial "person/li" . }}{{ end }}
      {{ end }}
    </ul>
  </body>
</html>
```

Obviously a lot of this should be factored into partials such as a
`top` and `bottom` so that other views can be quickly added but
this conveys the idea.

***Trade Offs:*** Unfortunately the [grouping
functions](https://gohugo.io/templates/list/#grouping-content) you
may have become accustomed to when dealing with `.Data.Pages` simply
don't work with `.Site.Data` and probably never will. But we really
don't need them because we can use the standard [`range
sort`](https://gohugo.io/templates/functions/#sort), which is
probably a better habit and dependency anyway.

### Experiment

Now that we've created the `students` data view (or section) you can
experiment with tweaking everything to add views for `users`,
`creators`, `admins`, and `people`.

## Conclusion

At this point you should have all the boilerplate you need to get
a solid, data-driven Hugo site up and running. The data model is
up to you. Perhaps you have it already polished in UML, perhaps
not.  But this approach affords the most flexibility and efficiency
when implementing changes to the most important part of many complex
sites, the data.

