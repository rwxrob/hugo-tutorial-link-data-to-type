
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
types](//gohugo.io/content/types).  We use substring of the less
known `.File.LogicalName` template variable in the
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
and more common as static sites continue to efficiently and effectively
replace the overly complex and expensive dynamic database + caching
server architectures out there. Indeed, this is the same reasoning
behind the increase in JSON Web and REST APIs in general.

### Some Must Use .Site.Data

As we've read on the forums, many developers of complex Hugo sites,
(which we likely will never see because of their enterprise-ness),
are actually forced to design their Hugo site structure entirely
from the `data` directory because the precious, primary source of
the data is behind an API or secure data store that requires it to
be "dumped" and validated before Hugo builds it. 

This not only requires the `data` directory be used but also a
structure to the data that can be easily validated. In a sense,
this data dump-and-validate step covers more robust data validation
constraints that are normally implemented in the database itself or
application business logic.

### Automated Data Views

You can probably see where this is going. Most data is read and
viewed way more often than it is written. Workflows are even emerging
with watchdogs and cron jobs that auto dump to `data` triggering a
live data validation process followed by a Hugo compile and push
deployment. In this sense Hugo sites are just what traditional
database people would see as "views" of the data. This concept of
"views" to organize and simplify large amounts of data is a
long-standing best-practice and architecture. Hugo just happens to
be perfect for the job.

## What You Already Need to Know

You should have a firm grasp on Hugo [source
organization](//gohugo.io/overview/source-directory) and how [data
files](//gohugo.io/extras/datafiles) work. This tutorial uses
[TOML](https://github.com/toml-lang/toml), which was designed for
structured data files that are maintained by humans.

## Overview

Ok, let's get on with it &hellip;

In this example, we'll use a logical `person` in the role of a
`student`. In other words, we'll have a `data/person/demo.toml`
file and display it as a singular web page `/student/demo`. We'll
also add a `/layouts/section/students.html` template to list all
the students, who are just `person` data objects that have the
`student` role.

## Use `layouts/section/student.html` for Collection (List) View

Hugo comes with a built in method for displaying collections of
content types, which Hugo docs refer to as "lists". While this is very
convenient for simple sites, it is insufficient for a site driven by
the `data` directory like the one we are building. So we make our own
section (list) view in `layouts/section/student.html` that will be
used when users put only `/student/` into the browser. We'll talk
about how to make `/students/` just work as well later.

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

