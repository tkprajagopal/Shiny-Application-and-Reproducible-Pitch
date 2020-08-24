Developing Data Products: Course Project
========================================================
 

Introduction and Overview
========================================================

This presentation and the associated Shiny app conclude the Coursera course: Developing Data products. 

Included in this project are:

- this presentation, providing an overview, code examples and links
- a Shiny app hosted on shinyapp.io
- the corresponding source code hosted via github

Please see the Appendix for links to the files.

UI Examples
========================================================

![Plot Tab](https://github.com/tkprajagopal/Shiny-Application-and-Reproducible-Pitch/blob/master/gapminder_using_shiny_1.PNG)

![Table Tab](https://github.com/tkprajagopal/Shiny-Application-and-Reproducible-Pitch/blob/master/gapminder_using_shiny_2.PNG)

server.R
========================================================


```r
library(plotly)
library(colourpicker)
library(ggplot2)
library(gapminder)
library(shinycustomloader)
library(DT)

server <- function(input, output) {
    filtered_data <- reactive({
        data <- gapminder
        data <- subset(
            data,
            lifeExp >= input$life[1] & lifeExp <= input$life[2]
        )
        if (input$continent != "All") {
            data <- subset(
                data,
                continent == input$continent
            )
        }
        data
    })
    
    output$table <- DT::renderDataTable({
        data <- filtered_data()
        data
    })
    
    output$download_data <- downloadHandler(
        filename = function() {
            paste("gapminder-data-", Sys.Date(), ".csv", sep="")
        },
        content = function(file) {
            write.csv(filtered_data(), file)
        }
    )
    
    
    output$plot <- renderPlotly({

        ggplotly({
            data <- filtered_data()
            
            p <- ggplot(data, aes(gdpPercap, lifeExp)) +
                geom_point(size = input$size, col = input$color) +
                scale_x_log10() +
                ggtitle(input$title) 
                
            p
        })
    })
}
```

ui.R
========================================================
```r
library(plotly)
library(colourpicker)
library(ggplot2)
library(gapminder)
library(shinycustomloader)
library(DT)

ui <- fluidPage(
    
    # App title ----
    titlePanel("Gapminder Data Visualization using Shiny and Plotly"),
    
    # Sidebar layout with input and output definitions ----
    sidebarLayout(
        
        # Sidebar panel for inputs ----
        sidebarPanel(
            
            # Input: Select the random distribution type ----
            textInput("title", "Title", "GDP vs life exp"),
            numericInput("size", "Point size", 1, 1),
            colourInput("color", "Point color", value = "blue"),

            
            selectInput("continent", "Continent",
                        choices = c("All", levels(gapminder$continent))),

            sliderInput(inputId = "life", label = "Life expectancy",
                        min = 0, max = 120,
                        value = c(30, 50)),
            
            downloadButton("download_data")
            
            
        ),
        
        # Main panel for displaying outputs ----
        mainPanel(
            
            # Output: Tabset w/ plot, summary, and table ----
            tabsetPanel(type = "tabs",
                        
                        tabPanel("Plot", withLoader(plotlyOutput("plot")) ),
                        tabPanel("Table", withLoader(DT::dataTableOutput("table")))
    
                        
            )
            
        )
    )
)
```

Links and Appendix
========================================================

- Shiny app: https://halici.shinyapps.io/Gapminder-Data-Visualization-using-Shiny-and-Plotly/
- Source Code: https://github.com/tkprajagopal/Developing-Data-Products-course--Assignment-Week-4-Shiny-Application-and-Reproducible-Pitch/tree/master/Shiny 
- Presentation is available via RPubs: http://rpubs.com/tkprajagopal/Reproducible-Pitch-Presentation
