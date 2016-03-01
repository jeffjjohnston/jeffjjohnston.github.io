---
layout: post
title:  "Faster rendering in RStudio"
date:   2016-03-01 09:00:00 -0600
categories: RStudio RMarkdown
---

Nearly all of the analysis I do in R is in the form of RMarkdown documents. I tend to build these documents iteratively, adding or modifying code chunks progressively and rendering the document to HTML to see the results. For some time I accomplished this with three applications: a text editor (I use [TextMate][textmate]), a Terminal window running R, and a Safari window. I created two custom commands and bound them to shortcut keys in TextMate: **Command-K** (for Knit), which sends `rmarkdown::render("active_document.Rmd")` to the R Terminal window, and **Command-P**, which opens `active_document.html` in Safari. This allows me to quickly render my document and view the resulting HTML.

Although this setup generally works well for me, I've been wanting to move my workflow over to [RStudio][rstudio] ever since I saw its impressive (and highly useful) [code diagnostics][rstudio-codediags] feature. One immediate issue I noticed when I tried using RStudio exclusively is that rendering is quite a bit slower. Most of my analysis documents load a handful of [Bioconductor][bioc] libraries, and this can take many seconds on a fast workstation. With my previous workflow, rendering in the same R console session is quite fast as it avoids reloading the packages on each render. However, when you press **Knit** in RStudio, it launches a new R process in the background and renders your document, requiring all libraries to be loaded fresh. This turned out to significantly slow me down when making minor adjustments to a document and frequently re-rendering it.

With RStudio's [new addin support][rstudio-addins], I decided to build a simple addin for rendering the current RMarkdown document in the console session. The addin, [RStudioConsoleRender][rstudioconsolerender], provides a single RStudio addin command called **Render in console**. Executing it will call `rmarkdown::render(active_document_path, envir=.GlobalEnv)` in the RStudio console and then launch a viewer for the resulting rendered document. The effect is very similar to the **Code** \| **Run region** \| **Run all** command, except that it also renders your output document. Unlike the **Knit** button, however, it does not create a fresh environment. This has some important consequences to keep in mind.

First, you get to avoid reloading packages on each render. This was obviously my primary motivation for building the addin.

Second, because rendering occurs in the console's environment, it is easy to introduce mistakes that you won't catch until you render in an empty environment via the **Knit** command. For example, if you load a package in your console environment but forget to load it in your RMarkdown document, the package will be available when you render via the console but not when you use the **Knit** command. I find that it is a good idea to periodically use the **Knit** command to make sure my RMarkdown document does not have any dependencies on the state of the console environment.

With this simple addin, I can recreate my previous preferred workflow while taking advantage of RStudio's code diagnostics.

[textmate]: https://macromates.com
[rstudio]: https://www.rstudio.com/products/RStudio
[rstudio-codediags]: https://support.rstudio.com/hc/en-us/articles/205753617-Code-Diagnostics
[bioc]: http://bioconductor.org
[rstudio-addins]: https://rstudio.github.io/rstudioaddins
[rstudioconsolerender]: https://github.com/jeffjjohnston/RStudioConsoleRender
