---
layout: article
title: Navigating the Front End Wilderness Part 2
modified:
categories: Development
excerpt: 
tags: [Javascript, CORS, React, JQuery, Firefox]
image:
  feature: 
  teaser: "bar.jpg"
  thumb:
post-author: James
---

_Its fun being out of your depth so long as you are too stubborn to stop swimming!_

Today I ran into major problems trying to setup a React script to query JSON from a site and parse it into something useable.  I'm new to this stuff and used an existing <a href="https://facebook.github.io/react/tips/initial-ajax.html">tutorial</a> to mangle according to my needs.

I ended up with the following code:

~~~ javascript
var PlanetList = React.createClass({
    getInitialState: function() {
        return {Planets: []};
    },

    componentDidMount: function() {
        this.serverRequest = $.getJSON(this.props.source, function (result) {
            var parray = result.map(function(p) {
                return {
                    PlayerName: p.PlayerName,
                    PlanetName: p.PlanetName
                };
            });
            this.setState({Planets: parray});
        }.bind(this));
    },

    renderPlanets: function(){
        return this.state.Planets.map(function(planet){
            return(
                <li>
                    Player {planet.PlayerName} has planet {planet.PlanetName}.
                </li>)
        });
    },

    componentWillUnmount: function() {
        this.serverRequest.abort();
    },

    render: function() {
        return(
            <ul>
                {this.renderPlanets()}
            </ul>)
    }
});

ReactDOM.render(
    <PlanetList source="http://localhost:8000" />,
    document.getElementById('container')
);
~~~

As you can see I am reading in a source from a local webserver which contains JSON for an array of objects, example:

~~~ javascript
[
  {
    "ContainsPlanet": true,
    "PlayerName": "Computer",
    "PlayerTokens": 2,
    "PlanetName": "g"
  },
  {
    "ContainsPlanet": false,
    "PlayerName": "",
    "PlayerTokens": 0,
    "PlanetName": ""
  },
  {
    "ContainsPlanet": false,
    "PlayerName": "",
    "PlayerTokens": 0,
    "PlanetName": ""
  }
]
~~~

The problem I experienced is that my array was not being processed and any code I placed inside the JQuery code block was not run.  I tried the following:

* Checked webserver output to verify JSON response is being issued correctly
* Checked webserver logs to verify its receiving requests from the React application
* Googled my &^%#%@ ass off trying all manner of JQuery/React combinations to try to get the JSON data and parse it

Nothing worked.  Finally in my browser when loading the React page I decided to enable Firefox's Security logging.  I then received the following message:

`Cross-Origin Request Blocked: The Same Origin Policy disallows reading the remote resource at http://localhost:8000/. (Reason: CORS header 'Access-Control-Allow-Origin' missing).`

**Bingo!**

Since I'm currently working in a local testing phase I simply added the following to my Go function for serving the JSON data and like magic my React application sprang to life!
 
~~~ golang
w.Header().Set("Access-Control-Allow-Origin", "*")
~~~

 A breath of fresh air!  Now time to move to a deeper depth :)


