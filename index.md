---
layout: page
status: publish
title: howisFelix.today?
type: page
published: true
meta: {}
---

{% include instapipe.html %}
<script type="text/javascript">
  let url = "https://backend.howisfelix.today/api.json"

  function daysAgo(date) {
    const deltaDays = Math.floor((Date.now() - date.getTime()) / (1000 * 3600 * 24));
    if (deltaDays === 0) {
      return "today";
    } else if (deltaDays === 1) {
      return "yesterday";
    } else {
      return deltaDays + " days ago";
    }
  }
  function daysOrHoursAgo(date) {
    if (daysAgo(date) == "today") {
      const hours = Math.floor((Date.now() - date.getTime()) / (1000 * 3600));
      if (hours == 1) { return "1 hour ago"; }
      else if (hours < 1) { return "less than an hour ago"; }
      else { return hours + " hours ago"; }
    } else {
      return daysAgo(date);
    }
  }

  httpGetAsync(url, function(data) {
    const otherFxLifeData = data["otherFxLifeData"]

    // Render map
    document.getElementById("currentLocationMap").style.background = "url('" + data["mapsUrl"] + "') no-repeat"

    // Render current & next locations
    if (data["isMoving"] == false) {
      document.getElementById("isMovingContainer").style.display = "none"
      document.getElementById("currentCityB").innerHTML = data["currentCityText"]
    } else {
      document.getElementById("isMovingContainer").style.display = "block"
      document.getElementById("currentCityContainer").style.display = "none"
      document.getElementById("nextCityB").innerHTML = data["currentCityText"].replace("✈️ ", "")
    }
    if (data["nextCityText"]) {
      document.getElementById("nextCityText").innerHTML = data["nextCityText"]
      document.getElementById("nextCityTime").innerHTML = data["nextCityDate"]
      document.getElementById("nextCityContainer").style.display = "block"
    }

    if (data["nextStays"].length > 0) {
      // Iterate over all the next stays, and append a new tr to the table
      for (let i = 0; i < data["nextStays"].length; i++) {
        const stay = data["nextStays"][i]
        const tr = document.createElement("tr")
        const td = document.createElement("td")
        td.innerHTML = stay["name"].replace("United States", "USA")
        tr.appendChild(td)
        const td2 = document.createElement("td")
        let fromDate = new Date(stay["fromDate"])
        fromDate.setDate(fromDate.getDate());
        td2.innerHTML = fromDate.toLocaleDateString("en-US", {day: 'numeric', month: 'short'})
        tr.appendChild(td2)
        const td3 = document.createElement("td")
        let toDate = new Date(stay["toDate"])
        toDate.setDate(toDate.getDate());
        td3.innerHTML = toDate.toLocaleDateString("en-US", {day: 'numeric', month: 'short'})
        tr.appendChild(td3)
        document.getElementById("next-cities-table").appendChild(tr)
      }
      document.getElementById("next-cities-table").style.display = "inline"
    }

    // Render today's metadata
    document.getElementById("current-weight").innerHTML = 
      "<span class='highlighted'>" + (otherFxLifeData["weight"]["value"]).toFixed(1) + " kg</span>/ " +
      (otherFxLifeData["weight"]["value"] / 0.453592).toFixed(1) + " lbs"
    document.getElementById("current-weight-time").innerHTML = "(" + daysAgo(new Date(otherFxLifeData["weight"]["time"])) + ")"
    document.getElementById("current-sleep-duration").innerHTML =
      "<span class='highlighted'>" + otherFxLifeData["sleepDurationWithings"]["value"] + " hours</span> <span class=\"ago-subtle\">(last night)</span>"
    document.getElementById("last-workout").innerHTML = daysAgo(new Date(otherFxLifeData["gym"]["time"]))
    document.getElementById("data-points").innerHTML = otherFxLifeData["totalAmountOfEntries"]["value"].toLocaleString()
    document.getElementById("data-entries-count").innerHTML = otherFxLifeData["totalAmountOfEntries"]["value"].toLocaleString() + " data entries"
    document.getElementById("total-computer-time").innerHTML = otherFxLifeData["totalComputerUsageHours"]["value"].toLocaleString() + " hours"
    document.getElementById("inbox-count").innerHTML = otherFxLifeData["emailsInbox"]["value"].toLocaleString() + " emails"
    document.getElementById("trello-count").innerHTML = data["numberOfTodoItems"].toLocaleString() + " tasks"
    document.getElementById("visitors-count").innerHTML = data["websiteVisitors"]["howisfelix.today"]["alltime"].toLocaleString() + " <span class=\"ago-subtle\">(<a href='https://plausible.io/howisfelix.today?period=all' target='_blank'>since August</a>)</span>"

    // Overview of data sources
    document.getElementById("h-rescuetime").innerHTML = otherFxLifeData["rescue_time"]["value"].toLocaleString()
    document.getElementById("h-swarm").innerHTML = otherFxLifeData["swarm"]["value"].toLocaleString()
    document.getElementById("h-timeranges").innerHTML = otherFxLifeData["timeRanges"]["value"].toLocaleString()
    document.getElementById("h-weather").innerHTML = otherFxLifeData["weather"]["value"].toLocaleString()
    document.getElementById("h-health").innerHTML = otherFxLifeData["dailySteps"]["value"].toLocaleString()

    document.getElementById("h-manually").innerHTML = (otherFxLifeData["totalAmountOfEntries"]["value"] - 
          otherFxLifeData["rescue_time"]["value"] -
          otherFxLifeData["swarm"]["value"] - 
          otherFxLifeData["timeRanges"]["value"] - 
          otherFxLifeData["weather"]["value"] - 
          otherFxLifeData["dailySteps"]["value"]).toLocaleString();

    // Git Details
    document.getElementById("git-time-ago").innerHTML = daysOrHoursAgo(new Date(data["lastCommitTimestamp"]))
    document.getElementById("git-link").href = data["lastCommitLink"]
    document.getElementById("git-link").innerHTML = data["lastCommitMessage"]
    document.getElementById("git-repo-link").href = "https://github.com/" + data["lastCommitRepo"]
    document.getElementById("git-repo-link").innerHTML = data["lastCommitRepo"]

    // Mood
    document.getElementById("current-feeling").innerHTML = data["currentMoodLevel"] + " " + data["currentMoodEmoji"]
    document.getElementById("mood-hours-ago").innerHTML = "(" + data["currentMoodRelativeTime"] + ")"

    // Render food data (if available)
    if (data["todaysMacros"] && data["todaysMacros"]["kcal"] > 0) {
      document.getElementById("todaysMacros-kcal").innerHTML = data["todaysMacros"]["kcal"] + " kcal"
      const totalKcal = otherFxLifeData["macrosCarbs"]["value"] * 4 + otherFxLifeData["macrosProtein"]["value"] * 4 + otherFxLifeData["macrosFat"]["value"] * 9;
      document.getElementById("total-kcal").innerHTML = totalKcal

      document.getElementById("todaysMacros-carbs").innerHTML = data.todaysMacros.carbs + "g carbs"
      document.getElementById("todaysMacros-protein").innerHTML = data.todaysMacros.protein + "g protein"
      document.getElementById("todaysMacros-fat").innerHTML = data.todaysMacros.fat + "g fat"

      document.getElementById("macrosCarbs-value").innerHTML = data.otherFxLifeData["macrosCarbs"]["value"]
      document.getElementById("macrosProtein-value").innerHTML = data.otherFxLifeData["macrosProtein"]["value"]
      document.getElementById("macrosFat-value").innerHTML = data.otherFxLifeData["macrosFat"]["value"]


      document.getElementById("todaysMacros-kcal-bar-inner").style.width = Math.min(100, Math.round((data["todaysMacros"]["kcal"] / totalKcal) * 100)) + "%"
      document.getElementById("todaysMacros-protein-bar-inner").style.width = Math.min(100, Math.round((data.todaysMacros.protein / data.otherFxLifeData["macrosProtein"]["value"]) * 100)) + "%"
      document.getElementById("todaysMacros-carbs-bar-inner").style.width = Math.min(100, Math.round((data.todaysMacros.carbs / data.otherFxLifeData["macrosCarbs"]["value"]) * 100)) + "%"
      document.getElementById("todaysMacros-fat-bar-inner").style.width = Math.min(100, Math.round((data.todaysMacros.fat / data.otherFxLifeData["macrosFat"]["value"]) * 100)) + "%"

      // Turn the bars red when too high
      if (data.todaysMacros.protein > data.otherFxLifeData["macrosProtein"]["value"]) {
        document.getElementById("todaysMacros-protein-bar-inner").style.background = "red";
      }
      if (data.todaysMacros.carbs > data.otherFxLifeData["macrosCarbs"]["value"]) {
        document.getElementById("todaysMacros-carbs-bar-inner").style.background = "red";
      }
      if (data.todaysMacros.fat > data.otherFxLifeData["macrosFat"]["value"]) {
        document.getElementById("todaysMacros-fat-bar-inner").style.background = "red";
      }
      if (data["todaysMacros"]["kcal"] > totalKcal) {
        document.getElementById("todaysMacros-kcal-bar-inner").style.background = "red";
      }

      // Render the food list
      let foodEntriesTable = document.getElementById("foodEntriesTable")
      foodEntriesTable.innerHTML = "<tr><th>Food</th><th>Amount</th></tr>"
      foodEntriesTable = foodEntriesTable.getElementsByTagName("tbody")[0]
      for (let i = 0; i < data.todaysFoodItems.length; i++) {
        let foodItem = data.todaysFoodItems[i]
        let row = document.createElement("tr")
        row.className = i >= 3 ? "hidden-food" : ""
        row.style.display = i >= 3 ? "none" : ""
        const amount = foodItem.amount.split("/")[0] // Sometimes mfp has weird slashes, with the units ending up too long
        row.innerHTML = "<td>" + foodItem.name.replace("Parkhurst - ", "") + "</td><td>" + amount.replace(" gram", " g") + "</td>"
        foodEntriesTable.appendChild(row)
      }
      if (data.todaysFoodItems.length > 3) {
        let row = document.createElement("tr")
        row.className = "show-more-food"
        row.innerHTML = "<td colspan='2'><a onclick='toggleFood()' id='show-all-food-a'>Show all food entries</a></td>"
        foodEntriesTable.appendChild(row)
      }
    } else {
      document.getElementById("food-container").style.display = "none"
    }

    // Render the most recent social media photos
    var photos = data["recentPhotos"]
    var personalCarousel = document.getElementById("personalCarousel")
    for (let photoIndex in photos) {
      let currentPhoto = photos[photoIndex]

      var linkNode = document.createElement("a");
      linkNode["href"] = currentPhoto["permalink"] || "https://instagram.com/krausefx"
      linkNode["target"] = "_blank"
      var imageNode = document.createElement("span")
      imageNode["style"] = "background-image: url(" + currentPhoto["thumbnail_url"] + ")"
      imageNode["alt"] = currentPhoto["text"]

      linkNode.appendChild(imageNode)
      personalCarousel.appendChild(linkNode)
    }

    document.getElementById("realTimeDataDiv").style.display = "block"

    while (document.getElementsByClassName("blurred").length > 0) {
      document.getElementsByClassName("blurred")[0].classList.remove("blurred")
    }

    // Render the average tables
    const overviewTable = otherFxLifeData["overviewTable"]["value"];
    const keys = [{
        "name": "gym",
        "customTitle": "Gym visits",
        "unit": "",
        "rounding": 0,
        "good": "up",
        "percentage": true
    }, {
        "name": "withingssleepinghr",
        "customTitle": "Sleeping Heart Rate",
        "unit": "beats/min",
        "rounding": 1,
        "good": "bad",
        "percentage": false
    }, {
        "name": "veggies",
        "customTitle": "Ate Vegetables",
        "unit": "",
        "rounding": 0,
        "good": "up",
        "percentage": true
    }];
    // Key have to be lower case
    const rows = ["week", "month", "quarter", "alltime"]; /* "year" */
    for (var i = 0; i < keys.length; i++) {
        const keyData = keys[i];
        const key = keyData["name"]
        const value = overviewTable[key];
        const container = document.getElementById("table-container-averages");
        const table = document.createElement("table");
        table.setAttribute("cellspacing", 0)
        table.setAttribute("cellpadding", 0)
        table.setAttribute("id", key);
        table.setAttribute("class", "metrics-table");
        const thead = document.createElement("thead");
        const tr = document.createElement("tr");
        const th = document.createElement("th");
        th.innerHTML = keyData["customTitle"];
        th.colSpan = "2"
        tr.appendChild(th);
        thead.appendChild(tr);
        table.appendChild(thead);
        const tbody = document.createElement("tbody");

        // Calculate the minimum and maximum value of each metric
        let minValue = null;
        let maxValue = null;
        for (var j = 0; j < rows.length; j++) {
            let calculatedValue = overviewTable[key + rows[j]]
            if (calculatedValue != null) {
                if (minValue == null || calculatedValue < minValue) {
                    minValue = calculatedValue;
                }
                if (maxValue == null || calculatedValue > maxValue) {
                    maxValue = calculatedValue;
                }
            }
        }

        for (var j = 0; j < rows.length; j++) {
            const tr = document.createElement("tr");
            const td = document.createElement("td");
            td.innerHTML = rows[j].charAt(0).toUpperCase() + rows[j].slice(1); // capitalize
            td.innerHTML = td.innerHTML.replace("Alltime", "All");
            tr.appendChild(td);
            const td2 = document.createElement("td");
            let calculatedValue = overviewTable[key + rows[j]]
            if (calculatedValue == maxValue) {
                if (keyData["good"] == "up") {
                    td2.setAttribute("class", "best-value");
                } else {
                    td2.setAttribute("class", "worst-value");
                }
            } else if (calculatedValue == minValue) {
                if (keyData["good"] == "up") {
                    td2.setAttribute("class", "worst-value");
                } else {
                    td2.setAttribute("class", "best-value");
                }
            }
            let suffix = "";
            if (keyData["percentage"]) {
                calculatedValue *= 100
                suffix = "% of days"
            }
            calculatedValue = roundNumberExactly(calculatedValue, keyData["rounding"]);

            if (keyData["unit"]) {
                suffix = " " + keyData["unit"];
            }
            td2.innerHTML = calculatedValue + suffix;
            // td.innerHTML += " (" + overviewTable[key + rows[j] + "count"] + ")";
            tr.appendChild(td2);
            tbody.appendChild(tr);
        }
        table.appendChild(tbody);
        container.appendChild(table);
    }
  })

  // Round the given number, exact number of decimal places
  function roundNumberExactly(number, decimals) {
    return (Math.round(number * Math.pow(10, decimals)) / Math.pow(10, decimals)).toFixed(decimals);
  }
  function httpGetAsync(url, callback) {
    var xmlHttp = new XMLHttpRequest();
    xmlHttp.onreadystatechange = function() { 
        if (xmlHttp.readyState == 4 && xmlHttp.status == 200) {
            callback(JSON.parse(xmlHttp.responseText));
        }
    }
    xmlHttp.open("GET", url, true);
    xmlHttp.send(null);
  }

  function toggleFood() {
    // Iterate over all `hidden-food` rows and toggle their display
    let rows = document.getElementsByClassName("hidden-food")
    for (let i = 0; i < rows.length; i++) {
      rows[i].style.display = rows[i].style.display == "none" ? "table-row" : "none"
    }
    const a = document.getElementById("show-all-food-a")
    a.innerHTML = a.innerHTML == "Show all food entries" ? "Hide food entries" : "Show all food entries"
  }

  // Now, add an additional ? to the URL to make it look howisFelix.today?
  // By modifying the GET parameters
  window.history.replaceState(null, null, "?");
