---
layout: post
title:  "Creating resizable Flexbox four-pane layout in React"
date:   2018-05-02 16:48:52 +0200
published: true
category: frontend
tags: frontend react flexbox
---

As most full stack developers, I wasn’t „born” as one - almost all of us came from differents background, so we have a unique perspective and problems on our ways. I originated from backend and for kinda a long time I had no idea about frontend - CSS was absolutely black magic fuckery to me. And yet here I am, trying my best at doing something that not only (maybe just barely, but still) works,  but also something that won’t make users’ eyes bleed.

I am in no way an expert at designing pretty stuff nor at cascade stylesheets, but lately I’ve been doodling a project from scratch in React and found out that there was very little written on this „resizable layout” subject, so I decided to describe my way of doing this thing.


## An idea
 I had a general vision of what I wanted to achieve when it come to the basic layout of the application, and it looked something like this: 

![Layout draft]({{"images/four-pane-resizable/draft.png" | absolute_url}})


As you see, it’s nothing fancy. Four panes: a static header with fixed size, a sidebar resizable horizontally to some extend and two main panes where each one can be shrinked in favor of the other.


## The tools
It is supposed to be an React application. For resizability I’m going to use [interact.js](http://interactjs.io), as it’s easy to use and does exactly what we want. For debug purposes I’ll also use [react-measure](https://github.com/souporserious/react-measure) - an easy to use React component which allows us to measure the size of our panes. All the layout stuff is going to be done with vanilla CSS3 using Flexbox.


## Starting from scratch - four-pane HTML structure
First we need to identify what we have to work with. There are 4 panes, so we need 4 divs. However, if you have ever touched any layouting with CSS, you know this is not enough - we need to pack it up in some wrappers in order to make it work. Flexbox can group your items in two ways - or _directions_, if you insist - horizontally (rows) or vertically (columns), so let’s identify our rows and cols.

![Grouped layout draft]({{"images/four-pane-resizable/draft2.png" | absolute_url}})


{% highlight html %}
<div class="wrapper">
    <div class="header"></div>
    <div class="content">
        <div class="sidebar"></div>
        <div class="main-part">
            <div class="upper-pane"></div>
            <div class="lower-pane"></div>
        </div>
    </div>
</div>
{% endhighlight %}

Notice, how all the divs created on the same indentation level forms either a column or a row - neat!


## Basic CSS and Flexbox
So now we have our structure, but nothing interesting shows up in the browser just yet. Let’s define some basic styles for our divs, shall we?

```css
html, body {
    margin: 0;
    height: 100%;
}

/* Wrappers */
.wrapper {
    width: 100%;
    height: 100%;
}
.content {}
.main-part {}

/* Panes */
.header {
    width: 100%;
    height: 150px;
    background-color: aqua; /* temporary color */
}
.sidebar {
    width: 30%;
    height: 100%;
    background-color: red; /* temporary color */
}
.upper-pane {
    width: 70%;
    height: 50%;
    background-color: green; /* temporary color */
}
.lower-pane {
    width: 70%;
    height: 50%;
    background-color: yellow; /* temporary color */
}
```

So, what do we have here? Not much, actually - I just made our divs the size we want them to be by default and gave them some color in order to actually see what I’m doing. The coloring thing is just for debug purposes - we’ll get rid of it later, when everything is actually placed where it belongs.

All we have to do now is just to put this things together and we can move on to more interesting, interactive stuff. But first, let’s form a love-hate relationship with Flexbox. Flexbox is basically this cool feature that comes with CSS3 that lets you arrange things the way you want to and define the way items manage the space you give them. I realize that explaining what Flexbox is to people in 2018 might seem silly, but bear with me.

First things first - even though our header is exactly where we want it to be, let’s tell our webpage that it’s not an accident:

```css
.wrapper {
    display: flex;
    flex-direction: column;
    width: 100%;
    height: 100%;
}
```

We told our wrapper that we want it to use Flexbox (with `display: flex`) and then made it position it’s children vertically.  Easy! What’s up next?

```css
.content {
    display: flex;
    flex-direction: row;
    width: 100%;
    height: 100%;
}
.main-part {
    display: flex;
    flex-direction: column;
    width: 100%;
    height: 100%;
}
```

Basically, same thing. Our content forms a row, our main part another column. OK, so how does our webpage look now?

![Bad layout]({{"images/four-pane-resizable/bad-layout.png" | absolute_url}})


Not bad, all of our divs are in place, but our content is a bit short, as it doesn’t fill the whole width. This is because we need to look at our divs differently now when we use Flexbox. 70% of the width we set in the upper and lower pane no longer means 70% of the screen size, but rather 70% of the Flexbox container it’s in. So theoretically all we have to do to fix it is change the width to 100%. However, Flexbox will always try to use all the place the div has in it’s container, so actually, we can just skip all the width settings for this divs and be all set. Other things that are redundant now are all the settings that are non-restrictive, like header’s width and sidebar’s height.

That was simple, wasn’t it? This is the „love” part of my relationship with Flexbox - putting elements in places is dead simple. But no worries, the „hate” part will come up too, eventually.

OK, so now we have it! Let’s get rid of those ugly colors, show some borders and move on to JS stuff!

```css
html,
body {
    margin: 0;
    height: 100%;
    text-align: center;
    font-family: sans-serif;
}

/* Wrappers */

.wrapper {
    display: flex;
    flex-direction: column;
    width: 100%;
    height: 100%;
    
}
.content {
    display: flex;
    flex-direction: row;
    width: 100%;
    height: 100%;
}

.main-part {
    display: flex;
    flex-direction: column;
    width: 100%;
    height: 100%;
}

/* Panes */

.header {
    height: 150px;
    background-color: #222;
    color: white;
}

.sidebar {
    width: 30%;
    border-right: solid 1px darkgray;
}

.upper-pane {
    height: 50%;
    border-bottom: solid 1px darkgray;
}

.lower-pane {
    height: 50%;
}
```


![Good layout]({{ "images/four-pane-resizable/good-layout.png" | absolute_url }})


## Let’s React!
I’ll assume that you know how to create an application in React and know a bit on how it works. So after creating a new project just throw our HTML structure to the render method and include a CSS file.

First thing you should notice is that after you copy-paste our code is that none of our divs appears. After inspecting elements in your browser you may find that all of them have the height of 0 (or 1 if they have border). The fix for this issue is actually very simple - in our CSS file we just need to set the height of our root div, in which React keeps everything. Now we’re all set!

```css
#root {
  height: 100%;
}
```


## Measuring our divs
Now, let’s measure our divs. It will be useful for later to find out if everything works just as we planned. To install react-measure with npm run:
`npm install -s react-measure`. For instructions on how to use it take a look at [examples on GitHub](https://github.com/souporserious/react-measure#example). We could of course achieve the same by using React’s `ref`  ([Refs and the DOM - React](https://reactjs.org/docs/refs-and-the-dom.html)), but I don’t think there’s any need for reinventing the wheel, especially if we use it just for debug purposes.

First thing we want to do is to hold our divs’ dimensions somewhere in application’s state in order to update it every time our pane resizes and to be able to show it inside of our containers. In real application you probably would prefer to create a component for every pane, but for our purposes I think there’s no such need. So let’s set our app’s initial state:

```js
 constructor(props) {
    super(props);

    this.state = {
        sidebar: {
          width: 0,
          height: 0
        },

        upperPane: {
          width: 0,
          height: 0
        },

        lowerPane: {
          width: 0,
          height: 0
        },
    }
  }
```

Great! Now it’s time to set our Measure component:

```jsx
import Measure from 'react-measure';
// ...
render() {
    const {sidebar, upperPane, lowerPane} = this.state;
    return (
		// ...
      <div class="content">
        <Measure bounds onResize={(rect) => { this.setState({ sidebar: rect.bounds }) }}>
          {
            ({ measureRef }) =>
                <div ref={measureRef} class="sidebar"> 
                    {Math.round(sidebar.width)}x{Math.round(sidebar.height)}
                </div>
          }
        </Measure>
	// ...
	);
}
```

OK, so a lot of things is happening here.  For clarity I skipped all non-crucial parts of our `render` method, as well as panes other than sidebar, because all the others looks exactly the same. 

But back to the point - Measure component needs to wrap any component  that dimensions we wanna get, as it provides us a ref object, allowing it to access element’s DOM. In it’s props you can see a onResize callback. This should be used to do anything you want to do with components rect, as you won’t be able to access it elsewhere. Next thing to notice - Measure *children* (the part between opening and closing tags) have to be a function taking a single argument. This argument then needs to be later passed as ref prop to the element we’re measuring. And that’s it! You can now access these values right from your app’s state!


## Let’s make it move, finally!
In order to be able to resize our divs we need interact.js I mentioned at the beginning. To get it, just type: `npm install -s interactjs` and you’re ready to go!

Following [the docs](interactjs.io), let’s make our divs resizable right after React set ups all the DOM elements:

```js
  componentDidMount() {
    interact('.sidebar').resizable({
      edges: { left: false, right: true, bottom: false, top: false},

      restrictEdges: {
        outer: 'parent',
        endOnly: true,
      },

      // minimum size
      restrictSize: {
        min: { width: 150 },
        max: { width: 500 },
      },

      inertia: false,

    }).on('resizemove', event => {
      event.target.style.width = (Math.round(event.rect.width)) + 'px';
    });

    interact('.upper-pane').resizable({
      edges: { left: false, right: false, bottom: true, top: false},

      restrictEdges: {
        outer: 'parent',
        endOnly: true,
      },

      // minimum size
      restrictSize: {
        min: { height: 150 },
        max: { height: 700 },
      },

      inertia: false,

    }).on('resizemove', event => {
      event.target.style.height = (Math.round(event.rect.height)) + 'px';
    });
  }
```

Few things to notice here before we move on. We only make 2 out of 4 our divs resizable, as header has fixed size and lower-pane height is dependent on upper-pane’s height, so there is no need to make both resizable. First, we specify which edges should be used for us to change sizes. Next we make restrictions - we want our divs to be contained inside parent element at all times and we don’t want to make them too big or too little. We don’t need inertia here, as we want divs to be exactly as big as we specify, no matter how fast we move our mouse. Then, finally, we have to specify an action for „resizemove” event - here’s where we actually change our divs’ sizes.  I believe the code is self-explanatory enough so we don’t have to talk about every line.

Now if we run our code we’ll see, that everything works! …Well, kind of, at least. We indeed can resize our panes, but there’s a tiny problem. When we look at our measures we’ll see that our restrictions aren’t really working - we we can make a sidebar to be way thinner than 150px and reaching the upper limit of 500px is virtually impossible. The same goes to our upper pane. What’s more, when we resize the divs it… well, it just doesn’t feel right. 

Fixing this problem took me some time. I thought that I screwed up something configuring interact.js, or maybe that it simply doesn’t work well with React. After few tries I finally found out that the fix was very simple, and the problem was caused by…


## Flexbox - we meet again!
Yup, our CSS is messed up. Remember when I said that Flexbox manages both positions and how our elements uses the space we give them? We set up our positions and it’s fine, however we did nothing about this space management thing. And here comes the „hate” part of my relationship with Flexbox. By default, Flex wants to use only as much space as we specify in our CSS. If the size changes it has no problem with shrinking, however doesn’t really want to grow. This is why our resizing works the way it does - shrinking doesn’t cause any problems, but growing them makes our divs freak out. In order to fix it, we need to mess with [ `flex` property](https://developer.mozilla.org/en-US/docs/Web/CSS/flex) a bit. 

OK, so first we need to think what do we want from Flexbox? `flex` property is a shorthand of `flex-grow` ,  `flex-shrink`  and `flex-basis`, so it’s not hard to notice that it has something to do with elements sizing. So, what does `flex-grow` do? It sets how eager our element is to grow. If it is more important then the others, then the higher value it have, the more space (proportionally) it will take over other elements in flex container. So if we have 3 elements, with first and third having `flex-grow: 1` and the middle one with `flex-grow: 3;`, the middle one will take roughly 3/5 of containers’ size. Next we have `flex-shrink`. By analogy, this is how eager the container is to shrink, so the higher the value, the smaller will the element be when something else wants to take it’s space. And last but not least there is `flex-basis` . This one tells us what is the basic size of our container. Generally it’s the restriction, that specifies what is the minimal size that element should have if surrounded by elements with same `flex-grow` and `flex-shrink` values.

So the big question is - do we need any of this? Normally - definitely yes. It’s a great thing for responsive web design, as you can use it to specify how element should behavior on different browser sizes or if you don’t really care about „pixel-perfectness” of your container sizes. However in our case, when we want to specify said sizes on our own, it causes more damage than good, so I think we can simply switch it of by setting `flex: none`  (which translates to `flex-grow: 0`, `flex-shrink: 0`, `flex-basis: auto` ) in our sidebar,  as well as in upper and lower panes.


## And there you have it!

<div style="width:100%;height:0;padding-bottom:57%;position:relative;"><iframe src="https://giphy.com/embed/9ViAYQv7hRIoGAtHHL" width="100%" height="100%" style="position:absolute" frameBorder="0" class="giphy-embed" allowFullScreen></iframe></div>
<br/>

And this is how I managed to solve my problems trying to work with React, interact.js and Flexbox in order to create resizable four-pane layout. You can find all the source code on [my GitHub](https://github.com/sc0/resizable-four-pane-layout).

 As I mentioned at the beginning, I am I no way an expert in frontend nor any of the technologies used and am still learning them, so if you find my solution not as good as it could be (or straight up shit), let me know!