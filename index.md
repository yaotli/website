
``` r
require( parallel )
```

    ## Loading required package: parallel

``` r
options(rgl.printRglwidget = TRUE)
numCores = detectCores()


number_of_people_tested  = 86023
positive_case  = 487

prevalence  = 10^seq(-9, -2, by = 0.03)
sensitivity = seq(0.95, 1, by = 0.001)
specificity = seq(0.95, 1, by = 0.001)

param_grid = expand.grid(prevalence = prevalence, sensitivity = sensitivity, specificity = specificity)

SUB_param_grid_i = sample( size = 1000, dim( param_grid )[1] )
SUB_param_grid   = param_grid[ SUB_param_grid_i,  ]

likelihood = 
  mclapply( as.list( seq(1, dim( SUB_param_grid )[1] ) ),
          function(x)
          {
            real_covid = rbinom( n = 1000, size = number_of_people_tested, prob = SUB_param_grid[x,1] )  
            
            monte_carlo_nominal_positive = 
              sapply( real_covid, function(y) 
                {
                true_positive  = rbinom( 1, size = y, prob = SUB_param_grid[x,3] )
                false_positive = rbinom( 1, size = number_of_people_tested - y, prob = 1 - SUB_param_grid[x,3] )
                return(true_positive + false_positive)
                } )
            
            likelihood = dnorm( positive_case ,
                                mean = mean( monte_carlo_nominal_positive ),
                                sd   = sd( monte_carlo_nominal_positive ))
            
            return(likelihood)
            
          }, 
          mc.cores = numCores )

SUB_param_grid$elikelihood = likelihood
```

``` r
require( plotly )
```

    ## Loading required package: plotly

    ## Loading required package: ggplot2

    ## Warning: package 'ggplot2' was built under R version 3.5.2

    ## 
    ## Attaching package: 'plotly'

    ## The following object is masked from 'package:ggplot2':
    ## 
    ##     last_plot

    ## The following object is masked from 'package:stats':
    ## 
    ##     filter

    ## The following object is masked from 'package:graphics':
    ## 
    ##     layout

``` r
fig = plot_ly( SUB_param_grid[ SUB_param_grid$elikelihood > 1e-4, ], 
                x = ~prevalence, 
                y = ~sensitivity, 
                z = ~specificity, 
                opacity = 0.5,
                marker = list(color = ~elikelihood, colorscale = c('#FFE1A1', '#683531'), showscale = TRUE))
fig = fig %>% add_markers()
fig = fig %>% layout(scene = list( xaxis = list(title = 'Prevalence'),
                                   yaxis = list(title = 'Sensitivity'),
                                   zaxis = list(title = 'Specificity')),
                     annotations = list( x    = 1.13,
                                         y    = 1.05,
                                         text = "Estimated likelihood",
                                         xref = 'paper',
                                         yref = 'paper',
                                         showarrow = FALSE
                                         ))

fig
```

![](test_files/figure-gfm/plot-1.png)<!-- -->