</script>

<div id="mapsContainer">
  <img id="currentLocationMap">
</div>
<div id="story-available" class="story-not-available">
  <img id="storyProfilePicture" src="assets/FelixKrauseCropped.jpg" onclick="showStories()" />
</div>

<div id="realTimeDataDiv" class="hidden-due-to-project-paused">
  <h2 id="currentCityContainer"><a id="fx-user-link" href="https://bsky.app/profile/krausefx.bsky.social">Felix</a> is currently in <b id="currentCityB" class="highlighted blurred">Vienna, AT</b></h2>
  <h3 id="isMovingContainer" style="display: none"><a id="fx-user-link" href="https://bsky.app/profile/krausefx.bsky.social">Felix</a> is currently heading to <b id="nextCityB" class="highlighted"></b></h3>
  <h4 id="nextCityContainer" style="display: none">Leaving for <span id="nextCityText"></span> <span id="nextCityTime"></span></h4>

  <table id="next-cities-table" cellspacing="0" cellpadding="0">
    <thead>
        <tr>
            <th>Upcoming trips</th>
            <th>From</th>
            <th>To</th>
        </tr>
    </thead>
  </table>

  <hr />
  <h3 id="feels-h">
    Felix feels <span class="highlighted blurred" id="current-feeling">happy, excited 😃</span>
    <span id="mood-hours-ago" class="ago-subtle blurred">(3 hours ago)</span>
  </h3>
  <hr />

  <div id="food-container">
    <h3>Felix ate today</h3>
    <div class="food-overview blurred">
      <div>
        <span class="highlighted" id="todaysMacros-kcal">54kcal</span> of <span id="total-kcal">2920 </span>
        <span class="macro-bar-outer">
          <div class="macro-bar-inner" id="todaysMacros-kcal-bar-inner"></div>
        </span>
      </div>
      <div>
        <span class="highlighted" id="todaysMacros-carbs">54g carbs</span> of <span id="macrosCarbs-value">350</span>g
        <span class="macro-bar-outer">
          <div class="macro-bar-inner" id="todaysMacros-carbs-bar-inner"></div>
        </span>
      </div>
      <div>
        <span class="highlighted" id="todaysMacros-protein">24g protein</span> of <span id="macrosProtein-value">200</span>g
        <span class="macro-bar-outer">
          <div class="macro-bar-inner" id="todaysMacros-protein-bar-inner"></div>
        </span>
      </div>
      <div>
        <span class="highlighted" id="todaysMacros-fat">16g fat</span> of <span id="macrosFat-value">80</span>g
        <span class="macro-bar-outer">
          <div class="macro-bar-inner" id="todaysMacros-fat-bar-inner"></div>
        </span>
      </div>
    </div>

    <div id="foodEntries" class="blurred">
      <table id="foodEntriesTable" cellspacing="0" cellpadding="0">
        <tr><th>Food</th><th>Amount</th></tr>
        <tr><td>Club Mate</td><td>500 ml</td></tr>
        <tr><td>Chicken Breast</td><td>500c</td></tr>
        <tr><td>Rice</td><td>200g</td></tr>
        <tr class="show-more-food"><td colspan='2'><a onclick='' id='show-all-food-a'>Show all food entries</a></td></tr>
      </table>
    </div>
    <hr />
  </div>

  <div id="table-container">
    <table id="real-time-table" cellspacing="0" cellpadding="0">
      <tr>
        <td>Weight</td>
        <td>
          <span id="current-weight" class="blurred"><span class="highlighted">81.8kg</span> / 180.4lbs</span>
          <span id="current-weight-time" class="ago-subtle blurred">today</span>
        </td>
      </tr>
      <tr>
        <td>Height</td>
        <td>
          <span class="blurred"><span class="highlighted">1.93m</span> (6'4")</span>
        </td>
      </tr>
      <tr>
        <td>Slept</td>
        <td id="current-sleep-duration" class="blurred"><span class="highlighted">9 hours</span> <span class="ago-subtle">(last night)</span></td>
      </tr>
      <tr>
        <td>Last Workout</td>
        <td><span class="highlighted blurred" id="last-workout">2 days ago</span></td>
      </tr>
      <tr>
        <td>Computer Time</td>
        <td>
          <span class="highlighted blurred" id="total-computer-time">12,677 hours</span>
          <span class="ago-subtle blurred" id="since2013">(since 2013)</span>
        </td>
      </tr>
      <tr>
        <td>Inbox</td>
        <td><span class="highlighted blurred" id="inbox-count">40 emails</span></td>
      </tr>
      <tr>
        <td>Personal Todo Items</td>
        <td><span class="highlighted blurred" id="trello-count">173 tasks</span></td>
      </tr>
      <tr>
        <td>Unique Website Visitors</td>
        <td><span class="highlighted blurred" id="visitors-count">162,457 (<a href="https://plausible.io/howisfelix.today" target="_blank">since August</a>)</span></td>
      </tr>
      <tr>
        <td>Database size</td>
        <td><span class="highlighted blurred" id="data-entries-count">380,000 data entries</span></td>
      </tr>
    </table>
  </div>

  <hr />

  <h2 style="margin-top: -20px;">Most recent photos</h2>
  <div class="imageCarousel" id="personalCarousel"></div>
  <hr />

  <p style="margin-top: -25px;" class="git-footnote" style="margin-bottom: -10px; margin-top: -20px;">Last code commit: <span id="git-time-ago" class="blurred git-footnote">an hour ago</span></p>
  <h3 id="git-header">
    <a target="_blank" href="" id="git-link" class="blurred">Improve design of graph</a>
  </h3>
  <p class="git-footnote" style="margin-bottom: 0px; margin-top: -10px;">on GitHub repo <a target="_blank" href="" id="git-repo-link" class="blurred">KrauseFx/krausefx.com</a></p>
  <hr />

  {% include averages.html %}
  <hr />
</div>

<div style="margin-top: 300px">
</div>

<h1>Background: Why I put my whole life into a single database</h1>

Back in 2019, I started collecting all kinds of [metrics about my life](https://github.com/KrauseFx/FxLifeSheet). Every single day for the last 3 years I tracked over 100 different data types - ranging from fitness & nutrition to social life, computer usage and weather.

<div class="social-media-list" style="float: right; margin-top: 8px; margin-left: 15px;">
  <p>
    <b>Ideas or suggestions?</b><br />
    <span class="ideas-subtitle">I'd love to hear from you!</span>
  </p>
  <ul>
    <li>{% include icon-twitter.html username="KrauseFx" %}</li>
    <li>{% include icon-instagram.html %}</li>
    <li>{% include icon-url.html %}</li>
  </ul>
</div>

**The goal of this project was to answer questions about my life, like**

- How does living in different cities affect other factors like fitness, productivity and happiness?
- How does sleep affect my day, my fitness level, and happiness?
- How does the weather, and the different seasons affect my life?
- Are there any trends over the last few years?
- How does computer time, work and hours in meetings affect my personal life?

Since the start of this project, I collected <b><span id="data-points">~380,000</span> data points</b>, with the biggest data sources being:

<table id="data-sources-overview">
  <tr><th>Data Source</th><th>Number of data entries</th><th>Type of data</th></tr>
  <tr><td><a href="https://rescuetime.com" target="_blank">RescueTime</a></td><td><span class="highlighted" id="h-rescuetime">149,466</span></td><td>Daily computer usage (which website, which apps)</td></tr>
  <tr><td><a href="https://swarmapp.com" target="_blank">Foursquare Swarm</a></td><td><span class="highlighted" id="h-swarm">126,285</span></td><td>Location and POI data, places I've visited</td></tr>
  <tr><td>Manually entered</td><td><span class="highlighted" id="h-manually">67,031</span></td><td>Fitness, mood, sleep, social life, health, nutrition, energy levels, TV, stress, ...</td></tr>
  <tr><td>Manually entered date ranges</td><td><span class="highlighted" id="h-timeranges">19,273</span></td><td>Occupation, lockdown status, living setup</td></tr>
  <tr><td>Weather API</td><td><span class="highlighted" id="h-weather">15,442</span></td><td>Temperature, rain, sunlight, wind</td></tr>
  <tr><td>Apple Health</td><td><span class="highlighted" id="h-health">3,048</span></td><td>Days of steps data</td></tr>
</table>

Naturally after I started collecting this data, I wanted to visualize what I was learning, so I created this page. Initially, the domain [`whereisFelix.today`](https://howisFelix.today) (now renamed to [`howisFelix.today`](https://howisFelix.today)) started as a joke to respond to friends asking when I'd be back in NYC or San Francisco. Rather than send them my schedule, I'd point them to this domain. However, now it's more than my location: it's all of me.

**Rules I setup for the project**

- Use a single database, owned and hosted by me, with all the data I've collected over the years
- Be able to [easily add and remove questions](https://github.com/KrauseFx/FxLifeSheet/blob/master/lifesheet.json) on the fly, as I learn what's beneficial to track
- Full control of how the data is visualized
- Works well for frequent flyers with mixed time zones
- 100% [fully open source](https://github.com/KrauseFx/FxLifeSheet), MIT licensed and self-hosted

I selected <span class="highlighted">48</span> graphs to show publicly on this page. For privacy reasons, and to prevent any accidental data leaks, the graphs below are snapshots taken on a given day.

<p class="graph-overview-footer">
  Note: some graphs were created using third party services or apps, however most were generated using my <a href="https://github.com/KrauseFx/FxLifeSheet/tree/master/visual_playground">own visualization code</a> using plotly.js.
</p>



<div id="graphs-container">
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-number-of-entries">
      <h3>Number of Data Entries over Time</h3>
      <p class="graph-description">
        Visualization of the number of data entries in <a href='https://github.com/KrauseFx/FxLifeSheet' target='_blank'>FxLifeSheet</a> over the last 10 years, and where the data came from.
      </p>

      <ul>
        
          <li>Initially (2014) the only data used was <a href='https://rescuetime.com' target='_blank'>RescueTime</a> and <a href='https://swarmapp.com' target='_blank'>Foursquare Swarm</a> location data</li>
        
          <li>Once I started the <a href='https://github.com/KrauseFx/FxLifeSheet' target='_blank'>FxLifeSheet project</a> in April 2019, I manually tracked <span class='highlighted'>75 data entries every day</span>, ranging from mood, sleep, social life, to fitness data</li>
        
          <li>I was able to retrospectively fetch the historic weather data based on my location on a given day</li>
        
          <li>I also implemented other import sources, like fetching my historic weight and the number of steps from Apple Health</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        
      </span>

      <div class="image-container">
        <img src="graphs/screens/number-of-entries.png" class="image-link" alt="Number of Data Entries over Time" onclick="enlargeImage(this, 'graphs/screens/number-of-entries.png', 'Number of Data Entries over Time')" />
      </div>

      <span class="graph-date">10 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-happy-mood-entries-buckets">
      <h3>Days tracked my Mood to be Happy & Excited</h3>
      <p class="graph-description">
        On days where I tracked my mood to be "happy" & "excited", the following other factors of my life were affected
      </p>

      <ul>
        
          <li>50% more likely to have pushed my comfort zone</li>
        
          <li>44% more likely to have meditated that day</li>
        
          <li>33% more excited about what's ahead in the future</li>
        
          <li>31% more likely to drink alcohol that day (parties, good friends and such)</li>
        
          <li>28% more time spent reading or listening to audio books</li>
        
          <li>26% more likely to have worked on interesting technical challenges</li>
        
          <li>26% warmer temperature that day</li>
        
          <li>20% more likely to have learned something new that day</li>
        
          <li><span class='highlighted'>45% less time spent in video & audio calls that day</span></li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Manually
      </span>

      <div class="image-container">
        <img src="graphs/screens/happy-mood-entries-buckets.png" class="image-link" alt="Days tracked my Mood to be Happy & Excited" onclick="enlargeImage(this, 'graphs/screens/happy-mood-entries-buckets.png', 'Days tracked my Mood to be Happy & Excited')" />
      </div>

      <span class="graph-date">2.5 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-jetlovers-stats">
      <h3>Flying Stats - General</h3>
      <p class="graph-description">
        All flights taken within the last 7 years, tracked using <a href='https://swarmapp.com' target='_blank'>Foursquare Swarm</a>, analyzed by <a href='https://www.jetlovers.com/' target='_blank'>JetLovers</a>.
      </p>

      <ul>
        
          <li>The stats clearly show the impact of COVID starting 2020</li>
        
          <li>Sunday has been my "commute" day, flying between San Francisco, New York City and Vienna</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        JetLovers, Swarm
      </span>

      <div class="image-container">
        <img src="graphs/screens/jetlovers-stats.png" class="image-link" alt="Flying Stats - General" onclick="enlargeImage(this, 'graphs/screens/jetlovers-stats.png', 'Flying Stats - General')" />
      </div>

      <span class="graph-date">7 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-jetlovers-top">
      <h3>Flying Stats - Top</h3>
      <p class="graph-description">
        All flights taken within the last 7 years, tracked using <a href='https://swarmapp.com' target='_blank'>Foursquare Swarm</a>, analyzed by <a href='https://www.jetlovers.com/' target='_blank'>JetLovers</a>.
      </p>

      <ul>
        
          <li>Frankfurt - Vienna was the flight connecting me with most US airports</li>
        
          <li>Germany is high up on the list due to layovers, even though I didn't spend actually much time there</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        JetLovers, Swarm
      </span>

      <div class="image-container">
        <img src="graphs/screens/jetlovers-top.png" class="image-link" alt="Flying Stats - Top" onclick="enlargeImage(this, 'graphs/screens/jetlovers-top.png', 'Flying Stats - Top')" />
      </div>

      <span class="graph-date">7 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-life-in-weeks">
      <h3>My Life in Weeks</h3>
      <p class="graph-description">
        Inspired by <a href='https://waitbutwhy.com/2014/05/life-weeks.html' target='_blank'>Your Life in Weeks by WaitButWhy</a>, I use Google Sheets to visualize every week of my life, with little notes on what city/country I was in, and other life events that have happened.
      </p>

      <ul>
        
          <li>The first 14 years I didn't really get much done</li>
        
          <li>I can highly recommend taking a few weeks (or even months) off between jobs (if you have the possibility)</li>
        
          <li>Yellow area indicates no active full-time employment</li>
        
          <li>Shades of blue indicate my full-time employments</li>
        
          <li>Shades of red is formal education</li>
        
          <li>You can create your own version using <a href='https://www.producthunt.com/posts/your-life-in-weeks' target='_blank'>my template</a></li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Manually
      </span>

      <div class="image-container">
        <img src="graphs/screens/life-in-weeks.png" class="image-link" alt="My Life in Weeks" onclick="enlargeImage(this, 'graphs/screens/life-in-weeks.png', 'My Life in Weeks')" />
      </div>

      <span class="graph-date">27.5 years of data - Last updated on 2022-02-24</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-daily-steps">
      <h3>Average Daily Steps over Time</h3>
      <p class="graph-description">
        Average daily steps measured through the iPhone's Apple Health app. I decided against using SmartWatch data for steps, as SmartWatches have changed over the last 8 years.
      </p>

      <ul>
        
          <li>I walked a total of <span class='highlighted'>22,830,860</span> steps over last 8 years</li>
        
          <li>I walk more than twice as much when I'm in New York, compared to any other city</li>
        
          <li>In NYC I had the general rule of thumb to walk instead of taking public transit whenever it's less than 40 minutes. I used that time to call friends & family, or listen to audio books</li>
        
          <li>Although Vienna is very walkable, the excellent public transit system with subway trains coming every 3-5 minutes, has caused me to walk less</li>
        
          <li>San Francisco was always scary to walk</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Apple Health
      </span>

      <div class="image-container">
        <img src="graphs/screens/daily-steps.png" class="image-link" alt="Average Daily Steps over Time" onclick="enlargeImage(this, 'graphs/screens/daily-steps.png', 'Average Daily Steps over Time')" />
      </div>

      <span class="graph-date">8 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-weight-sleeping-hr">
      <h3>Correlation: Weight and Resting Heart Rate</h3>
      <p class="graph-description">
        This graph clearly shows the correlation between my body weight and my sleeping/resting heart rate. The resting heart rate is measured by the <a href='https://www.withings.com/us/en/scanwatch' target='_blank'>Withings ScanWatch</a> while sleeping, and indicates how hard your heart has to work while not being active. Generally the lower the resting heart rate, the better.
      </p>

      <ul>
        
          <li>I started my lean bulk (controlled weight gain combined with 5 workouts a week) in August 2020</li>
        
          <li>My resting heart rate went from 58bpm to 67bpm (<span class='highlighted'>+9bpm</span>) from August 2020 to March 2021 with a weight gain of <span class='highlighted'>+8.5kg</span> (+19lbs) as part of a controlled lean-bulk combined with a 5-day/week workout routine</li>
        
          <li>The spike in resting heart rate in July & August 2021 was due to bars and nightclubs opening up again in Austria</li>
        
          <li>After a night of drinking, my resting/sleeping heart rate was about 50% higher than after a night without any alcohol</li>
        
          <li>The spike in resting heart rate in Oct/Nov/Dec 2021 was due to having bronchitis and a cold/flu, not getting correct treatment early enough</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Withings ScanWatch, Withings Scale
      </span>

      <div class="image-container">
        <img src="graphs/screens/weight-sleeping-hr.png" class="image-link" alt="Correlation: Weight and Resting Heart Rate" onclick="enlargeImage(this, 'graphs/screens/weight-sleeping-hr.png', 'Correlation: Weight and Resting Heart Rate')" />
      </div>

      <span class="graph-date">1.5 years of data - Last updated on 2022-02-09</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-healthy">
      <h3>How healthy have I been over the Years?</h3>
      <p class="graph-description">
        Every day I answered the question on how healthy I felt. In the graph, the yellow color indicates that I felt a little under the weather, not sick per se. Red means I was sick and had to stay home. Green means I felt energized and healthy.
      </p>

      <ul>
        
          <li>During the COVID lockdowns I tended to stay healthier. This may be due to not going out, no heavy drinking, less close contact with others, etc. which resulted in me having better sleep.</li>
        
          <li>Usually during excessive traveling I get sick (cold/flu)</li>
        
          <li>Q4 2021 I had bronchitis, however, I didn't know about it at the time and didn't get proper treatment</li>
        
          <li>Overall I'm quite prone to getting sick (cold/flu)</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Manually
      </span>

      <div class="image-container">
        <img src="graphs/screens/healthy.png" class="image-link" alt="How healthy have I been over the Years?" onclick="enlargeImage(this, 'graphs/screens/healthy.png', 'How healthy have I been over the Years?')" />
      </div>

      <span class="graph-date">2.5 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-alcoholic-beverages-buckets">
      <h3>Days with more than 4 Alcoholic Drinks</h3>
      <p class="graph-description">
        On days where I had more than 4 alcoholic beverages (meaning I was partying), the following other factors were affected
      </p>

      <ul>
        
          <li>21x more likely to dance</li>
        
          <li>80% more likely to take a nap the day of, or the day after</li>
        
          <li>60% more time spent with friends</li>
        
          <li>40% warmer temperatures, and 40% less precipitation. There weren't many opportunities for parties in Winter due to lockdowns in the last 2 years. Also, people are more motivated to go out when it's nice outside.</li>
        
          <li>25% more steps</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Manually
      </span>

      <div class="image-container">
        <img src="graphs/screens/alcoholic-beverages-buckets.png" class="image-link" alt="Days with more than 4 Alcoholic Drinks" onclick="enlargeImage(this, 'graphs/screens/alcoholic-beverages-buckets.png', 'Days with more than 4 Alcoholic Drinks')" />
      </div>

      <span class="graph-date">2.5 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-mood-graph-months">
      <h3>Mood over the Years</h3>
      <p class="graph-description">
        My <a href='https://github.com/KrauseFx/FxLifeSheet' target='_blank'>FxLifeSheet bot</a> asks me 4 times a day how I'm feeling at the moment.
      </p>

      <ul>
        
          <li>This graph groups the entries by month, and shows the % of entries for each value (0 - 5) with 5 being very excited, and 0 being worried.</li>
        
          <li>I designed the ranges so that 0 or 5 are not entered as much. 0 is rendered as dark green at the top, whereas 5 is rendered as light green at the bottom.</li>
        
          <li>For privacy reasons I won't get into some of the details on why certain months were worse than others.</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Manually
      </span>

      <div class="image-container">
        <img src="graphs/screens/mood-graph-months.png" class="image-link" alt="Mood over the Years" onclick="enlargeImage(this, 'graphs/screens/mood-graph-months.png', 'Mood over the Years')" />
      </div>

      <span class="graph-date">4 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-map-world-trip">
      <h3>Location History</h3>
      <p class="graph-description">
        Every <a href='https://swarmapp.com' target='_blank'>Swarm</a> check-in over the last 7 years visualized on a map, including the actual trip (flight, drive, etc.)
      </p>

      <ul>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Swarm
      </span>

      <div class="image-container">
        <img src="graphs/screens/map-world-trip.jpg" class="image-link" alt="Location History" onclick="enlargeImage(this, 'graphs/screens/map-world-trip.jpg', 'Location History')" />
      </div>

      <span class="graph-date">7 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-map-combined">
      <h3>Location History - Cities</h3>
      <p class="graph-description">
        Every <a href='https://swarmapp.com' target='_blank'>Swarm</a> check-in over the last 7 years visualized, zoomed in
      </p>

      <ul>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Swarm
      </span>

      <div class="image-container">
        <img src="graphs/screens/map-combined.jpg" class="image-link" alt="Location History - Cities" onclick="enlargeImage(this, 'graphs/screens/map-combined.jpg', 'Location History - Cities')" />
      </div>

      <span class="graph-date">7 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-swarm-cities-top">
      <h3>Most visited POIs, grouped by City</h3>
      <p class="graph-description">
        Each time I did a check-in at a place (e.g. Coffee, Restaurant, Airport, Gym) on <a href='https://swarmapp.com' target='_blank'>Foursquare Swarm</a> at a given city, this is tracked as a single entry.
      </p>

      <ul>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Swarm
      </span>

      <div class="image-container">
        <img src="graphs/screens/swarm-cities-top.png" class="image-link" alt="Most visited POIs, grouped by City" onclick="enlargeImage(this, 'graphs/screens/swarm-cities-top.png', 'Most visited POIs, grouped by City')" />
      </div>

      <span class="graph-date">7 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-swarm-cities-top-years">
      <h3>Number of Check-Ins grouped by City</h3>
      <p class="graph-description">
        Each check-in at a given city is counted as a single entry, grouped by years
      </p>

      <ul>
        
          <li>2016 and 2017 I lived in San Francisco</li>
        
          <li>2018 and 2019 I lived in New York City</li>
        
          <li>2020 and 2021 I lived in Vienna</li>
        
          <li>The longer it's been since I moved away from Austria, the more time I actually spent back home in Austria for visits and vacations</li>
        
          <li>2020 clearly shows the impact of COVID</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Swarm
      </span>

      <div class="image-container">
        <img src="graphs/screens/swarm-cities-top-years.png" class="image-link" alt="Number of Check-Ins grouped by City" onclick="enlargeImage(this, 'graphs/screens/swarm-cities-top-years.png', 'Number of Check-Ins grouped by City')" />
      </div>

      <span class="graph-date">7 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-swarm-categories-top">
      <h3>Top Categories of Check-Ins</h3>
      <p class="graph-description">
        Each check-in at a given category is tracked, and summed up over the last years
      </p>

      <ul>
        
          <li>In 2020 and 2021, check-ins at Offices went down to zero due to COVID, and a distributed work setup</li>
        
          <li><span class='highlighted'>Airports being the #4 most visited category</span> was a surprise, but is accurate. A total of 403 airport check-ins, whereas a flight with a layover would count as 3 airport check-ins</li>
        
          <li>Earlier in my life, I didn't always check into 'commute' places like public transit and super markets</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Swarm
      </span>

      <div class="image-container">
        <img src="graphs/screens/swarm-categories-top.png" class="image-link" alt="Top Categories of Check-Ins" onclick="enlargeImage(this, 'graphs/screens/swarm-categories-top.png', 'Top Categories of Check-Ins')" />
      </div>

      <span class="graph-date">7 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-swarm-number-of-checkins">
      <h3>Number of Swarm Check-Ins over Time</h3>
      <p class="graph-description">
        Number of <a href='https://swarmapp.com' target='_blank'>Foursquare Swarm</a> check-ins on each quarter over the last 10 years. I didn't use Foursquare Swarm as seriously before 2015. Once I moved to San Francisco in Q3 2015 I started my habit of checking into every point of interest (POI) I visit.
      </p>

      <ul>
        
          <li>Q3 2015 I moved to San Francisco, however I couldn't use Swarm yet, since my move was a secret until the official announced at the Twitter Flight conference</li>
        
          <li>Q2 2020 clearly shows the impact of COVID with Q3 already being open in Austria</li>
        
          <li>Q3 2021 the vaccine was already widely available and I was able to travel/visit more again</li>
        
          <li>My time in New York was the most active when it comes to check-ins. When I'm in NYC, I tend to eat/drink out more, and grab to-go food, which I do way less in Vienna</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Swarm
      </span>

      <div class="image-container">
        <img src="graphs/screens/swarm-number-of-checkins.png" class="image-link" alt="Number of Swarm Check-Ins over Time" onclick="enlargeImage(this, 'graphs/screens/swarm-number-of-checkins.png', 'Number of Swarm Check-Ins over Time')" />
      </div>

      <span class="graph-date">10 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-map-world-overview">
      <h3>Location History</h3>
      <p class="graph-description">
        Every <a href='https://swarmapp.com' target='_blank'>Swarm</a> check-in visualized on a map. Only areas where I've had multiple check-ins are rendered.
      </p>

      <ul>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Swarm
      </span>

      <div class="image-container">
        <img src="graphs/screens/map-world-overview.jpg" class="image-link" alt="Location History" onclick="enlargeImage(this, 'graphs/screens/map-world-overview.jpg', 'Location History')" />
      </div>

      <span class="graph-date">7 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-lockdown-days">
      <h3>Days in Full Lockdown</h3>
      <p class="graph-description">
        Number of days per year that I've spent in full lockdown, meaning restaurants, bars and non-essential stores were closed.
      </p>

      <ul>
        
          <li>I escaped parts of the Austrian lockdown by spending time in the US when I was already vaccinated</li>
        
          <li>Surprisingly 2021 I spent more days in a full lockdown than in 2020, even with vaccines available</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Manually
      </span>

      <div class="image-container">
        <img src="graphs/screens/lockdown-days.png" class="image-link" alt="Days in Full Lockdown" onclick="enlargeImage(this, 'graphs/screens/lockdown-days.png', 'Days in Full Lockdown')" />
      </div>

      <span class="graph-date">3 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-lockdowns">
      <h3>Effects of the Lockdowns</h3>
      <p class="graph-description">
        How was my life affected by the recent COVID lockdowns? As lockdown day I classify every day where places like restaurants, gyms and non-essential stores were closed.
      </p>

      <ul>
        
          <li>200% more time spent in audio & video calls with friends (non-work related)</li>
        
          <li>60% more likely to follow my meal plan (macros & calories)</li>
        
          <li>24% less steps</li>
        
          <li>40% less alcoholic beverages</li>
        
          <li>50% colder temperatures: Lockdowns tended to happen in Autumn and Winter</li>
        
          <li>100% less likely to dance</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Manually
      </span>

      <div class="image-container">
        <img src="graphs/screens/lockdowns.png" class="image-link" alt="Effects of the Lockdowns" onclick="enlargeImage(this, 'graphs/screens/lockdowns.png', 'Effects of the Lockdowns')" />
      </div>

      <span class="graph-date">3 years of data - Last updated on 2022-05-23</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-manual-alcohol-intake">
      <h3>Alcoholic Beverages per Day</h3>
      <p class="graph-description">
        Alcoholic drinks per day. Days with no data are rendered as white
      </p>

      <ul>
        
          <li>Friday and Saturday nights are clearly visible on those graphs</li>
        
          <li>2021 and summer/winter of 2019 also show the Wednesday night party in Vienna</li>
        
          <li>Q2 and Q4 2020 clearly show the COVID lockdowns, as well as Q2 2021</li>
        
          <li>Summer of 2021 all bars and dance clubs were open in Vienna</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Manually
      </span>

      <div class="image-container">
        <img src="graphs/screens/manual-alcohol-intake.png" class="image-link" alt="Alcoholic Beverages per Day" onclick="enlargeImage(this, 'graphs/screens/manual-alcohol-intake.png', 'Alcoholic Beverages per Day')" />
      </div>

      <span class="graph-date">2.5 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-manual-alcohol-intake-months">
      <h3>Alcoholic Beverages per Month</h3>
      <p class="graph-description">
        Each bar represents a month, the graph shows the number of alcoholic beverages on a given day, with '5' being 5 or more drinks. 
      </p>

      <ul>
        
          <li>Unless there are social obligations, or there is a party, I usually drink 0 alcoholic drinks</li>
        
          <li>In May 2019, the 0 bar is rendered to be 55%, meaning on 55% days of that month I didn't drink any alcohol</li>
        
          <li>July 2021 Austrian night life was fully open up again, with most COVID restrictions being lifted</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Manually
      </span>

      <div class="image-container">
        <img src="graphs/screens/manual-alcohol-intake-months.png" class="image-link" alt="Alcoholic Beverages per Month" onclick="enlargeImage(this, 'graphs/screens/manual-alcohol-intake-months.png', 'Alcoholic Beverages per Month')" />
      </div>

      <span class="graph-date">2.5 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-manual-gym-visits">
      <h3>Gym Workouts</h3>
      <p class="graph-description">
        Each green square represents a strength-workout in the gym, I try my best to purchase day passes at gyms while traveling
      </p>

      <ul>
        
          <li>During the first lockdown in Q2 2020 I didn't have access to a gym, and didn't count home-workouts as gym visits</li>
        
          <li>In 2021 I tended to work out less often on Sunday, the day I visit my family's place</li>
        
          <li>In Q4 2021 I had a cold for a longer time</li>
        
          <li>I went from <span class='highlighted'>~50 workouts per year</span> in 2014/2015 to <span class='highlighted'>~200 per year</span> in 2018 - 2021</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Manually
      </span>

      <div class="image-container">
        <img src="graphs/screens/manual-gym-visits.png" class="image-link" alt="Gym Workouts" onclick="enlargeImage(this, 'graphs/screens/manual-gym-visits.png', 'Gym Workouts')" />
      </div>

      <span class="graph-date">7 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-macros">
      <h3>Food Tracking - Macros</h3>
      <p class="graph-description">
        On weeks where I have a routine (not traveling), I track most of my meals. Whenever I scale my food, I try to guess the weight before to become better at estimating portion sizes.
      </p>

      <ul>
        
          <li>Tracking macros precisely is the best way to lose/gain weight at a controlled and healthy speed</li>
        
          <li>When I first started tracking macros, additionally to already working out regularly, is when I started seeing the best results</li>
        
          <li>It takes a lot of energy to be consistent, but it's worth it, and it will make you feel good (e.g. feeling good about using a spoon to eat plain Nutella if you have the macros left)</li>
        
          <li>Gaining and losing weight is pretty much math, if you track your macros and body weight</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        MyFitnessPal
      </span>

      <div class="image-container">
        <img src="graphs/screens/macros.png" class="image-link" alt="Food Tracking - Macros" onclick="enlargeImage(this, 'graphs/screens/macros.png', 'Food Tracking - Macros')" />
      </div>

      <span class="graph-date">4 years of data - Last updated on 2022-02-17</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-manual-gym-visits-grouped">
      <h3>Gym Workouts grouped by Month</h3>
      <p class="graph-description">
        Percentage of days I went to the gym
      </p>

      <ul>
        
          <li>During the first lockdown in Q2 2020 I didn't have access to a gym, and didn't count home-workouts as gym visits</li>
        
          <li>Generally I hit the gym 5 days a week when I'm at home, but as shown on this graph it ends up being less, mostly due to traveling, and occasional colds</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Manually
      </span>

      <div class="image-container">
        <img src="graphs/screens/manual-gym-visits-grouped.png" class="image-link" alt="Gym Workouts grouped by Month" onclick="enlargeImage(this, 'graphs/screens/manual-gym-visits-grouped.png', 'Gym Workouts grouped by Month')" />
      </div>

      <span class="graph-date">2.5 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-fitness-bulk">
      <h3>Fitness Bulk & Cut Season</h3>
      <p class="graph-description">
        I'm following a pretty regular bodybuilding routine, a 3-day workout split, and a normal bulk & cut seasons
      </p>

      <ul>
        
          <li>During times where fitness isn't high up on my priority list, I usually switch to maintenance</li>
        
          <li>During a cut, I have a slight kcal deficit of about 300kcal, to not negatively impact the rest of my life</li>
        
          <li>During the bulk, I aim to gain about 300g of body weight per week</li>
        
          <li>During a cut, I tend to track my meals more precisely than during a bulk or while maintaining</li>
        
          <li>Purple = Cut</li>
        
          <li>Brown = Bulk</li>
        
          <li>Blue = Maintenance</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Manually
      </span>

      <div class="image-container">
        <img src="graphs/screens/fitness-bulk.png" class="image-link" alt="Fitness Bulk & Cut Season" onclick="enlargeImage(this, 'graphs/screens/fitness-bulk.png', 'Fitness Bulk & Cut Season')" />
      </div>

      <span class="graph-date">4 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-weight-graph-1">
      <h3>Weight History of 8 Years</h3>
      <p class="graph-description">
        Historic weight, clearly showing the various bulks and cuts I've made over the years. Only the last 5 years are rendered in this graph, with the last 3 years having tracked my weight way more frequently.
      </p>

      <ul>
        
          <li>Lowest recorded weight was <span class='highlighted'>69kg</span>/152lbs in 2014 (age 20)</li>
        
          <li>Highest recorded weight was <span class='highlighted'>89.8kg</span>/198lbs in 2021 (age 27)</li>
        
          <li>I gained 20kg (44lbs) while staying under 12% body fat on 193cm (6"4)</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Withings Scale
      </span>

      <div class="image-container">
        <img src="graphs/screens/weight-graph-1.png" class="image-link" alt="Weight History of 8 Years" onclick="enlargeImage(this, 'graphs/screens/weight-graph-1.png', 'Weight History of 8 Years')" />
      </div>

      <span class="graph-date">9 years of data - Last updated on 2022-02-17</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-weight-graph-2">
      <h3>Bulk & Cut Phase visualized</h3>
      <p class="graph-description">
        I usually have slow bulk & cut phases, where I gain and lose weight at a controlled speed, combined with tracking my meals.
      </p>

      <ul>
        
          <li>I only track my weight if I didn't drink alcohol the night before, and after getting a full night's sleep, since otherwise the weight fluctuates a lot</li>
        
          <li>Generally, you can pretty much calculate your weight on a certain date, if you consistently track your meals, and correctly calculated your macros</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Withings Scale, weightgrapher.com
      </span>

      <div class="image-container">
        <img src="graphs/screens/weight-graph-2.png" class="image-link" alt="Bulk & Cut Phase visualized" onclick="enlargeImage(this, 'graphs/screens/weight-graph-2.png', 'Bulk & Cut Phase visualized')" />
      </div>

      <span class="graph-date">1 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-daily-steps-buckets">
      <h3>Days with more than 15,000 Steps</h3>
      <p class="graph-description">
        On days where I walked more than 15,000 steps (I walked an average of 9200 steps a day the last 3 years)
      </p>

      <ul>
        
          <li>60% more time spent with friends in real-life</li>
        
          <li>50% warmer temperature that day</li>
        
          <li>40% more likely to socialize with friends before going to sleep</li>
        
          <li>35% more likely to be a weekend day</li>
        
          <li>20% less likely to take a nap that day</li>
        
          <li>30% less likely to correctly track my macros (food)</li>
        
          <li>30% less time spent in work calls/meetings</li>
        
          <li>30% less anxious</li>
        
          <li>40% less likely to meditate</li>
        
          <li>40% less time spent writing code</li>
        
          <li>40% less likely to be a rainy day</li>
        
          <li>50% less time spent watching TV</li>
        
          <li>95% less likely to be a day in lockdown</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Apple Health
      </span>

      <div class="image-container">
        <img src="graphs/screens/daily-steps-buckets.png" class="image-link" alt="Days with more than 15,000 Steps" onclick="enlargeImage(this, 'graphs/screens/daily-steps-buckets.png', 'Days with more than 15,000 Steps')" />
      </div>

      <span class="graph-date">2.5 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-sleep-duration-buckets">
      <h3>How does longer Sleep Duration affect my Day?</h3>
      <p class="graph-description">
        On days where I had a total sleep duration of more than 8.5 hours, the following other factors were affected
      </p>

      <ul>
        
          <li>65% more likely to have cold symptoms</li>
        
          <li>60% more likely to have headache</li>
        
          <li>40% more social media usage</li>
        
          <li>30% more likely to be a rainy day</li>
        
          <li>20% more likely to be a weekend</li>
        
          <li>15% more likely to have bad/stressful dreams</li>
        
          <li>12% colder weather on average</li>
        
          <li>20% less likely to hit the gym that day</li>
        
          <li>24% less energy</li>
        
          <li>Overall, it's clear that <span class='highlighted'>days with a longer sleep duration tend to be significantly worse days</span>. This is most likely due to the fact that I try to sleep a maximum of 8 hours, and if I actually sleep more than usually because I'm not feeling too well, or even sick</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Withings Scan Watch, Manually
      </span>

      <div class="image-container">
        <img src="graphs/screens/sleep-duration-buckets.png" class="image-link" alt="How does longer Sleep Duration affect my Day?" onclick="enlargeImage(this, 'graphs/screens/sleep-duration-buckets.png', 'How does longer Sleep Duration affect my Day?')" />
      </div>

      <span class="graph-date">2.5 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-air-quality">
      <h3>Air Quality in various Rooms</h3>
      <p class="graph-description">
        I used the <a href='https://www.getawair.com/products/element' target='_blank'>Awair Element</a> at home in Vienna, in every room over multiple days
      </p>

      <ul>
        
          <li>Overall, the air quality in Vienna is excellent. Just opening the windows for a few minutes is enough to get a 100% air score</li>
        
          <li>The smaller the room, the more difficult it is to keep the CO2 levels low</li>
        
          <li>The only room where I'm still struggling to keep the CO2 levels low is my bedroom, which is small, and doesn't offer any ventilation. Keeping the doors open mostly solves the high CO2 levels, however comes with other downsides like more light</li>
        
          <li>Probably obvious for many, but <span class='highlighted'>I didn't realize ACs don't transport any air into the room, but just moves it around</span></li>
        
          <li>Opening the windows for a longer time in winter will cause low humidity and low temperatures, so it's often not a good option</li>
        
          <li>I didn't learn anything from the Chemicals and PM2.5 graphs</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Awair
      </span>

      <div class="image-container">
        <img src="graphs/screens/air-quality.png" class="image-link" alt="Air Quality in various Rooms" onclick="enlargeImage(this, 'graphs/screens/air-quality.png', 'Air Quality in various Rooms')" />
      </div>

      <span class="graph-date">1 years of data - Last updated on 2022-02-09</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-rescue-time-usage-categories">
      <h3>RescueTime Computer Usage Categories</h3>
      <p class="graph-description">
        Using <a href='https://rescuetime.com' target='_blank'>RescueTime</a>, I tracked my exact computer usage, and the categories of activities in which I spend time with.
      </p>

      <ul>
        
          <li>Over the last 7 years, I went from spending around 10% of my time with communication & scheduling, to around 30%</li>
        
          <li>Over the last 7 years, I went from spending around 33% of my time with software development, to less than 10% (now that I'm not currently full-time employed)</li>
        
          <li>All other categories stayed pretty much on the same ratio over the last 7 years</li>
        
          <li>Total amount spent on my computer went down significantly after having left my job late 2021</li>
        
          <li>Tracking productivity levels automatically is difficult to get right, as it's hard to classify certain activities.</li>
        
          <li>Day to day I'm using <a href='https://timingapp.com/' target='_blank'>Timing App</a> which has a significantly nicer user experience, however I just had many additional years of data on RescueTime</li>
        
          <li><span class='highlighted'>I highly recommend every professional to track their computer usage.</span> It's surprising to see how many hours a day you actually get on your computer, and how much of it is considered productive. <a href='https://timingapp.com/' target='_blank'>Timing App</a> shows the amount of hours of the day on the Mac's status bar, and it's been extremely useful</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        RescueTime
      </span>

      <div class="image-container">
        <img src="graphs/screens/rescue-time-usage-categories.png" class="image-link" alt="RescueTime Computer Usage Categories" onclick="enlargeImage(this, 'graphs/screens/rescue-time-usage-categories.png', 'RescueTime Computer Usage Categories')" />
      </div>

      <span class="graph-date">7 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-socializing">
      <h3>Happy with my Social Life?</h3>
      <p class="graph-description">
        Each day I answer the question "Do you feel like you need to socialize more?" with the answers ranging from "No" (5 = light green) to "Feeling isolated" (0 = dark green)
      </p>

      <ul>
        
          <li>Since I'm an Introvert, I tend to feel like I socialize too much, rather than too little</li>
        
          <li><span class='highlighted'>Early COVID lockdown March 2020 was still good, however as the lockdown continued I started feeling more isolated</span>, also partly because I was single, and had just moved to Vienna</li>
        
          <li>End of July 2020 Austria opened up most social venues again, bringing this graph back to an ideal level again</li>
        
          <li>December 2019 I was working remotely from Buenos Aires, however I got a pretty bad cold. Being sick in a remote city wasn't the best feeling</li>
        
          <li>Over the last 2.5 years I actually never felt isolated, therefore there is no entry for the value '0'</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Manually
      </span>

      <div class="image-container">
        <img src="graphs/screens/socializing.png" class="image-link" alt="Happy with my Social Life?" onclick="enlargeImage(this, 'graphs/screens/socializing.png', 'Happy with my Social Life?')" />
      </div>

      <span class="graph-date">2.5 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-season-summer">
      <h3>How does Summer affect my Life?</h3>
      <p class="graph-description">
        Summer (being from 21st June to 23rd September) has the following effects on my life:
      </p>

      <ul>
        
          <li>Chances of taking a freezing cold shower goes up by 100%</li>
        
          <li>I walk 33% more steps in Summer</li>
        
          <li>30% more likely to hit the gym</li>
        
          <li>23% more alcoholic beverages (there were no lockdowns in Summer)</li>
        
          <li>20% less time spent reading news & opinions (as per <a href='https://rescuetime.com' target='_blank'>RescueTime</a> classifications)</li>
        
          <li><span class='highlighted'>40% less likely to be sick</span></li>
        
          <li>40% less time spent on Entertainment on my computer (as per <a href='https://rescuetime.com' target='_blank'>RescueTime</a> classifications)</li>
        
          <li>30% less likely to have bad dreams</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Manually
      </span>

      <div class="image-container">
        <img src="graphs/screens/season-summer.png" class="image-link" alt="How does Summer affect my Life?" onclick="enlargeImage(this, 'graphs/screens/season-summer.png', 'How does Summer affect my Life?')" />
      </div>

      <span class="graph-date">2.5 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-season-winter">
      <h3>How does Winter affect my Life?</h3>
      <p class="graph-description">
        Winter (being from 21st Dec to 20th March) has the following effects on my life:
      </p>

      <ul>
        
          <li>Online shopping time going up by 100%, most likely due to the last Winters having spent in lockdown</li>
        
          <li><span class='highlighted'>Chances of having cold symptoms goes up 45%</span></li>
        
          <li>35% less likely to take a freezing cold shower</li>
        
          <li>Unsurprisingly, I'm exposed to less solar energy in Winter, make sure to get enough Vitamin D</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Manually
      </span>

      <div class="image-container">
        <img src="graphs/screens/season-winter.png" class="image-link" alt="How does Winter affect my Life?" onclick="enlargeImage(this, 'graphs/screens/season-winter.png', 'How does Winter affect my Life?')" />
      </div>

      <span class="graph-date">2.5 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-countries">
      <h3>Days spent in each Country</h3>
      <p class="graph-description">
        Percentage of days spent in each country over the last 4 years
      </p>

      <ul>
        
          <li>International travel has slowed down significantly since COVID started</li>
        
          <li>Traveling without a stop home for more than 3 weeks gets exhausting for me, I prefer doing more shorter travels instead</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Manually
      </span>

      <div class="image-container">
        <img src="graphs/screens/countries.png" class="image-link" alt="Days spent in each Country" onclick="enlargeImage(this, 'graphs/screens/countries.png', 'Days spent in each Country')" />
      </div>

      <span class="graph-date">4 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-manual-living-style-years">
      <h3>Living Style</h3>
      <p class="graph-description">
        From late 2017 to early 2020 I lived without a permanent home address as a <a href='https://krausefx.com/blog/one-year-nomad' target='_blank'>digital nomad</a>, staying at various Airbnbs or taking over a few weeks of a lease from a friend
      </p>

      <ul>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Manually
      </span>

      <div class="image-container">
        <img src="graphs/screens/manual-living-style-years.png" class="image-link" alt="Living Style" onclick="enlargeImage(this, 'graphs/screens/manual-living-style-years.png', 'Living Style')" />
      </div>

      <span class="graph-date">7 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-cold-symptoms">
      <h3>Cold Symptoms</h3>
      <p class="graph-description">
        Number of days each month where I had cold symptoms (dark green [1] = day with cold symptoms), which I classify as having a runny nose, feeling light-headed or having light ear pain.
      </p>

      <ul>
        
          <li>Clearly the cold months of 2019 and 2021 I had significantly more days with cold symptoms</li>
        
          <li>Winter 2021 was a full lockdown, I assume this reduced the risk of getting sick</li>
        
          <li>October and November 2021 I had bronchitis, without getting the proper treatment</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Manually
      </span>

      <div class="image-container">
        <img src="graphs/screens/cold-symptoms.png" class="image-link" alt="Cold Symptoms" onclick="enlargeImage(this, 'graphs/screens/cold-symptoms.png', 'Cold Symptoms')" />
      </div>

      <span class="graph-date">2.5 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-day-is-weekend">
      <h3>What are Weekends like?</h3>
      <p class="graph-description">
        Weekends for me naturally involve less working time, more going out, and social interactions, as well as visiting family members outside the city
      </p>

      <ul>
        
          <li>150% more computer time spent on "Entertainment", which is mostly coming from occasionally playing <a href='https://www.factorio.com/' target='_blank'>Factorio</a> or Age Of Empires with friends at LAN-Party gatherings</li>
        
          <li>74% more time spent watching TV</li>
        
          <li>60% more time spent with friends IRL</li>
        
          <li>55% more dancing, when going out</li>
        
          <li>45% more likely to have a headache, due to drinking alcohol the night before</li>
        
          <li>25% less gym visits, mostly due to family gatherings and not being in the city</li>
        
          <li>45% less likely to properly track my macros/meals, mostly due to family gatherings and going out</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Manually
      </span>

      <div class="image-container">
        <img src="graphs/screens/day-is-weekend.png" class="image-link" alt="What are Weekends like?" onclick="enlargeImage(this, 'graphs/screens/day-is-weekend.png', 'What are Weekends like?')" />
      </div>

      <span class="graph-date">2.5 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-weather-temperature-max">
      <h3>Maximal Temperature each Day of the Year</h3>
      <p class="graph-description">
        Historic weather data based on the location I was at on that day based on my Swarm check-ins. Days with no data are rendered as white
      </p>

      <ul>
        
          <li>Turns out, days in Summer are warmer than days in Winter</li>
        
          <li>Summer 2019 I spent in New York City, while 2020 and 2021 I spent in Vienna. The graph shows the summer in NYC to reach higher temperatures</li>
        
          <li>Week 36 in 2021 I spent in Iceland, therefore significantly lower temperatures</li>
        
          <li>End of November in 2019 I spent in Buenos Aires, therefore way higher temperatures</li>
        
          <li>December 2019 I spent in Columbus, Ohio, being the coldest week of the year</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Visual Crossing historic weather data, Swarm
      </span>

      <div class="image-container">
        <img src="graphs/screens/weather-temperature-max.png" class="image-link" alt="Maximal Temperature each Day of the Year" onclick="enlargeImage(this, 'graphs/screens/weather-temperature-max.png', 'Maximal Temperature each Day of the Year')" />
      </div>

      <span class="graph-date">3 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-weather-conditions">
      <h3>Weather Conditions per Year</h3>
      <p class="graph-description">
        Historic weather data based on the location I was at on that day based on my Swarm check-ins.
      </p>

      <ul>
        
          <li>2019 I experienced more cloudy weather than the following years</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Visual Crossing historic weather data, Swarm
      </span>

      <div class="image-container">
        <img src="graphs/screens/weather-conditions.png" class="image-link" alt="Weather Conditions per Year" onclick="enlargeImage(this, 'graphs/screens/weather-conditions.png', 'Weather Conditions per Year')" />
      </div>

      <span class="graph-date">3 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-co2-emission">
      <h3>CO2 Emissions from flying</h3>
      <p class="graph-description">
        My radiation exposure, as well as CO2 emissions from all my flights over the last 7 years
      </p>

      <ul>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Nomadlist
      </span>

      <div class="image-container">
        <img src="graphs/screens/co2-emission.png" class="image-link" alt="CO2 Emissions from flying" onclick="enlargeImage(this, 'graphs/screens/co2-emission.png', 'CO2 Emissions from flying')" />
      </div>

      <span class="graph-date">7 years of data - Last updated on 2022-04-28</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-sleeping-heart-rate">
      <h3>Resting Heart Rate based on location</h3>
      <p class="graph-description">
        I noticed myself feeling more refreshed whenever I spent a night at my childhood bedroom at my parents' on the countryside.
      </p>

      <ul>
        
          <li>On average, my sleeping heart rate when sleeping at my parents' home is less than when sleeping at my own place in Vienna</li>
        
          <li>All nights above 70 bpm are usually associated with either drinking alcohol in the evening, being sick, or sleeping for only a short amount of time (e.g. before a flight)</li>
        
          <li>Even removing all 70+ bpm nights, there is a clear difference between nights spent at my own place in the city, compared to the nights spent on the countryside</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Withings ScanWatch, Manual
      </span>

      <div class="image-container">
        <img src="graphs/screens/sleeping-heart-rate.png" class="image-link" alt="Resting Heart Rate based on location" onclick="enlargeImage(this, 'graphs/screens/sleeping-heart-rate.png', 'Resting Heart Rate based on location')" />
      </div>

      <span class="graph-date">1 years of data - Last updated on 2022-05-30</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-spotify-listen-duration">
      <h3>Spotify Listening Duration over Time</h3>
      <p class="graph-description">
        Spotify tracks every single interaction on their end, including every single song you've ever listened to, and if you finished or skipped each song.
      </p>

      <ul>
        
          <li>Since 2013, I've listened to a total of <span class='highlighted'>480,155 minutes</span> of music, that's 8,002 hours, or 334 days</li>
        
          <li>That's <span class='highlighted'>202,148</span> songs, however I only finished 49% of them</li>
        
          <li>End of 2013 I didn't have an active Spotify subscription, due to issues accessing it from Austria</li>
        
          <li>During some areas in my life, I listened to Cercle/Coccolino Deep sets on YouTube, therefore using Spotify less</li>
        
          <li>I stored a total of 312 Spotify Discover Weekly playlists, every single one since December 2015</li>
        
          <li>As part of this script, I built a tool to create a playlist of the 500 most listened to songs, as well as the first 500 songs listened to</li>
        
          <li>In total, I have paid 1,080 EUR for Spotify Premium</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Spotify
      </span>

      <div class="image-container">
        <img src="graphs/screens/spotify-listen-duration.png" class="image-link" alt="Spotify Listening Duration over Time" onclick="enlargeImage(this, 'graphs/screens/spotify-listen-duration.png', 'Spotify Listening Duration over Time')" />
      </div>

      <span class="graph-date">9 years of data - Last updated on 2022-05-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-instagram-stories">
      <h3>Number of Instagram Stories per Day</h3>
      <p class="graph-description">
        Since I've built <a href='https://instapipe.net'>instapipe.net</a>, I have all my Instagram Stories available in my own database. The blue 'Monthly' area shows the monthly average of IG stories posted, while the green shows the 'Quarterly' average and purple the 'Yearly' average.
      </p>

      <ul>
        
          <li>April 2019 I built <a href='https://instapipe.net'>instapipe</a> while traveling in Hong Kong and South Korea</li>
        
          <li>November 2019 I had a deadline at work, as well as being sick for a few days</li>
        
          <li>March 2020, the start of COVID, shows a clear downwards trend the following months until June when Austria opened up again</li>
        
          <li>September 2021 I was on a roadtrip through Iceland, posting many pictures & videos</li>
        
          <li>Overall you can see a clear downwards trend of the purple 'Year' line over the last 3 years</li>
        
          <li>Since April 2019, I have published a total of 1,906 Instagram stories, averaging 1.6 Stories per day</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        
      </span>

      <div class="image-container">
        <img src="graphs/screens/instagram-stories.png" class="image-link" alt="Number of Instagram Stories per Day" onclick="enlargeImage(this, 'graphs/screens/instagram-stories.png', 'Number of Instagram Stories per Day')" />
      </div>

      <span class="graph-date">3 years of data - Last updated on 2022-06-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-weather-temperature-buckets">
      <h3>How does High Temperature affect my Life?</h3>
      <p class="graph-description">
        On days with a maximum temperature of more than 26 Celsius (78.8 Fahrenheit), the following other factors were affected
      </p>

      <ul>
        
          <li>80% more likely to take a freezing cold shower</li>
        
          <li>37% more likely to hit the gym</li>
        
          <li>26% more daily steps</li>
        
          <li>20% more alcoholic beverages</li>
        
          <li>15% more likely to go out in the evening</li>
        
          <li>13% less likely to take a nap</li>
        
          <li>20% less time in a code editor</li>
        
          <li>21% less likely to be sick</li>
        
          <li>Generally lower stress/anxiety levels</li>
        
          <li>100% less likely to be in a COVID related lockdown</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Visual Crossing historic weather data, Manually
      </span>

      <div class="image-container">
        <img src="graphs/screens/weather-temperature-buckets.png" class="image-link" alt="How does High Temperature affect my Life?" onclick="enlargeImage(this, 'graphs/screens/weather-temperature-buckets.png', 'How does High Temperature affect my Life?')" />
      </div>

      <span class="graph-date">2.5 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-github-contributions">
      <h3>GitHub Open Source Contributions</h3>
      <p class="graph-description">
        GitHub open source contributions visualized using GitHub's own contribution graph. The graphs only count code commits to open source repositories.
      </p>

      <ul>
        
          <li>I started <a href='https://fastlane.tools' target='_blank'>fastlane</a> September 2014, and worked on it until early 2018</li>
        
          <li>Early 2018 I started working on <a href='https://github.com/fastlane/ci' target='_blank'>fastlane.ci</a> which slowly got shut down over time by Google</li>
        
          <li>January 2019 I decided to leave Google, as fastlane.ci wasn't going anywhere</li>
        
          <li>Early 2019 I worked on <a href='https://instapipe.net' target='_blank'>instapipe.net</a> and this very project via <a href='https://github.com/KrauseFx/FxLifeSheet' target='_blank'>FxLifeSheet</a></li>
        
          <li>Second half of 2019, until Summer 2021 I worked at Root on commercial software, which doesn't show up on those graphs</li>
        
          <li>End of 2021 I resumed work on FxLifeSheet to build this very page</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        GitHub
      </span>

      <div class="image-container">
        <img src="graphs/screens/github-contributions.png" class="image-link" alt="GitHub Open Source Contributions" onclick="enlargeImage(this, 'graphs/screens/github-contributions.png', 'GitHub Open Source Contributions')" />
      </div>

      <span class="graph-date">11 years of data - Last updated on 2022-01-01</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-money-distribution">
      <h3>Investment Management & Simulations</h3>
      <p class="graph-description">
        Every second week I track my current investments and cash positions. It shows me how to most efficiently re-balance my investments in case they're off track, while also minimizing the occuring trading fees.
      </p>

      <ul>
        
          <li>I have a burndown chart, which visualizes how my networth would grow/drop over time in various scenarios</li>
        
          <li>I have a graph that ensures I have the right target distribution of high, medium and low risk positions</li>
        
          <li><span class='highlighted'>Whenever I take on a potential new project/role, I simulate the various outcome scenarios</span> that could happen over the next few years</li>
        
          <li>For obvious reasons, I can't include more of the graphs and simulations I have available</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Manually
      </span>

      <div class="image-container">
        <img src="graphs/screens/money-distribution.png" class="image-link" alt="Investment Management & Simulations" onclick="enlargeImage(this, 'graphs/screens/money-distribution.png', 'Investment Management & Simulations')" />
      </div>

      <span class="graph-date">8 years of data - Last updated on 2022-02-09</span>
    </div>
  
    
    
    

    <div class="graphs-entry" id="graphs-entry-money-flow">
      <h3>Annual Money Flow</h3>
      <p class="graph-description">
        Inspired by a <a href='https://old.reddit.com/r/PFPorn' target='_blank'>Reddit finance community</a>, I had been creating annual money flow charts to clearly see how much money I earned, where I spent how much, and how much I saved. The chart below isn't mine (for privacy reasons), but just an example I created.
      </p>

      <ul>
        
          <li><span class='highlighted'>Not owning an apartment, and <a href='https://krausefx.com/blog/one-year-nomad' target='_blank'>living in Airbnbs as a nomad</a> I actually spent less money.</span> This was due to not having to buy any furniture, appliances, etc. but also because the number of items I could buy for myself was limited, as everything was limited to be able to fit into 2 suitcases. Additionally I didn't have to pay any rent whenever I traveled for my job, or during conferences.</li>
        
          <li>Misc. subscriptions/expenses are adding up quickly</li>
        
          <li>Having an End-Of-Year summary from your credit card provider, and always using your card when possible, is a great way to get those numbers</li>
        
          <li>Creating this chart is still a lot of work, as you have to keep in mind that some expenses are not solely yours, but you might have covered for a group (e.g. booked a larger Airbnb), or got reimbursed</li>
        
      </ul>

      <span class="graph-sources">
        <span class="graph-sources-header">
          Sources: 
        </span>
        Chase Credit Card Year Summary, sankeymatic.com, Manually
      </span>

      <div class="image-container">
        <img src="graphs/screens/money-flow.png" class="image-link" alt="Annual Money Flow" onclick="enlargeImage(this, 'graphs/screens/money-flow.png', 'Annual Money Flow')" />
      </div>

      <span class="graph-date">5 years of data - Last updated on 2022-02-09</span>
    </div>
  
</div>

<h3 style="margin-top: 15px;">More Details on how this works</h3>

This project is custom-built for my own personal use, the resulting code is 100% open source on <a href="https://github.com/KrauseFx/FxLifeSheet">KrauseFx/FxLifeSheet</a>.
There are 3 components to this project:

<b>&#8226; Database</b>

A timestamp-based key-value database of all data entries powered by Postgres. This allows me to add and remove questions on-the-fly.

<img src="/graphs/assets/fxlifesheet-database.png" />

Each row has a `timestamp`, `key` and `value`. 

- The `timestamp` is the time for which the data was recorded for. This might differ from `imported_at` which contains the timestamp on when this entry was created. Additionally I have a few extra columns like `yearmonth` (e.g. 202010), which makes it easier and faster for some queries and graphs.
- The `key` describes **what** is being recorded (e.g. `"weight"`, `"locationLat"`, `"mood"`). This can be any string, and I can add and remove keys easily on the fly in the [FxLifeSheet configuration file](https://github.com/KrauseFx/FxLifeSheet/blob/master/lifesheet.json) without having to modify the database.
- The `value` is the actual value being recorded. This can be any number, string, boolean, etc. 

Early on in the project I made the decision not to associate an entry to a specific day due to complexities when traveling and time zones, since the idea was just to detect trends using the collected data. It became clear that detecting trends is only a small part of what can be done with the data, so I [wrote a script](https://github.com/KrauseFx/FxLifeSheet/tree/master/ruby_importers/importers/tag_days) to associate each entry to the correct date.


<b>&#8226; Data Inputs</b>

<img src="/graphs/assets/fxlifesheet-questions.png" id="lifesheet-questions" />

To populate the data, I manually answer questions [multiple times a day](https://github.com/KrauseFx/FxLifeSheet) through a Telegram bot. These questions span from fitness or health (e.g. nutrition, exercise, sleep, etc.) to questions about my life (e.g. how I'm feeling, how much time I spend on social media, etc.).
The extensible use of the [Telegram API](https://core.telegram.org/) made this easy, and even allowed me to customize the keyboard to have predefined replies based on the question asked.

Additionally I can fill-out date ranges with specific values, for example lockdown periods, and bulking/cutting fitness seasons.

<b>&#8226; Data Visualizations</b>

After having tried various tools available to visualize, I ended up writing my own data analysis layer using Ruby, JavaScript together with [Plotly](https://plotly.com/javascript/). You can find the full source code on [KrauseFx/FxLifeSheet - visual_playground](https://github.com/KrauseFx/FxLifeSheet/tree/master/visual_playground).

<hr style="margin-top: 30px; margin-bottom: 20px" />

<h3>But what about privacy?</h3>

I actually published many [privacy projects for iOS](https://krausefx.com/privacy). That little green/orange LED that's rendered on iOS when you record something? Yep, [that was me](https://krausefx.com/blog/ios-privacy-watchuser-access-both-iphone-cameras-any-time-your-app-is-running)! So why this project?

- Much of the data visualized here, is data many larger companies already have about you. Why shouldn't you have it as well?
- The data is stored in my own, private database, not connected to any service
- The data shown above seems very personal but doesn't actually expose any sensitive information. For example, disclosing your current location, your home address, or stores and places you frequently visits is sensitive and potentially dangerous. Even looking at the map of my movement data, you won't get any more information that what most people have in their public Twitter bio.

<h3>Can I use this for my life?</h3>

Yes, you could. However, setting up [FxLifeSheet](https://github.com/KrauseFx/FxLifeSheet) is a bit of a challenge and requires engineering skills and time. Some of the features are specific to my life (like the services I use), and so you'd have to modify parts of the codebase. This project is 100% open source MIT licensed, so you can use, and modify everything to your liking. However, I don't plan on productionizing this project, nor providing any support on GitHub for it. 

Realistically, if you want to get started with QuantifiedSelf, you can use one of the services availalbe. I'd recommend looking for the following:

- Make sure you get full access to the data, as a data export, or a public API
- See how the project maintainers react to feedback. Do they listen to their users, and address issues?
- How much time do you want to invest tracking your data? 
- What's their business model? Are they sustainable and charge money for their service? 
- Don't use anything that's only available as an app: In my experience apps tend to be their own data silo, their feature sets are limited, and analyzing data on the big screen is way more efficient.

<h2>Conclusion</h2>

I've always been fascinated by tracking my own data, and seeing it visualized. While I can't pinpoint the reason why, I remember having this fascination growing up, asking myself questions like `How many steps did I walk in my life?`, and `Did I turn right more often or left?`.

Having read many articles similar to [r/QuantifiedSelf/](https://old.reddit.com/r/QuantifiedSelf/), I really loved the visualizations, but disliked the fact that almost all solutions were data-silos (e.g. standalone iOS apps, Gyroscope, ...) without having full control over the data, nor how it's visualized. Because the data collection spans multiple years, I was apprehensive to rely on any startup or service that runs the risk of being shut down. Additionally, every individual may have differences in how they want to visualize or analyze the data.

Apple was in a great position to improve the current state with Apple Health, but they completely failed with their implementation on both the APIs, as well as the actual Health app.

One aspect I overestimated is the number of days you will track: If you want to look into how many steps you walked in a given city, you'll quickly notice the number of days in each city already being quite low. You'd then also slice the data by season or temperature, since you naturally walk less on very cold days, ending up with only a handful of days outside your main residence.

Overall, having spent a significant amount of time building this project, scaling it up to the size it's at now, as well as analysing the data, <span class="highlighted">the main conclusion is that it is not worth building your own solution, and investing this much time.</span>
When I first started building this project 3 years ago, I expected to learn way more surprising and interesting facts. There were some, and it's super interesting to look through those graphs, however retrospectively, it did not justify the hundreds of hours I invested in this project.

I'll likely continue tracking my mood, as well as a few other key metrics, however will significantly reduce the amount of time I invest in it.

I'm very happy I've built this project in the first place, as it gave me a much better awareness of everything going on in my life. I'm excited to have built this website to wrap-up this project and show-case some of the outcomes to the public.

<hr style="margin-top: 10px; margin-bottom: 10px;" />

<b>2025 Update:</b> I've stopped collecting data and working on this project, but I will keep this page alive.

<hr style="margin-top: 10px;" />

<footer>
  <p>
      Fork this page <a href="https://github.com/krausefx/howisFelix.today" target="_blank">on GitHub</a>
  </p>
  <p>
      Trip data comes from <a href="https://nomadlist.com/@krausefx" target="_blank">nomadlist.com</a>
  </p>
  <p>
      Data comes from <a href="https://github.com/KrauseFx/FxLifeSheet" target="_blank">FxLifeSheet</a>
  </p>
  <p>
      How do I travel? <a href="https://krausefx.com/blog/going-nomad" target="_blank">I live out of a suitcase</a>
  </p>
</footer>

<div class="social-media-list" id="social-media-list-bottom" style="float: left; margin-top: -133px;">
  <p>
    <b>Ideas or suggestions?</b><br />
    <span class="ideas-subtitle">I'd love to hear from you!</span>
  </p>
  <ul>
    <li>{% include icon-twitter.html username="KrauseFx" %}</li>
    <li>{% include icon-instagram.html %}</li>
    <li>{% include icon-url.html %}</li>
  </ul>
</div>
<br /><br />
<hr />

<div id="enlargedImageContainer" style="display: none">
  <div id="enlargedImageContainerBackground"></div>
  <a target="_blank" id="enlargedImageLink">
    <img id="enlargedImage" />
  </a>
  <p id="enlargedImageTitle" onclick="dismissImage()"></p>
</div>

<div id="arrow-left-button" class="arrow-button"></div>
<div id="arrow-right-button" class="arrow-button"></div>

<script type="text/javascript">
  // Subscribe to mouse wheel
  const imageLinks = document.getElementsByClassName("image-link")
  for (let i = 0; i < imageLinks.length; i++) {
    imageLinks[i].addEventListener("mousedown", function(event) {
      if (event.button === 1) { // 1 = center mouse button
        event.preventDefault()
        window.open(this.getAttribute("src"), '_blank');
      }
    })
  }

  let lastNode = null;

  function enlargeImage(node, img, title) {
    // Check if this is a center mouse click (Control or Meta Key)
    if (event.ctrlKey || event.metaKey) {
      window.open(img, '_blank');
      return;
    }

    // Check if it's already been the selected one
    if (lastNode == node.parentElement.parentElement) {
      showFullScreen(img, title);
      return;
    }

    if (lastNode) {
      lastNode.classList.remove("graph-entry-selected");
    }
    node.parentElement.parentElement.classList.add("graph-entry-selected");

    alignArrowKeys(node.parentElement.parentElement);

    // If we have a small screen, we also want to immediately use full-screen mode
    if (useMobileUI()) { 
      showFullScreen(img, title); 
    } else {
      node.parentElement.parentElement.scrollIntoView();
      window.scrollBy({ top: -15 });
    }

    lastNode = node.parentElement.parentElement;
  }

  function useMobileUI() {
    var width = window.innerWidth || document.documentElement.clientWidth || document.body.clientWidth;
    return width < 800;
  }

  function alignArrowKeys(node) {
    const arrowLeft = document.getElementById("arrow-left-button");
    const arrowRight = document.getElementById("arrow-right-button");
    if (node) {
      // Attach the arrows to be next to the `graph-entry-selected`
      const topPx = (node.offsetTop + 250 - arrowRight.offsetHeight / 2) + "px";

      arrowLeft.style.left = (node.offsetLeft - arrowLeft.offsetWidth - 10) + "px";
      arrowLeft.style.top = topPx;

      arrowRight.style.left = (node.offsetLeft + node.offsetWidth - arrowRight.offsetWidth + 50) + "px";
      arrowRight.style.top = topPx;
    } else {
      // This was the first or last graph, hide the arrow keys
      arrowLeft.style.left = "-100px";
      arrowRight.style.left = "-100px";
    }
  }

  function showFullScreen(img, title) {
    document.getElementById("enlargedImageContainer").style.display = "block";
    document.getElementById("enlargedImage").src = img;
    document.getElementById("enlargedImageLink").href = img;
    document.getElementById("enlargedImageTitle").innerHTML = title;
  }

  function dismissImage() {
    document.getElementById("enlargedImageContainer").style.display = "none";
  }

  function nextGraph() {
    if (lastNode) {
      lastNode.classList.remove("graph-entry-selected");
      lastNode = lastNode.nextElementSibling;

      if (lastNode) {
        lastNode.classList.add("graph-entry-selected");

        alignArrowKeys(lastNode)
        lastNode.scrollIntoView();        
        window.scrollBy({ top: -15 });
        clearSelection();
      } else { alignArrowKeys(lastNode); }
      return false;
    }
    return true;
  }
  function previousGraph() {
    if (lastNode) {
      lastNode.classList.remove("graph-entry-selected");
      lastNode = lastNode.previousElementSibling;
      if (lastNode) {
        lastNode.classList.add("graph-entry-selected");

        alignArrowKeys(lastNode)
        lastNode.scrollIntoView();        
        window.scrollBy({ top: -15 });
        clearSelection();
      } else { alignArrowKeys(lastNode); }
      return false;
    }
    return true;
  }

  function clearSelection() {
    // As sometimes the arrows get selected
    if (window.getSelection) {window.getSelection().removeAllRanges();}
    else if (document.selection) {document.selection.empty();}
  }

  document.getElementById("arrow-left-button").addEventListener("click", previousGraph);
  document.getElementById("arrow-right-button").addEventListener("click", nextGraph);
  document.getElementById("enlargedImageContainerBackground").addEventListener("click", dismissImage); // desktop browsers

  // so that mobile devices don't scroll 
  // Important to use `touchmove``, and not `touchstart`, as `touchstart` would be triggered for a regular click also
  // causing the user to click the element behind when trying to dismiss the image
  document.getElementById("enlargedImageContainerBackground").addEventListener("touchmove", dismissImage); 

  window.addEventListener('resize', function(event) {
    // Reposition the arrows
    alignArrowKeys(lastNode);
  }, true);

  window.addEventListener("keyup", function(e) {
    if (e.keyCode == 27) { // ESC
      if (document.getElementsByClassName("graph-entry-selected").length > 0 && document.getElementById("enlargedImageContainer").style.display == "none") {
        clearSelection();
        lastNode.classList.remove("graph-entry-selected");
        lastNode = null;
      }
      dismissImage();
      alignArrowKeys();
      return true;
    }

    // Use arrow keys
    if (e.keyCode == 37) { // Left key
      return previousGraph();
    }
    if (e.keyCode == 39) { // Right key
      return nextGraph();
    }
  }, false);
</script>

<style type="text/css">
  .hidden-due-to-project-paused {
    display: none !important;
  }
  #graphs-container {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-around;
    margin-top: 10px;
  }
  #enlargedImageContainerBackground {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.8);
    z-index: -100;
  }
  #enlargedImageContainer {
    position: fixed;
    z-index: 100;
    top: 0;
    cursor: pointer;
    left: 0;
    right: 0;
    bottom: 0;
    height: 100%%;
    width: 100%%;
    text-align: center;
    padding-top: 40px;
  }
  #enlargedImage {
    background-color: green;
    object-fit: contain;
    max-height: calc(100% - 80px);
    max-width: 1200px;
  }
  @media (max-width: 1200px) {
    #enlargedImage {
      max-width: 100%;
      max-height: 100%;
    }
  }
  #enlargedImageTitle {
    color: white;
    font-size: 30px;
    font-weight: bold;
    margin-top: 20px;
    line-height: 1.2;
  }
  .graphs-entry {
    max-width: 470px;
    margin: 10px;
    border: 2px solid #e4e7ef;
    border-radius: 9px;
    padding: 20px;
  }
  .graphs-entry > ul > li {
    color: #555;
  }
  .graphs-entry > h3 {
    font-size: 140%;
  }
  .graph-description {
    color: #555;
  }
  .graph-sources-header {
    color: #555;
    font-weight: bold;
  }
  .graph-sources {
    color: #777;
  }
  .graph-date {
    float: right;
    color: #777;
    margin-top: 10px;
    font-size: 75%;
    margin-bottom: -10px;
  }
  .image-container {
    width: 100%;
    text-align: center;
  }
  .image-link {
    padding-top: 10px;
    cursor: pointer;
    max-height: 900px;
  }

  /* Real-Time Dashboard UI */
  #mapsContainer {
    width: 100%;
    height: 'auto';
    position: absolute;
    top: 0px;
    left: 0px;
    z-index: -1;
  }
  #currentLocationMap {
    width: 100%;
    background-position: center !important;
    background-size: cover !important;
    height: 230px;

    /* Show loading map by default while loading */
    background: url("/graphs/assets/loading-map.jpg") no-repeat;
  }
  #storyProfilePicture {
    width: 128px;
    height: 128px;
    border-radius: 70px;
    cursor: pointer;
    border: 4px solid #fff;
  }
  #story-available {
    margin-left: auto;
    margin-right: auto;
    left: 0;
    right: 0;
    position: absolute;
    top: 158px;
    z-index: 10 !important;
  }
  #realTimeDataDiv {
    width: 100%;
    text-align: center;
    margin-bottom: -20px;
    padding-bottom: 10px;
    margin-top: 300px;
  }
  .blurred {
    filter: blur(5px);
  }
  .highlighted {
    font-weight: normal;
    background-color: rgba(255, 255, 0, 0.35);
    padding: 5px;
    margin-left: -5px;
  }
  #food-container {
    margin-bottom: 40px;
    margin-top: -20px;
  }
  #feels-h {
    margin-top: -15px;
    margin-bottom: 20px;
  }
  hr {
    border: 0;
    height: 1px;
    background-image: linear-gradient(to right, rgba(0, 0, 0, 0), rgba(40, 40, 40, 0.3), rgba(0, 0, 0, 0));
    margin-bottom: 35px;
    background-color: transparent !important;
  }
  .ago-subtle {
    color: #666;
    font-size: 14px;
  }
  #real-time-table {
    border: none;
  }
  #real-time-table > * > tr > td:first-child {
    text-align: right;
    white-space: nowrap;
  }
  #real-time-table tr {
    line-height: 16px;
    font-size: 88%;
  }
  #real-time-table tr .ago-subtle {
    font-size: 85%;
  }
  #real-time-table tr .highlighted {
    padding: 4px;
  }
  @media screen and (max-width: 800px) {
    #real-time-table tr {
      line-height: 12px;
    }
  }
  #table-container { 
    margin-top: 20px; 
    margin-left: auto; 
    margin-right: auto; 
    text-align: left; 
    width: 450px;
  }
  .git-footnote {
    text-align: center;
    color: #666;
    font-size: 14px;
  }
  #git-repo-link {
    text-decoration: none !important;
  }
  #git-header {
    margin-top: 10px;
    font-size: 25px;
    margin-bottom: 20px;
  }
  #git-header > a {
    color: #333 !important;
    font-weight: 400 !important;
    font-size: 90%;
  }
  @media screen and (max-width: 800px) {
    #current-weight-time {
      display: none;
    }
    #real-time-table > * > tr > td:first-child {
      width: 115px;
    }
    #since2013 {
      display: none;
    }
  }
  @media screen and (max-width: 450px) {
    #real-time-table {
      max-width: 80%;
    }
  }
  .food-overview {
    line-height: 60px;
  }
  .food-overview > div {
    margin: 0 10px;
    display: inline-block;
    position: relative;
    text-align: center;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    width: 150px;
    font-size: 90%;
    height: 70px;
  }
  .macro-bar-outer {
    width: 110px;
    background-color: rgba(205, 205, 205, 0.5);
    height: 8px;
    border-radius: 5px;
    position: absolute;
    margin-top: 55px;
    margin-left: auto;
    margin-right: auto;
    left: 0;
    right: 0;
    text-align: center;
  }
  .macro-bar-inner {
    width: 0px;
    background-color: rgba(0, 220, 0, 1);
    height: 8px;
    border-radius: 5px;
  }
  #foodEntriesTable {
    border: none;
    max-width: 430px;
    margin-left: auto;
    margin-right: auto;
    display: inline;
    white-space: nowrap;
    border-spacing: 4px;
    border-collapse: separate;
  }
  #foodEntries {
    margin-top: -10px;
    width: 100%;
    text-align: center;
  }
  #foodEntriesTable tr {
    background-color: rgba(255, 255, 255, 0.9) !important;
    font-size: 90%;
    line-height: 1.35;
  }
  #foodEntriesTable tr > td {
    color: #669;
    padding: 3px 10px 0;
    border: none;
    text-overflow: ellipsis;
    overflow: hidden;
  }
  #foodEntriesTable th {
    color: #039;
    border-bottom: 1px solid #6678b1;
    padding: 5px 5px;
    border-top: 0;
    border-right: 0;
    text-align: center;
    border-left: 0;
    font-size: 14px;
    background-color: transparent;
    white-space: nowrap;
  }
  #foodEntriesTable tr > td:first-child {
    max-width: 270px;
  }
  #foodEntriesTable tr > td:last-child {
    max-width: 80px;
    width: 80px;
  }
  .hidden-food {
    display: none;
  }
  #show-all-food-a {
    font-weight: bold;
    text-decoration: none;
    cursor: pointer !important;
  }
  .graph-entry-selected {
    max-width: 100%;
    border: 3px solid #00BFFF;
  }
  .arrow-button {
    position: absolute;
    height: 40px;
    top: calc(50% - 10px);
    cursor: pointer;
    top: -100px;
  }
  #arrow-right-button {
    content: url('/graphs/assets/source_icons_arrow-right-circled.svg');
  }
  #arrow-left-button {
    content: url('/graphs/assets/source_icons_arrow-left-circled.svg');
  }
  @media screen and (max-width: 800px) {
    #arrow-right-button {
      display: none;
    }
    #arrow-left-button {
      display: none;
    }
  }
  #nextCityContainer {
    margin-top: -10px;
  }
  #next-cities-table {
    display: none;
    margin-left: auto;
    max-width: 500px;
    margin-right: auto;
    border: none;
    margin-bottom: 0px;
    border-spacing: 4px;
    border-collapse: separate;
  }
  #next-cities-table > * > tr > th {
    color: #039;
    border-bottom: 1px solid #6678b1;
    padding: 5px 5px;
    border-top: 0;
    border-right: 0;
    text-align: center;
    border-left: 0;
    font-size: 14px;
    background-color: transparent;
    white-space: nowrap;
  }
  #next-cities-table > tr {
    background-color: transparent !important;
  }
  #next-cities-table > tr > td {
    color: #669;
    border: none;
    padding: 5px 20px 0px;
    background-color: transparent !important;
    white-space: nowrap;
  }
  @media screen and (max-width: 800px) {
    #next-cities-table > tr > td {
      padding: 0px 10px 0px;
    }
  }
  footer {
    margin-top: 30px;
  }
  footer > p {
    text-align: right;
    color: #777;
    margin-top: 0px;
    margin-bottom: 5px;
    font-size: 14px;
  }
  .imageCarousel {
    margin-top: 10px;
    height: 127px; /* this is important to prevent weird scrolling on iOS */
    width: 100%;
    overflow-y: none;
    overflow-x: scroll;
    white-space: nowrap;
  }
  .imageCarousel > a > img {
    height: 120px;
    width: auto;
    max-width: none; /* to override page wide attribute */
    display: inline-block;
  }
  #personalCarousel > a > span {
    /* I didn't spend the time investigating why this is necessary */
    margin-right: 5px;
    height: 120px;
    display: inline-block;
    width: 120px; /* IG pictures should always be square */
    background-size: cover;
    background-repeat: no-repeat;
    background-position: 50% 50%;
  }
  @media screen and (max-width: 1100px) {
    #data-sources-overview td {
      padding: 5px 10px;
    }
    #data-sources-overview th {
      padding: 5px 10px;
    }
  }
  @media screen and (max-width: 800px) {
    #data-sources-overview td {
      font-size: 90%;
    }
    #data-sources-overview th {
      font-size: 90%;
    }
  }
  #lifesheet-questions {
    height: 510px;
    float: right;
    margin-left: 20px;
    margin-bottom: 15px;
    margin-top: -70px;
  }
  @media screen and (max-width: 800px) {
    #lifesheet-questions {
      float: none;
      margin-top: 10px;
    }
  }
  #fx-user-link {
    color: #111 !important;
    text-decoration: none;
  }
  .social-media-list {
    border: 1px solid #ccc;
    border-radius: 5px;
    padding: 15px;
    padding-bottom: 0px;
    width: auto;
    height: auto;
  }
  .social-media-list p {
    margin-bottom: 10px;
    line-height: 1.5;
  }
  .social-media-list ul {
    list-style: none;
    margin-left: 0px;
  }
  .social-media-list li {
    color: #353535;
    font-size: 16px;
    font-weight: 400;
    line-height: 1.3;
  }
  .social-media-list a {
    text-decoration: none;
  }
  .social-media-list span {
    vertical-align: middle;
  }
  .social-media-list svg {
    color: #353535 !important;
  }
  .icon > svg {
    display: inline-block;
    vertical-align: middle;
  }
  .social-media-list .url-details {
    margin-left: 4px; /* no idea why this is needed */
  }
  .icon > svg path {
    fill: #828282;
  }
  .social-media-list .icon {
    padding-right: 5px;
  }
  .ideas-subtitle {
    color: #666;
  }
  @media screen and (max-width: 800px) {
    #social-media-list-bottom {
      float: right !important;
      margin-top: 10px !important;
      margin-bottom: 20px;
      margin-left: 15px;
    }
  }
  .graph-overview-footer {
    color: #777;
    text-align: left;
    font-size: 14px;
    margin-bottom: 0;
  }
</style>

<div class="footer-col footer-col-3">
  {% include newsletter.html %}
</div>

<script type="text/javascript">
  var links = document.links;
  for (var i = 0, linksLength = links.length; i < linksLength; i++) {
    if (links[i].hostname != window.location.hostname) {
      links[i].target = '_blank';
    } 
  }
</script>
