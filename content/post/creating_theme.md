+++
date = "2018-02-08T19:32:52-06:00"
draft = false
title = "Creating a New ggplot theme"
tags = ["R"]

+++

# Introduction
This post introduces simple steps on how to create your own ggplot2 theme. Of course, it will not go into details of changing all the aspects. Instead, it will highlight the process using few useful elements.

Theme is basically a set of pre-defined(default) values for elements that make up a plot in ggplot2 library. Major elements are:

- Plot, the outermost area used by a plot. Think of this as let's say your A4 sheet on which you want to draw a plot.
- Panel, this is the plotting area. This is defined by a rectangle inside the panel.
- Legend, another rectangle defining the plot legend. This can be anywhere in the panel, inside or outside of the plot.
- x-axis
- y-axis
- Strip, I don't know what this is, yet!

## Default Theme
Instead of rewriting properties of all elements, it is more efficient to pick a theme that is closer to what you want and the just adjust it. In my case, I like the `theme_bw` so I will use that as a starting point. A scatter plot on this theme looks like this:


{{<highlight R>}}
theme_set(theme_bw())
ggplot(mpg, aes(displ, hwy)) + geom_point()
{{</highlight>}}

{{<figure src="../images/basetheme-1.png" width="70%" >}}

## New Theme
I like white background and black border of this theme. Few changes I would like to make to this theme are:

- remove gridlines. I personally think gridlines do not server much purpose while doing analysis and are rather a distraction.
- increase the base font size a bit to be more clear.
- note that the axis marker font is grey color 

To make these changes, we will simply 'add' these elements to our base theme:


{{<highlight R>}}
theme_simple <- function() {
    theme_bw(base_size = 14,
             base_family = "") %+replace%
    theme(panel.grid = element_blank(),
          axis.text = element_text(color = "black"))
}
{{</highlight>}}

Notice that I have created a new function for this theme instead of just storing it in an object. This makes it flexible since now it can be sourced in any project and used. Let's see how our plot looks now with the new theme.


{{<highlight R>}}
theme_set(theme_simple())
ggplot(mpg, aes(displ, hwy)) + geom_point()
{{</highlight>}}

{{<figure src="../images/plot-1.png" width="70%" >}}

It looks more clear now bringing more attention to the data instead of theme or plot elements. You can find what else can be changed by `?theme`. 

