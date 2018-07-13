---
layout: post
title: Jekyll Date Formatting Examples
tagline: ""
tags: ["jekyll", "date"]
keywords: jekyll
ref: jekyll-date-format
lang: en
---

[Original link](http://alanwsmith.com/jekyll-liquid-date-formatting-examples )

![jekyll-logo](http://alanwsmith.com/image-graphics/jekyll-logo-180x100.png)
## [Summary](#summary)
These examples provide tested code snippets for displaying several date formats on a Jekyll site1<sup>[1](#footnote1)</sup>. They should also work on GitHub Pages, Shopify or anything else that uses Liquid. An alternate title for this post should be:

_Everything you wanted to know about Jekyll date formatting but were afraid to ask._


## [Overview](#overview)
Jekyll<sup>[2](#footnot2)</sup> (the simple, blog aware, static website generator) uses Shopify's Liquid Template Engine<sup>[3](#footnote3)</sup>. Displaying dates is done using the ``{{ page.date }}`` supplied Liquid tag<sup>[4](#footnote4)</sup>. With no other alteration, the dates produced look something like:
```
2013-11-29 00:00:00 +0000
```
If there are designs that use that format, they are few and far between. Creating friendlier looking dates is done by applying Liquid's ``"date:"`` filter. For example, the tag/filter combination:
```
{{ page.date | date: '%B %d, %Y' }}
```
produces more reader friendly dates like:
```
November 29, 2013
```
Much better, but depending on the date, subtle design issues show up. For example, during the first nine days of each month "leading zeros" crop up (e.g. "August 03, 2013" instead of "August 3, 2013"). Other gotchas with the basic Liquid filters include:
1. Adding a period behind the abbreviated month names has to be adjusted to handle May. For example, "Aug. 16, 2013" is fine. "May. 16, 2013" is not.
2. September is generally abbreviated "Sept." instead of Liquid's default "Sep."
3. To comply with the AP style guide the months April, May, June and July should not be abbreviated. Similar alterations are necessary to meet with the guidelines from the Chicago Manual of Style.

Most designs go with the available defaults. Either using a format that doesn't have these issues or, more frequently, letting the details slip. The information still gets across and every web site has a punch list of potential modifications that stretches to the horizon. So, I understand putting off finding a solution, but not having proper date formatting has always bugged me. While solving the issue for myself<sup>[5](#footnote5)</sup>, I decided to put together this post as a public reference as well. I don't yet have the Ruby chops to contribute directly to Jekyll, but I can provide this reference to give back a little at least.

This set of Liquid date filters solves the issues listed above and explores a few other formatting options. Each one provides a solution for a specific display format and is provided with four output examples for the following date: 
1. May 3, 2013, 
2. July 4, 2013, 
3. September 23, 2013 
4. November 26, 2013. 

These examples demonstrate if/how the various formatting issues are handled. After the examples, a few snippets of code for individual elements are provided. With these samples, just about any date format desired should be within easy reach.

## [Built-in Jekyll Date Filters](#built-in-filters)
- Date to String
```
{{ page.date | date_to_string }}
```
Output Example 1: **03 May 2013** 
Output Example 2: **04 Jul 2013** 
Output Example 3: **23 Sep 2013**
Output Example 4: **26 Nov 2013**

- Date to Long String
```
{{ page.date | date_to_long_string }}
```
Output Example 1: **03 May 2013**
Output Example 2: **04 July 2013**
Output Example 3: **23 September 2013**
Output Example 4: **26 November 2013**

- Date to XML Schema
```
{{ page.date | date_to_xmlschema }}
```
Output Example 1: **2013-05-03T13:14:00+00:00**
Output Example 2: **2013-07-04T13:14:00+00:00**
Output Example 3: **2013-09-23T13:14:00+00:00**
Output Example 4: **2013-11-26T13:14:00+00:00**

- Date to RFC-822
```
{{ page.date | date_to_rfc822 }}
```
Output Example 1: **Fri, 03 May 2013 13:14:00 +0000**
Output Example 2: **Thu, 04 Jul 2013 13:14:00 +0000**
Output Example 3: **Mon, 23 Sep 2013 13:14:00 +0000**
Output Example 4: **Tue, 26 Nov 2013 13:14:00 +0000**

## [Liquid Jekyll Date Formatting Examples](#liquid_examples)
- ISO 8601 Date<sup>[6](#footnote6)</sup>
```
{{ page.date | date: "%Y-%m-%d" }}
```
Output Example 1: 2013-05-03
Output Example 2: 2013-07-04
Output Example 3: 2013-09-23
Output Example 4: 2013-11-26

- U.S. Numeric Style with Four Digit Years (With leading zeros.)
```
{{ page.date | date: "%m/%d/%Y" }}
```
Output Example 1: 05/03/2013
Output Example 2: 07/04/2013
Output Example 3: 09/23/2013
Output Example 4: 11/26/2013

- U.S. Numeric Style with Four Digit Years (Leading zeros removed.)
```
{{ page.date | date: "%-m/%-d/%Y" }}
```
Output Example 1: 5/3/2013 
Output Example 2: 7/4/2013 
Output Example 3: 9/23/2013 
Output Example 4: 11/26/2013

- U.S. Numeric Style with Two Digit Year (Leading zeros removed.)
```
{{ page.date | date: "%-m/%-d/%y"}}
```
Output Example 1: 5/3/13 
Output Example 2: 7/4/13 
Output Example 3: 9/23/13 
Output Example 4: 11/26/13

- Outside U.S. Style with Full Month Name (Leading zeros removed.)
```
{{ page.date | date: "%-d %B %Y"}}
```
Output Example 1: 3 May 2013 
Output Example 2: 4 July 2013 
Output Example 3: 23 September 2013 
Output Example 4: 26 November 2013

- Outside U.S. Style with Non-English Full Month Name (Leading zeros removed.)
This example uses the German names for months. Any other language can be used by simply substituting the proper names for each month.
```
{% assign m = page.date | date: "%-m" %}
{{ page.date | date: "%-d" }}
{% case m %}
  {% when '1' %}Januar
  {% when '2' %}Februar
  {% when '3' %}März
  {% when '4' %}April
  {% when '5' %}Mai
  {% when '6' %}Juni
  {% when '7' %}Juli
  {% when '8' %}August
  {% when '9' %}September
  {% when '10' %}Oktober
  {% when '11' %}November
  {% when '12' %}Dezember
{% endcase %}
{{ page.date | date: "%Y" }}
```
Output Example 1: 3 Mai 2013 
Output Example 2: 4 Juli 2013 
Output Example 3: 23 September 2013 
Output Example 4: 26 November 2013

- U.S. Style with Full Month Name (Leading zeros removed.)
```
{{ page.date | date: "%B %-d, %Y" }}
```
Output Example 1: May 3, 2013 
Output Example 2: July 4, 2013 
Output Example 3: September 23, 2013 
Output Example 4: November 26, 2013

- U.S. Style with Full Month Names and Ordinalized Days (Leading zeros removed.)
```
{% assign d = page.date | date: "%-d"  %}
{{ page.date | date: "%B" }} 
{% case d %}
  {% when '1' or '21' or '31' %}{{ d }}st
  {% when '2' or '22' %}{{ d }}nd
  {% when '3' or '23' %}{{ d }}rd
  {% else %}{{ d }}th
  {% endcase %}, 
{{ page.date | date: "%Y" }}
```
Output Example 1: May 3rd, 2013 
Output Example 2: July 4th, 2013 
Output Example 3: September 23rd, 2013 
Output Example 4: November 26th, 2013

- U.S. Style with AP Month Abbreviations and Ordinalized Days (Leading zeros removed.)
```
{% assign d = page.date | date: "%-d" %} 
{% assign m = page.date | date: "%B" %} 
{% case m %}
  {% when 'April' or 'May' or 'June' or 'July' %}{{ m }}
  {% when 'September' %}Sept.
  {% else %}{{ page.date | date: "%b" }}.
  {% endcase %}
{% case d %}
  {% when '1' or '21' or '31' %}{{ d }}st
  {% when '2' or '22' %}{{ d }}nd
  {% when '3' or '23' %}{{ d }}rd
  {% else %}{{ d }}th
  {% endcase %}, 
{{ page.date | date: "%Y" }}
```
Output Example 1: May 3rd, 2013 
Output Example 2: July 4th, 2013 
Output Example 3: Sept. 23rd, 2013 
Output Example 4: Nov. 26th, 2013

- U.S. Style Full Day and Full Month Names (Leading zeros removed.)
```
{{ page.date | date: "%A, %B %-d, %Y" }}
```
Output Example 1: Friday, May 3, 2013 
Output Example 2: Thursday, July 4, 2013 
Output Example 3: Monday, September 23, 2013 
Output Example 4: Tuesday, November 26, 2013

- Chicago Manual of Style Day Abbreviations and U.S. Style Date (With "Thurs." and "Tues.")
```
{% assign dy = page.date | date: "%a" %}
{% case dy %}
  {% when "Tue" %}Tues
  {% when "Thu" %}Thurs
  {% else %}{{ dy }}
  {% endcase %}. 
~
{{ page.date | date: "%B" }} 
{{ page.date | date: "%-d" }}, 
{{ page.date | date: "%Y" }}
```
Output Example 1: Fri. ~ May 3, 2013 
Output Example 2: Thurs. ~ July 4, 2013 
Output Example 3: Mon. ~ September 23, 2013 
Output Example 4: Tues. ~ November 26, 2013


## [Individual Component Snippets for Liquid Date Formatting](#individual)
These individual snippets are for a few of the tricker formatting filters. Some that weren't used in the examples above. For those interested in the approach, the hack I'm using to remove leading zeros is to add "0" to the string. This turns the string into an integer. When the integer is rendered back as a string the leading zero disappears. Hurray for dynamic typing.

- Numeric Month with Leading Zeros Removed
```
{{ page.date | date: "%-m" }}
```

- Numeric Day with Leading Zeros Removed
```
{{ page.date | date: "%-d" }}
```

- Ordinalized Numeric Day with Leading Zeros Removed
```
{% assign d = page.date | date: "%-d" %}
{% case d %}
  {% when '1' or '21' or '31' %}{{ d }}st
  {% when '2' or '22' %}{{ d }}nd
  {% when '3' or '23' %}{{ d }}rd
  {% else %}{{ d }}th
  {% endcase %}
```

- AP Style Month Abbreviations
```
{% assign m = page.date | date: "%B" %}
{% case m %}
  {% when 'April' or 'May' or 'June' or 'July' %}{{ m }}
  {% when 'September' %}Sept.
  {% else %}{{ page.date | date: "%b" }}.
  {% endcase %}
```
(Produces: "Jan.", "Feb.", "Mar.", "April", "May", "June", "July", "Aug.", "Sept.", "Oct.", "Nov.", "Dec.")

- Chicago Manual of Style Day Abbreviations
```
{% assign dy = page.date | date: "%a" %}
{% case dy %}
  {% when "Tue" %}Tues
  {% when "Thu" %}Thurs
  {% else %}{{ dy }}
  {% endcase %}.
```
(Produces: "Sun.", "Mon.", "Tues.", "Wed.", "Thurs.", "Fri.", "Sat.")

- Chicago Manual of Style Month Abbreviations
```
{% assign m = page.date | date: "%B" %}
{% case m %}
  {% when 'May' or June' or 'July' %}{{ m }}
  {% when 'September' %}Sept.
  {% else %}{{ page.date | date: "%b" }}.
  {% endcase %}
```
(Produces: "Jan.", "Feb.", "Mar.", "Apr.", "May", "June", "July", "Aug.", "Sept.", "Oct.", "Nov.", "Dec.")

- Non-English Month Names
```
{% assign m = page.date | date: "%-m" %}
{% case m %}
  {% when '1' %}Januar
  {% when '2' %}Februar
  {% when '3' %}März
  {% when '4' %}April
  {% when '5' %}Mai
  {% when '6' %}Juni
  {% when '7' %}Juli
  {% when '8' %}August
  {% when '9' %}September
  {% when '10' %}Oktober
  {% when '11' %}November
  {% when '12' %}Dezember
{% endcase %}
```
(Produces: "Januar", "Febuar", "März", "April", "Mai", "Juni", "Juli", "August", "September", "Oktober", "November", "Dezember")

With that, you should be in pretty good shape. If you can't directly create what you need from the above samples or snippets you should at least be able to use a similar approach to piece together what you need.


## [Notes on the Examples](#note)
1. The hour, minute and second parts of the full date/time stamp aren't being used because Jekyll tends to zero them out.
2. In some of the examples, the code is split to multiple lines to help readability. If it's a natural break point where white space already exists, this won't effect HTML rendering. In some cases, it will introduce unwanted white space. Simply move everything back to one line to create the desired presentation.
3. Since the original version of this post went up, updates have been made to use the cleaner version of ``{{ page.date | date: "%-m" }}`` instead of ``{{ page.date | date: "%m" | plus:'0' }}`` and ``{{ page.date | date: "%-d" }}`` instead of ``{{ page.date | date: "%d" | plus:'0' }}`` to remove leading zeros.
4. An additional update was made to add an example for non-English month names.


## [Footnotes](#footnote)
1. <a name="footnote1"> </a> These examples were create on a Mac running OS X 10.8.5 with: Ruby 2.0.0p247 - Jekyll 1.2.1 - liquid 2.5.2. Your mileage may vary.
2. <a name="footnote2"> </a> The main [Jekyll website](http://jekyllrb.com/ ) provides a great overview of the software. You can learn even more with a visit to the [Jekyll's GitHub Repo](https://github.com/mojombo/jekyll ).
3. <a name="footnote3"> </a> [Shopify's Liquid template engine](http://wiki.shopify.com/Liquid ). "A small and fast template language which is quick to learn but very powerful for full customization."
4. <a name="footnote4"> </a> The Liquid Date Filter [offers some basic formatting elements](http://wiki.shopify.com/Date ) and is the basis of these code snippets. Note that in some cases "post.date" might be required instead of "page.date".
5. <a name="footnote5"> </a> Observant readers will notice that as of the time this post went live, my design still uses a date format with leading zeros. I have the solution, it just hasn't been implement yet. It'll go in with the next set of design changes.
6. <a name="footnote6"> </a> [ISO 8601](http://en.wikipedia.org/wiki/ISO_8601 ) - "Data elements and interchange formats - Information interchange - Representation of dates and times is an international standard covering the exchange of date and time-related data." A perfect example of how Time really is [wibbly wobbly timey wimey… stuff](http://www.youtube.com/watch?v=vY_Ry8J_jdw ).

