---
description: |
  Use hugo shortcodes to quickly compose site pages.
title: Shortcodes
weight: 5
---


Why shortcodes? While markdown is sufficient to produce simple pages, it's insufficient where complex page structures are needed. Thusly, whenever we need special styling, shortcodes compliment the shortcomings of markdown.

This way, you can side step complex html and css boilerplate in your content files. 

Sometimes, the shortcode will wrap content, sometimes it won't. When content is wrapped, a closing shortcode tag is needed. Please see the link I provided above and the markdown files for examples. You'll get the gist pretty quickly.

I've setup the following shortcodes:

### Block 

Takes positional modifiers

**Example**
```markdown
...
  {{</* block "modifiers" */>}}
  <!-- Nest columns or content  -->
  {{</* /block */>}}
...
```

### Column

It takes positional parameters

**Example**
```markdown
  {{</* column "mt-2 mb-2" */>}}
  <!-- applied margin top and margin bottom modifiers -->
  {{</* /column */>}}
```

### Video

This allows you to embed a youtube video in you content. You would achieve that using a positional parameter (needs no name )parameter, like so:

**Example:**

```markdown
  {{</* youtube "xWF59rWSceA" */>}}
  <!-- Use the youtube video id -->
```

### Button

This adds a styled link (styled like a button). It takes two no-optional parameters:

| PARAMETER | PURPOSE | OPTIONAL |
| :---  | :--- | :--- |
| label | button text | no |
| url | button link | no |

**Example**

```
  {{</* button url="/" label="doe nu mee" */>}}
```

### Picture

You want to use darkmode images when darkmode is enabled on a device and a regular image on lightmode? It takes 3 positional parameter

Store these images in the `static/images` directory. 

```
...
{{</* picture "lightModeImage.png" "darkModeImage.png" "Image alt text" */>}}
...
```
