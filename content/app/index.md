---
description: |
  Various shiny apps, dashboards, and static html/mds I've put together.
title: Apps
weight: 5
---

<link rel="stylesheet" href="/css/cards.css">


<div class="flex-container">

  <div class="card" style="width: 250px;cursor: pointer;" onclick="window.open('https://rjfranssen.shinyapps.io/tiger-tracks/')">
    <div class="container">
    <img src="/images/tiger-tracks-card.png" alt="tiger-tracks-shiny" style="width:100%">
    <p>my favorite shiny app with cool home ranging models and time lapse tiger tracking</p>
    </div>
  </div>

<!--
  <div class="card" style="width: 250px;cursor: pointer;" onclick="window.open('dashes/flexdashboard-snippets.html')">
    <div class="container">
    <img src="/images/flexdashboard-snippets-card.png" alt="flexdashboard-snippets" style="width:100%">
      static flexdashboard page that i use for fast chart and map snippets
    </div>
  </div>
-->

  <div class="card" style="width: 250px;cursor: pointer;" onclick="window.open('https://rjfranssen.shinyapps.io/covid-dash/')">
    <div class="container">
    <img src="/images/covid-dash-card.png" alt="covid-dash" style="width:100%">
    <p>shiny app i created to demo view covid data updates using r and js</p>
    </div>
  </div>
  
  <div class="card" style="width: 250px;cursor: pointer;" onclick="window.open('https://rjfranssen.shinyapps.io/crime-dash/')">
    <div class="container">
    <img src="/images/crime-dash-card.png" alt="crime-dash" style="width:100%">
      <p>one of my first apps - trying to get my hands around interactivity</p>
    </div>
  </div>
  
</div>


<!-- Notes: -->
<!-- flex-wrap: https://www.w3schools.com/css/css3_flexbox_container.asp -->
<!-- js onclick: https://stackoverflow.com/questions/45393499/javascript-onclick-into-a-new-window -->

<!-- Quick notes regarding relative file paths: relative file paths:   -->

<!-- Starting with “/” returns to the root directory and starts there -->
<!-- Starting with “../” moves one directory backwards and starts there -->
<!-- Starting with “../../” moves two directories backwards and starts there (and so on…) -->
<!-- To move forward, just start with the first subdirectory and keep moving forward -->