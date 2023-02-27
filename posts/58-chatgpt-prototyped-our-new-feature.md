---
Title: ChatGPT Prototyped Our New Feature
Date: 2023-01-24
Image: https://wakatime.com/static/img/blog/files-bubble-chart.png
Description: How we used ChatGPT to save time building our latest website feature.
Author: Alan Hamlett
AuthorUrl: https://wakatime.com/@alan
AuthorAvatar: https://wakatime.com/photo/@alan?size=420
Category: New Features
Tags: chatgpt
---

### Intro

I’m naturally a skeptic about OpenAI and ChatGPT, probably because I still think of AI as simple supervised machine learning with markov models.
That’s why it’s taken me a few months to try out the new hotness, [ChatGPT][chatgpt].
For my first try, I decided to see if it could prototype a new feature:

A bubble chart showing your most used files in a project.

### The Story, according to ChatGPT

Recently, the WakaTime team was in search of a way to visually represent the amount of time their users were spending on different files.
They knew that a bubble chart would be the perfect solution, but they were struggling with creating one from scratch.
That's when they came across ChatGPT, an AI language model developed by OpenAI.

To their surprise, ChatGPT was able to provide them with a working prototype of the bubble chart feature within minutes.
Using the d3.js library, the prototype generated a packed bubble chart that displayed the time spent on each file as the size of the corresponding bubble.
The team was amazed by the speed and accuracy of ChatGPT's work.

After some minor modifications and hooking the code up to real WakaTime data, the WakaTime team was able to successfully implement the feature on their website.
The new bubble chart feature has received positive feedback from users, who appreciate the visual representation of their file usage data.

The WakaTime team couldn't be happier with the results and are now looking for new ways to utilize ChatGPT's capabilities.
They were impressed by the AI's ability to understand their needs and provide a working solution in such a short amount of time.
This experience has proven to be a game changer for the team and they look forward to more opportunities to work with ChatGPT in the future.

<a href="https://wakatime.com/static/img/blog/openai-bubble-chart-2023-01-24.png"><img src="https://wakatime.com/static/img/blog/openai-bubble-chart-2023-01-24.png" class="img-thumbnail" alt="chatgpt conversation" style="width:100%" /></a>

### Results

Here’s the final code ChatGPT produced, and it works!

<img src="https://wakatime.com/static/img/blog/openai-bubble-chart-prototype.png" class="img-thumbnail" alt="output from running the code" style="width:60%" />

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

After some minor modifications and connecting real data, here’s a screenshot from a [WakaTime project dashboard][project dashboard].

And yes, the story section was generated by ChatGPT ;-)

<a href="https://wakatime.com/projects"><img src="https://wakatime.com/static/img/blog/files-bubble-chart.png" class="img-thumbnail" alt="time spent per file as bubble chart" style="width:90%" /></a>

[project dashboard]: https://wakatime.com/projects
[chatgpt]: https://chat.openai.com/
