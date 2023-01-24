---
Title: ChatGPT Prototyped Our New Feature
Draft: true
Date: 2023-01-24
Image: https://wakatime.com/static/img/blog/files-bubble-chart.png
Description: How we used ChatGPT to prototype our latest feature development.
Author: Alan Hamlett
AuthorUrl: https://wakatime.com/@alan
AuthorAvatar: https://wakatime.com/photo/@alan?size=420
Category: New Features
Tags: chatgpt
---

<img src="https://wakatime.com/static/img/blog/files-bubble-chart.png" class="img-thumbnail" alt="time spent per file as bubble chart" style="width:90%" />

<img src="https://wakatime.com/static/img/blog/openai-bubble-chart-2023-01-24.png" class="img-thumbnail" alt="chatgpt conversation" style="width:100%" />

Final working output from ChatGPT:

    <!DOCTYPE html>
    <html>
    <head>
      <script src="https://d3js.org/d3.v6.min.js"></script>
      <style>
        /* set the CSS */
        .bubble {
          stroke: white;
          stroke-width: 2px;
        }
        .label {
          font-size: 12px;
          text-anchor: middle;
        }
      </style>
    </head>
    <body>
      <div id="chart"></div>
      <script>
        // set the data
        var data = [
          { item: "Item 1", time: 20 },
          { item: "Item 2", time: 10 },
          { item: "Item 3", time: 30 },
          { item: "Item 4", time: 15 }
        ];

        // set the dimensions and margins of the chart
        var width = 600;
        var height = 600;
        var margin = 30;

        //create a color scale
        var color = d3.scaleOrdinal()
          .domain(data.map(function(d) { return d.item; }))
          .range(d3.schemeCategory10);

        // create the chart
        var chart = d3.select("#chart")
          .append("svg")
            .attr("width", width)
            .attr("height", height)
          .append("g")
            .attr("transform", "translate(" + margin + "," + margin + ")");

        // set the pack layout
        var pack = d3.pack()
          .size([width - margin, height - margin])
          .padding(2);

        // create the root node
        var root = d3.hierarchy({ children: data })
          .sum(function(d) { return d.time; });

        // create the nodes
        var nodes = pack(root).leaves();

        // create the bubbles
        var bubbles = chart.selectAll(".bubble")
          .data(nodes)
          .enter()
          .append("circle")
            .classed("bubble", true)
            .attr("r", function(d) { return d.r; })
            .attr("cx", function(d) { return d.x; })
            .attr("cy", function(d) { return d.y; })
            .style("fill", function(d) { return color(d.data.item); });

        //create the text label
        var labels = chart.selectAll(".label")
          .data(nodes)
          .enter()
          .append("text")
            .classed("label", true)
            .attr("x", function(d) { return d.x; })
            .attr("y", function(d) { return d.y; })
            .text(function(d) { return d.data.item; });
      </script>
    </body>
    </html>

