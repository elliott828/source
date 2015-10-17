title: R Package ggvis Flashback
date: 2015-10-07 20:56:41
categories: Codage
tags: [R, ggvis, visualization, dplyr, ggplot2]
---

Why ggvis?
----------

`ggvis` is an awesome data visualization package which builds data graphics with a syntax similar to `ggplot2` and creates rich interactive plots like `shiny`. Since the syntax is very structural, it's easy to learn and to use.

<!-- more -->

``` r
require(ggvis, quietly = T)
require(dplyr, quietly = T)
```

Basic ggvis Syntax
------------------

### Basic Components

`ggvis` recreates the grammar of graphics. The key syntax is like this:

``` r
data                                                                %>%
  ggvis(~<x axis var1>, ~<y axisvar2>, fill = ~<fill property>)     %>%
  layer_<marks>()
```

You could find 4 components from the chunk above:

$$ Graphic = Data + CoordinateSystem + Properties + Marks $$

For example, using built-in dataset `mtcars`:

``` r
mtcars                                                              %>%
  ggvis(~hp, ~mpg, fill := "blue")                                  %>%
  layer_points()
```
<<<<<<< HEAD
![img1](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot1.png)
=======
![img1](/img/codage8-plot1.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

Noticed that the coordinates, properties can be moved to the `layer_<marks>()`, `ggvis()` can generates plot without `layer_<marks>()`. Those will all be concretely introduced in the following part.

### Global vs. Local Declaration & Multiple Layers

`ggvis` allows multiple layers overlaid. When you put the coordinates and properties in `ggvis()`, you declare them globally. That means the coordinates and properties will be used commonly in all the following `layer_<marks>()`

``` r
mtcars                                                              %>%
  ggvis(~hp, ~mpg, stroke := "blue")                                %>%
  layer_points()                                                    %>%
  layer_smooths()
```
<<<<<<< HEAD
![img2](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot2.png)
=======
![img2](/img/codage8-plot2.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

We specify `~hp, ~mpg, stroke := "blue"` in `ggvis()`, they are applied on all the layers: respectively on `layer_points()` and `layer_smooths()` for the color of border of the points and the color of the smooth line. By default the `fill` color of points is black.

When we do that locally, we put the properties in each layer:

``` r
mtcars                                                              %>%
  ggvis(~hp, ~mpg)                                                  %>%
  layer_points(fill := "white", stroke := "blue")                   %>%
  layer_smooths(stroke := "red")                                    %>%
  layer_model_predictions(model = "lm", stroke := "navy")
```
<<<<<<< HEAD
![img3](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot3.png)
=======
![img3](/img/codage8-plot3.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

The properties are declared locally and both layers use `stroke` but the property works sperately in 2 layers and `fill` has no impact on `layer_smooths()`. Why we keep `~hp, ~mpg` in `ggvis()`? Because the program doesn't have to run them twice in each layer. Keeping them in `ggvis()` makes it more efficient.

### Assignment Symbols `=` & `:=`

The most important symbols are `=` and `:=`. You can note them as "mapping" and "setting".

There are 2 spaces when plotting something - a data space and a visualization space. For example the color have HTML color codes, RGB color codes, etc. If you provide a variable to specify the `fill` using `=` (normally followed by a tilde `~`, making `ggvis` to treat it as a variable), `ggvis` will **mapping** the variable value on color scales first before plotting.

``` r
pressure                                                            %>%
  ggvis(~temperature, ~pressure, 
        fill = ~pressure, size = ~temperature)                      %>%
  layer_points()
```
<<<<<<< HEAD
![img4](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot4.png)
=======
![img4](/img/codage8-plot4.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

If you directly pass a string with quotation mark to it, `ggvis` read it as a raw value.

``` r
pressure                                                            %>%
  ggvis(~temperature, ~pressure, 
        fill := "skyblue", size := 80)                              %>%
  layer_points()
```
<<<<<<< HEAD
![img5](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot5.png)
=======
![img5](/img/codage8-plot5.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

*settings vs. mapping only works for a property instead of a parameter.*
You could directly use `=` + values for a parameter.

Besides, `%>%` is based on package `magrittr` and is used widely in `dplyr`. It's a symbol of chaining and makes the program more readable.

Layers & Properties
-------------------

If you don't specify layer type, `ggvis` will use `layer_guess()` to give an approximate estimation (`?ggvis::layer_guess`). Besides the magic `layer_guess()`, I'd strongly recommand to learn more specified layers. We just show you 2 kinds of properties in previous. The basic layers and properties are as below (column names are `layer_<marks>` functions, row names are properties):

| layer_        | bars | boxplots | densities | freqpolys | histograms | lines | paths | points |
|---------------|------|----------|-----------|-----------|------------|-------|-------|--------|
| x / x2        | O    | O        | O         | O         | O          | O     | O     | O      |
| y / y2        | O    | O        | O         | O         | O          | O     | O     | O      |
| width         | O    | O        | X         | O         | O          | X     | X     | O      |
| opacity       | O    | O        | O         | O         | O          | O     | O     | O      |
| fill          | O    | O        | O         | O         | O          | O     | O     | O      |
| fillOpacity   | O    | O        | O         | O         | O          | O     | O     | O      |
| stroke        | O    | O        | O         | O         | O          | O     | O     | O      |
| strokeWidth   | O    | O        | O         | O         | O          | O     | O     | O      |
| strokeOpacity | O    | O        | O         | O         | O          | O     | O     | O      |
| size          | X    | O        | X         | X         | X          | X     | X     | O      |
| shape         | X    | X        | X         | X         | X          | X     | X     | O      |

Where `O` means supported, `X` means not supported.

### Barchart, Histogram & Frequency Polygon

For bar graphs of counts at each unique x value, in contrast to a histogram's bins along x ranges. Barchar and histogram both have `width` argument. However, the former one is used as column width in graphical space, the latter one to group the coutinuous data on x-axis.

``` r
# barchart
pressure                                                          %>% 
  ggvis(~temperature, ~pressure, fill := "red", opacity := .4)    %>% 
  layer_bars(width = 15)
```
<<<<<<< HEAD
![img6](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot6.png)
=======
![img6](/img/codage8-plot6.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

``` r
# histogram
iris                                                              %>% 
  ggvis(~Sepal.Length, fill := "red", opacity := .7)              %>% 
  layer_histograms(width = 0.5)
```
<<<<<<< HEAD
![img7](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot7.png)
=======
![img7](/img/codage8-plot7.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

``` r
# freqpoly
iris                                                              %>% 
  ggvis(~Sepal.Length)                                            %>% 
  layer_freqpolys(width = 0.5, fill := "red", fillOpacity := .3,
                  strokeWidth := 3, stroke := "orange")
```
<<<<<<< HEAD
![img8](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot8.png)
=======
![img8](/img/codage8-plot8.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

Frequency polygon treats the continuous data in the same logic as histogram but use a line to describe the frequency evolution across ranges. Notice that I use `fillOpacity` instead of `opacity` in the third plot. That means the transparency effect is not applied on the stroke (not applied on every layer). By default there is nothing filled under the curve of frequency polygon, since we specify it, the region is filled by transparent red.

### Boxplot

`width` is also a parameter of `layer_boxplots()`, the default value is 0.9. This parameter specify the distance among groups / the width of boxes. Besides the normal properties `fill`, `stroke`, etc., you can assign the `size` of outliers.

``` r
iris                                                              %>% 
  ggvis(~Species, ~Sepal.Width)                                   %>% 
  layer_boxplots(size := 20, width = .7, strokeOpacity := .7,
                 strokeWidth := 2, fill = ~Species)
```
<<<<<<< HEAD
![img9](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot9.png)
=======
![img9](/img/codage8-plot9.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

*Currently the `layer_boxplots()` seems to have a bit problems under `ggvis` (version 0.4.2) when we modify the value of `size` - the mustach move but the boxes don't. Waiting for the package update.*

### Density Plot

Density plots provide another way to display the distribution of a single variable. A density plot uses a line to display the density of a variable at each point in its range. You can think of a density plot as a continuous version of a histogram with a different y scale (although this is not exactly accurate).

``` r
faithful                                                          %>% 
  ggvis(~waiting)                                                 %>%
  layer_densities(stroke := "red", fill := "red", area = FALSE)
```
<<<<<<< HEAD
![img10](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot10.png)
=======
![img10](/img/codage8-plot10.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

You can specify the `area` parameter to decide whether there should be a shaded region drawn under the curve. In the chunk above, even you assign "red" to `fill`, there is nothing under the density curve (the dault setting is to draw a grey shadow).

### Lines & Paths

Firstly we compare these 2 layers:

``` r
mtcars                                                            %>%
  ggvis(~wt, ~mpg)                                                %>%
  layer_points(size := 30, shape := "cross", fill := "grey")      %>%
  layer_lines(stroke := "red")                                    %>%
  layer_paths(stroke := "skyblue")
```
<<<<<<< HEAD
![img11](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot11.png)
=======
![img11](/img/codage8-plot11.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

It seems that the `layer_paths()` is chaos, but it is not. It plots starting from the very first record until the last. Let's reorder the dataset.

``` r
mtcars                                                            %>%
  arrange(wt, desc(mpg))                                          %>%
  ggvis(~wt, ~mpg)                                                %>%
  layer_lines(stroke := "red", strokeWidth := 4, 
              strokeOpacity := .5)                                %>%
  layer_paths(strokeWidth := 1)
```
<<<<<<< HEAD
![img12](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot12.png)
=======
![img12](/img/codage8-plot12.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

Now `layer_paths()` have same trend as `layer_lines()`. `layer_paths()` is more powerful on geographical plots.

``` r
library(maps)
texas <- ggplot2::map_data("state", region = "texas")
texas %>% ggvis(~long, ~lat) %>% layer_paths()
```
<<<<<<< HEAD
![img13](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot13.png)
=======
![img13](/img/codage8-plot13.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

lines and paths plot can also use `fill` property.

``` r
texas %>% ggvis(~long, ~lat) %>% layer_paths(fill := "darkorange")
```
<<<<<<< HEAD
![img14](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot14.png)
=======
![img14](/img/codage8-plot14.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

### Scatterplot

We've been talking about `layer_points()` for many times. To be noticed that there are many values for the shape of points: `"circle"`(default), `"square"`, `"diamond"`, `"cross"`, `"triangle-up"` and `"triangle-down"`.

``` r
mtcars                                                            %>%
  ggvis(~hp, ~mpg)                                                %>%
  layer_points(shape := "triangle-up", fill = ~factor(cyl))
```
<<<<<<< HEAD
![img15](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot15.png)
=======
![img15](/img/codage8-plot15.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

`factor()` converts `cyl` from numeric to categorical data and that makes the plot more clear.

### Special Layers: Model Prediction & Smooths

`layer_model_predictions()` fits a model to the data and draw it with `layer_paths()` and, optionally, `layer_ribbons()`. `layer_smooths()` is a special case of layering model predictions where the model is a smooth `loess` curve whose smoothness is controlled by the span parameter. Both use same properties as `layer_paths()` and `layer_lines()`.

``` r
mtcars %>% ggvis(~wt, ~mpg) %>% layer_smooths()
```
<<<<<<< HEAD
![img16](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot16.png)
=======
![img16](/img/codage8-plot16.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

``` r
mtcars %>% ggvis(~wt, ~mpg) %>% layer_smooths(se = T)
```
<<<<<<< HEAD
![img17](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot17.png)
=======
![img17](/img/codage8-plot17.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

``` r
mtcars                                                            %>% 
  ggvis(~wt, ~mpg)                                                %>% 
  layer_points(shape := "diamond", size := 25, fill = ~cyl)       %>%
  layer_model_predictions(formula = mpg ~ wt, model = "lm",
                          stroke := "red", strokeOpacity := .5)
```
<<<<<<< HEAD
![img18](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot18.png)
=======
![img18](/img/codage8-plot18.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

If you don't specify the `formula` argument in `layer_model_predictions()`, `ggvis` will guess it based on the input in the global data space of `ggvis()`.

There are more properties and `layer_<marks>()`, you could make a research by typing `?ggvis::marks` and `??ggvis::layer_`.

Layers Equivalence
------------------

### Model Prediction, Smooths & Densities

Some layers can be realized in another way. Consider how is the `layer_model_predictions()` processed. 1. Estimate the model based on the dataset; 2. Compute predicted data; 3. Plot the predicted line.

In `ggvis`, `compute_model_prediction()` realizes the first 2 steps.

``` r
mtcars                                                            %>%
  compute_model_prediction(mpg ~ wt, model = "lm")                %>%
  head()
```

The function returns 2 columns `pred_` and `resp_`.

``` r
mtcars                                                            %>%
  compute_model_prediction(mpg ~ wt, model = "lm")                %>%
  ggvis(~pred_, ~resp_)                                           %>%
  layer_paths(stroke := "blue")
```
<<<<<<< HEAD
![img19](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot19.png)
=======
![img19](/img/codage8-plot19.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

The chunk returns the same line as the `layer_model_predictions()` did before. `layer_smooths()` and `layer_densities()` can be splitted to 2 steps in the same way.

And you can easily find the corresponding relations as below:

| layer_                | compute_                    | layer_ |
|-----------------------|-----------------------------|--------|
| **model_predictions** | model_predictions           | paths  |
| **smooths**           | smooths                     | paths  |
| **histograms**        | bin                         | rects  |
| **densities**         | density                     | lines  |
| **bar**               | count/tabulate/stack/align  | rects  |
| **boxplots**          | boxplot/stack               | rects  |

### Equivalence of Histograms

`compute_bin()` returns 5 columns where `count_` is the count in the each interval, `xmin_` and `xmax_` are the left-most and right-most border of the interval. We can use these 3 columns to plot the rectangles in the intervals.

``` r
iris                                                              %>%
  compute_bin(~Sepal.Length, width = 0.5)                         %>%
  ggvis(x = ~xmin_, x2 = ~xmax_, y = 0, y2 = ~count_)             %>% 
  layer_rects(fill := "red", opacity := .7)
```
<<<<<<< HEAD
![img20](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot20.png)
=======
![img20](/img/codage8-plot20.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

### Equivalence of Density Plot

`compute_density()` returns `pred_` and `resp_` - concatenate them by `layer_lines()` brings the exact density line.

``` r
faithful                                                          %>%
  compute_density(~waiting)                                       %>%
  ggvis(~pred_, ~resp_)                                           %>%
  layer_lines(stroke := "red", fill := "red", fillOpacity := 0.5)
```
<<<<<<< HEAD
![img21](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot21.png)
=======
![img21](/img/codage8-plot21.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

### Equivalence of Barchart

`compute_count()` and `compute_tabulate()` only returns 2 columns - `x_` at which point what's the level of corresponding data `count_`. And `compute_align()` helps to set up the lower and upper bound of interval.

``` r
pressure                                                          %>%
  compute_count(~temperature, ~pressure)                          %>%
  compute_align(~x_, length = 15)                                 %>%
  ggvis(x = ~xmin_, x2 = ~xmax_, y = 0, y2= ~count_)              %>% 
  layer_rects(fill := "red", opacity := .5)
```
<<<<<<< HEAD
![img22](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot22.png)
=======
![img22](/img/codage8-plot22.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

In `compute_align()`, `length` limits the length of an interval. It's equivalent to the `width` in `layer_bars()`

How does `compute_stack()` works?

``` r
mtcars                                                            %>%
  compute_stack(stack_var = ~wt, group_var = ~cyl)                %>%
  ggvis(~cyl, ~wt)                                                %>%
  layer_rects(x = ~cyl - 0.5, x2 = ~cyl + 0.5, 
              y = ~stack_upr_, y2 = ~stack_lwr_)
```
<<<<<<< HEAD
![img23](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot23.png)
=======
![img23](/img/codage8-plot23.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

`compute_stack()` generates 3 new variables based on data source: `group__`, `stack_upr_` and `stack_lwr_`. The latter 2 variables indicate the upper and lower y coordinates of each stack.

Try another classic stacked barchart:

``` r
library(data.table, quietly = T)

FordSales <- data.frame(ID = 1:16,
                        Year = rep(2011:2014, each = 4),
                        Car = c("Fiesta", "Focus", "Mondeo", "Escort"),
                        Sales = round((rnorm(16)+10)*100, 0))

FordSales                                                         %>%
  compute_stack(stack_var = ~Sales, group_var = ~Year)            %>%
  ggvis(~factor(Year), ~Sales, fill = ~Car, fillOpacity := .5)    %>%
  layer_rects(x = ~Year - .3, x2 = ~Year + .3,
              y = ~stack_upr_, y2 = ~stack_lwr_)
# You may find the ticks on x-axis are not appropriate.
```
<<<<<<< HEAD
![img24](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot24.png)
=======
![img24](/img/codage8-plot24.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

The compound way to realize a layer helps to understand how the high level layer is generated, and gives you a way to grab the key data for plotting.

### Working with `dplyr` Package

`ggvis` can work along with `dplyr` package. For example to recreate the plot above:

``` r
FordSales                                                         %>%
  group_by(Car)                                                   %>%
  ggvis(~factor(Year), ~Sales)                                    %>%
  layer_bars(fill = ~Car, fillOpacity := .5, width = 0.6)
```
<<<<<<< HEAD
![img25](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot25.png)
=======
![img25](/img/codage8-plot25.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

`group_by()` also works for multiple variables.

``` r
mtcars                                                            %>% 
  group_by(cyl, am)                                               %>% 
  ggvis(~mpg, fill = ~factor(cyl))                                %>%
  layer_densities()
```
<<<<<<< HEAD
![img26](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot26.png)
=======
![img26](/img/codage8-plot26.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

Interactive Output
------------------

*****
*In this article, all interactivity is disabled, only screenshots are provided*
*****

`ggvis` enables interactive HTML output by a series of `input_<form_controls>()` functions. There are 7 input widgets:

-   `input_checkbox()` creates an interactive checkbox;
-   `input_select()`, `input_checkboxgroup()` and `input_radiobuttons()`
    create interactive control to select on or more options from a list;
-   `input_slider()` creates an interactive slider; `input_numeric()`;
-   `input_text()` create an interactive numeric or text input box.

### Single Checkbox

``` r
mtcars                                                            %>% 
  ggvis(~wt, ~mpg)                                                %>%
  layer_smooths(se = input_checkbox(label = "Confidence interval"))
```
<<<<<<< HEAD
![img27](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot27.png)
=======
![img27](/img/codage8-plot27.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

You can also split the plotting steps if the input clause is too long.

``` r
model_type <- input_checkbox(label = "Use flexible curve",
  map = function(val) if(val) "loess" else "lm")

mtcars                                                            %>% 
  ggvis(~wt, ~mpg)                                                %>%
  layer_model_predictions(model = model_type)
```
<<<<<<< HEAD
![img28](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot28.png)
=======
![img28](/img/codage8-plot28.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

### Selection from A List

`input_select()` and `input_checkboxgroup()` allows multiple choice. `input_radiobuttons()` allows only one option to be picked up in the list.

``` r
# input_select()
faithful                                                          %>% 
  ggvis(~waiting, ~eruptions, fillOpacity := 0.5, 
        shape := input_select(label = "Choose shape:", 
                              choices = c("circle", "square", 
                                          "cross", "diamond", 
                                          "triangle-up",
                                          "triangle-down")),
        fill := input_select(label = "Choose color:", 
                             choice = c("black", "red", 
                                        "blue", "green")))        %>% 
  layer_points()
```
<<<<<<< HEAD
![img29](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot29.png)
=======
![img29](/img/codage8-plot29.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

``` r
mtcars                                                            %>% 
  ggvis(~mpg, ~wt, 
        fill = input_select(label = "Choose fill variable:", 
                            choice = names(mtcars), 
                            map = as.name))                       %>% 
  layer_points()
```
<<<<<<< HEAD
![img30](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot30.png)
=======
![img30](/img/codage8-plot30.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

``` r
# input_radiobuttons()
mtcars                                                            %>%
  ggvis(~wt, ~mpg)                                                %>%
  layer_model_predictions(model = input_radiobuttons(
    choices = c("Linear" = "lm", "LOESS" = "loess"),
    selected = "loess",
    label = "Model type"))
```
<<<<<<< HEAD
![img31](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot31.png)
=======
![img31](/img/codage8-plot31.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

``` r
# input_checkboxgroup()
mtcars                                                            %>%
  ggvis(x = ~wt, y = ~mpg)                                        %>%
  layer_points(
    fill := input_checkboxgroup(
      choices = c("Red" = "r", "Green" = "g", "Blue" = "b"),
      label = "Point color components",
      map = function(val) {
        rgb(0.8 * "r" %in% val, 0.8 * "g" %in% val, 0.8 * "b" %in% val)
      }
    )
  )
```
<<<<<<< HEAD
![img32](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot32.png)
=======
![img32](/img/codage8-plot32.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

The argument `map` should be function with one single argument and returns a modified value based on this funciton. **When you maps the variable name to a property, remember to use`=` instead of `:=`**

### Slider

A slider is quite useful for controlling the argument of continuous data, for example, control the binwidth of a histogram:

``` r
mtcars                                                            %>% 
  ggvis(~mpg)                                                     %>% 
  layer_histograms(width = input_slider(label = "Choose a binwidth:", 
                                        min = 1, max = 20))
```
<<<<<<< HEAD
![img33](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot33.png)
=======
![img33](/img/codage8-plot33.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

### Numeric & Text Input Box

To control the binwidth, you could also directly assign a value by `select_numeric()`.

``` r
mtcars                                                            %>% 
  ggvis(~mpg)                                                     %>% 
  layer_histograms(width = input_numeric(label = "Choose a binwidth:", 
                                         value = 1))
```
<<<<<<< HEAD
![img34](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot34.png)
=======
![img34](/img/codage8-plot34.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

To control the fill color, you could use `select_text()`.

``` r
mtcars                                                            %>% 
  ggvis(~mpg)                                                     %>% 
  layer_histograms(fill := input_text(label = "Choose a color:", 
                                      value = "skyblue"))
```
<<<<<<< HEAD
![img35](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot35.png)
=======
![img35](/img/codage8-plot35.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

Axes, Legends & Scales
----------------------

### Axes

We use `add_axis()` to adjust the axis.

``` r
faithful                                                          %>% 
  ggvis(~waiting, ~eruptions)                                     %>% 
  layer_points(fill := "red", fillOpacity := .3, size := 50)      %>% 
  add_axis("y", title = "Duration of eruption (m)")               %>%
  add_axis("x", title = "Time since previous eruption (m)")
```
<<<<<<< HEAD
![img36](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot36.png)
=======
![img36](/img/codage8-plot36.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

The first argument specify horizontal or vertical axis. Use `title` to name the axes. You can add more details:

``` r
faithful                                                          %>% 
  ggvis(~waiting, ~eruptions)                                     %>% 
  layer_points(fill := "red", fillOpacity := .3, size := 50)      %>% 
  add_axis("y", title = "Duration of eruption (m)",
           value = 2:5, subdivide = 9, orient = "right")          %>%
  add_axis("x", title = "Time since previous eruption (m)",
           values = seq(50, 90, 10), subdivide = 9, orient = "top")
```
<<<<<<< HEAD
![img37](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot37.png)
=======
![img37](/img/codage8-plot37.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

Compare carefully the difference between the 2 plots and you will find what do those arguments serve for.

### Legends

`add_legend()` works similarly to `add_axis()`, except that it alters the legend of a plot. Instead of specifying which axis to change, you have to specify the property you want to add to the legend. For example:

``` r
pressure                                                          %>% 
  ggvis(~temperature, ~pressure, fill = ~pressure)                %>%
  layer_points()                                                  %>%
  add_legend("fill", title = "~ pressure")
```
<<<<<<< HEAD
![img38](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot38.png)
=======
![img38](/img/codage8-plot38.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

`ggvis` will create a separate legend for each property that you use. To do this, you just need to feed `add_legend()` a vector of property names as its first argument. The code below creates legend for 3 properties: `fill`, `shape` and `size`.

``` r
faithful                                                          %>% 
  ggvis(~waiting, ~eruptions, opacity := 0.6, 
        fill = ~factor(round(eruptions)), 
        shape = ~factor(round(eruptions)), 
        size = ~round(eruptions))                                 %>%
  layer_points()                                                  %>%
  add_legend(c("fill", "shape", "size"), title = "~ duration (m)", 
             orient = "left", values = c(2,3,4,5))
```
<<<<<<< HEAD
![img39](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot39.png)
=======
![img39](/img/codage8-plot39.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

### Scales

`ggvis` provides several different functions for creating scales: `scale_datetime()`, `scale_logical()`, `scale_nominal()`, `scale_numeric()`, `scale_singular()`. Each maps a different type of data input to the visual properties that `ggvis` uses.

``` r
mtcars                                                            %>% 
  ggvis(~wt, ~mpg, fill = ~disp, stroke = ~disp, strokeWidth := 2)%>%
  layer_points()                                                  %>%
  scale_numeric("fill", range = c("red", "yellow"))               %>%
  scale_numeric("stroke", range = c("darkred", "orange"))
```
<<<<<<< HEAD
![img40](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot40.png)
=======
![img40](/img/codage8-plot40.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

The chunk above maps the value of `disp` on the scale range between `red` and `yellow` for `fill` color, between `darkred` and `orange` for `stroke` color.

The chunk below maps a categorical variable to fill. `cyl` has 3 unique values so we can provide a range of length 3 with the color names.

``` r
mtcars                                                            %>% 
  ggvis(~wt, ~mpg, fill = ~factor(cyl))                           %>%
  layer_points()                                                  %>%
  scale_nominal("fill", range = c("purple", "blue", "green"))
```
<<<<<<< HEAD
![img41](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot41.png)
=======
![img41](/img/codage8-plot41.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

You can adjust any visual property in your graph with a scale (not just color). For example you can specify the opacity and the domain of axes.

``` r
mtcars                                                            %>% 
  ggvis(x = ~wt, y = ~mpg, fill = ~factor(cyl), opacity = ~hp)    %>%
  layer_points()                                                  %>%
  scale_numeric("opacity", range = c(.2, 1))
```
<<<<<<< HEAD
![img42](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot42.png)
=======
![img42](/img/codage8-plot42.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

``` r
mtcars                                                            %>% 
  ggvis(~wt, ~mpg, fill = ~disp)                                  %>%
  layer_points()                                                  %>%
  scale_numeric("y", domain = c(0, NA))                           %>%
  scale_numeric("x", domain = c(0, 6))
```
<<<<<<< HEAD
![img43](http://7xndoy.com1.z0.glb.clouddn.com/codage8-plot43.png)
=======
![img43](/img/codage8-plot43.png)
>>>>>>> 9c59c24b32345a452f4eed352acbb12b705e1ef1

`scale_numeric("y", domain = c(0, NA))` means there is no limit on the maximum value on the y-axis.

*****
**Be aware: ggvis interactivity cannot be displayed in HTML file converted from `.Rmd` by `knitr`.**
*****

Reference
---------

1.  [Github - RStudio ggvis repository](https://github.com/rstudio/ggvis)
2.  [ggvis 0.4 overview](http://ggvis.rstudio.com/)
3.  [Data Visualization in R with ggvis](https://www.datacamp.com/courses/ggvis-data-visualization-r-tutorial)
4.  [ggvis: Interactive, intuitive graphics with R](ggvis:%20Interactive,%20intuitive%20graphics%20with%20R)
5.  [Goodbye static graphs, hello shiny, ggvis, rmarkdown (vs JS solutions)](http://www.r-bloggers.com/goodbye-static-graphs-hello-shiny-ggvis-rmarkdown-vs-js-solutions/)
