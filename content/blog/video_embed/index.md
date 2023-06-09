---
title: "Embedding Videos in R Markdown"
author: "Chris Barnett"
date: 2022-03-14
output:
  html_document:
    toc: yes
    toc_float: yes
---

<link href="/rmarkdown-libs/vembedr/css/vembedr.css" rel="stylesheet" />

<style type="text/css">
h1.title { /* Header 4 - and the author and date headers use this too  */
  font-size: 40px;
  font-style: normal;
  font-weight: bold;
  font-family: Tahoma, Verdana, sans-serif;
  color: Black;
}

h4.author { /* Header 4 - and the author and date headers use this too  */
  font-size: 20px;
  font-style: normal;
  font-family: Tahoma, Verdana, sans-serif;
  color: Black;

}
h4.date { /* Header 4 - and the author and date headers use this too  */
  font-size: 20px;
  font-family: Tahoma, Verdana, sans-serif;
  color: Black;
}

h2 {/* Header 2 */
  font-size: 24px;
  font-family: Tahoma, Verdana, sans-serif;
  color: Black;
  background-color: WhiteSmoke;
  text-indent: 5px;
}
</style>

------------------------------------------------------------------------

## R Markdown with video

Video can be included in your markdown document tutorial text. This can be a useful tool that allows you incorporate dynamic content within your markdown documents. [YouTube](https://www.youtube.com/watch?v=lJIrF4YjHfQ) is an obvious online platform with an extensive amount of video content. Here is a video example of how to embed videos from YouTube.

<iframe width="560" height="315" src="https://www.youtube.com/embed/lJIrF4YjHfQ" title="New Title" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen data-external="1">
</iframe>

The way this is achieved this was by copying the *embed* HTML code directly from YouTube. This code specifies an [iframe](https://www.w3schools.com/html/html_iframe.asp), which allows you to display a webpage within another webpage.

![](https://thumbs.gfycat.com/BlaringConcernedKid-size_restricted.gif)

Unfortunately, if you follow YouTube’s instructions exactly, the R Markdown fails to embed your video. This seems to be an issue with [Pandoc](https://pandoc.org/) (the free document converter) that R Markdown uses to convert your YAML and R code into HTML and CSS.

Let see what happens if you paste the HTML embed code from YouTube directly into your R Markdown document

``` css
<iframe width="560" height="315" src="https://www.youtube.com/embed/sxYE0BY1mdc" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen>
```

The code above doesn’t work because it is missing the **data-external=“1”** HTML code that isn’t included in the YouTube embed code. Add it to the end of the copied HTML code to embed the video. Hopefully, RStudio will update the verision of Pandoc at some point, but if you find yourself unable to paste HTML code directly from YouTube, or another video platform, try adding the **data-external=“1”** at the end of your code as in the example below.

``` css
<iframe width="560" height="315" src="https://www.youtube.com/embed/lJIrF4YjHfQ" title="New Title" frameborder="0" allow="accelerometer; autoplay; #clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen data-external="1">
</iframe>
```

<style type="text/css">
<iframe width="560" height="315" src="https://www.youtube.com/embed/lJIrF4YjHfQ" title="New Title" frameborder="0" allow="accelerometer; autoplay; #clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen data-external="1">
</iframe>
</style>

However, as is often the case with R, there is another way to embed video in your Markdown documents that seems to get around our Pandoc issues. You can also use the "*vembedr" package* to easily embed videos in code chunks using their URL, or a file path on your machine.

``` r
library(vembedr)
vembedr::embed_url("https://www.youtube.com/watch?v=lJIrF4YjHfQ")
```

<div class="vembedr">
<div>
<iframe src="https://www.youtube.com/embed/lJIrF4YjHfQ" width="533" height="300" frameborder="0" allowfullscreen="" data-external="1"></iframe>
</div>
</div>

It’s that easy!