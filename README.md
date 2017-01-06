# Running the Site

The site can be run either locally by opening `index.html` after cloning this repository, or you can check it out live at `http://www.ashduckett.com`. This will enable you to run PageSpeed Insights on it.

# Optimisations Made
## index.html
One of the goals of this project was to get [PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/) to give a score of 90 or higher for this specific page.

It's worth pointing out that the initial scores for mobile and desktop were 27 and 29 respectively. These scores are out of 100. This was when hosted on my personal website.

### Optimisation: Scaling down of pizzeria.jpg
I downloaded [GIMP](https://www.gimp.org/) to do this. At this stage I made the image the size the largest it needed to be for the larger version shown on the site. This gave a new set of scores: mobile 63 and desktop 73. What a difference!

### Optimisation: Remove render blocking on print.css

I noticed that there was one there for printing specifically. One of the things I've learned whilst doing this module is that regardless of whether or not your CSS file is required for your initial page load, maybe it's specifically for printing, for example, it will still be render blocking. You can stop this from being render blocking by removing it from the critical rendering path by telling the browser that it's specifically for printing in this case. Other examples might include devices at specific orientations, etc.  Anyway, the line that does this in `index.html` is `    <link href="css/print.css" rel="stylesheet" media="print">` which you should be able to see around line 10.

### Optimisation: Making analytics.js Asyncronous
During this module I learned that JavaScript is parser blocking. I believe this to mean that, by default, the browser cannot parse the text used to build the DOM until JavaScript has executed. So it'll read some of the text, hit the JavaScript, execute that, and then move on to the rest of the DOM. This is inefficient, particularly if you're loading JavaScript from a file at this point. There are a couple of tricks you can use to prevent the browser from halting DOM construction on hitting a `<script>` tag and the easiest is to use the `async` keyword. This means the DOM is able to continue being built since the JavaScript in this case doesn't rely on it being there before executing. This can be found in the line:
```<script async src="http://www.google-analytics.com/analytics.js"></script>``` which should be around line 149.

Unfortunately at this point there were no increases to my score on PageSpeed Insights! It did, however, suggest that I have two render blocking CSS files; the Google font, and `style.css`.

### Optimisation: Separating Out Mobile Specific CSS
I found a small piece of CSS that was intended only for use in mobile phones in the portrait orientation. This meant that I could place this into its own file and then prevent it being render blocking on what should be line 9 of index.html:
```    <link href="css/style-mobile.css" rel="stylesheet" media="orientation:portrait">```

Again, this made no different to my score, but I know it's good practice.

### Optimisation: Inlining of CSS
The CSS present in style.css is quite a small chunk. I wouldn't want to do this normally because I like to have CSS in separate files, particularly as this one is used in multiple places, but I have inlined this CSS in an effort to make the page load faster, and remove the blocking warning given by PageSpeed Insights. This runs from lines 12 to 136 of index.html.

Again, this made no difference to my score! I'll leave it as it is, however, to prevent the warning from PageSpeed Insights. This makes me question PageSpeed Insights. You can't change the fact that CSS is render blocking, but you need it for above the fold important CSS. It can't always know what's above the fold because it won't know what device you want specific analysis done on, so maybe some warnings are inevitable.

### Optimisation: Google Font
Looking at the Google Font, it seems that there are solutions out there to speed up the loading of this. One of them is to inline the CSS to load the font, although this can result in a Flash of Unstyled Text (FOUT). This means my options were to either remove the font altogether or to have that flash. Since the goal here is optimisation, maybe it's not up to me to worry about style, so I removed the font altogether.

This has given me new scores of 77 on mobile and 78 on desktop.

### Optimisation: More Image Optimisation
I next looked at the images again. I added versions of each thumbnail image on index.html which were thumbnail size. These can be found in `frontend-nanodegree-mobile-portfolio\views\images\thumbnails`. In doing this, I also removed the external resources used to get the other thumbnail images and placed them directly on the server. Whilst doing this, This has given me new scores of 89 on mobile and 91 on desktop. I then reduced the quality of the images using GIMP which gave even better scores of 90 and 92! I got there!

## Moving Pizzas on pizza.html
There were a couple of goals to this piece of work. When on this page, and the user is scrolling, pizzas should move around the page. My goal was to get the frame rate up to 60fps whilst this was happening.

### Optimisation: Reduce Number of Pizzas
When looking at this code I noticed that there were a huge number of pizzas being added but the user could only see a few of them. I couldn't get more than 7 across the screen despite the code indicating 8 so I reduced that by 1. I also noticed that the loop indicated adding 200 pizzas! This seemed massively unnecessary. I reduced this to 35 since this gave me the four rows I could see on my monitor plus another one should anyone be running with a larger monitor.

```
document.addEventListener('DOMContentLoaded', function() {
  var cols = 7;                   /* there only seems to be 7 in a row so lowered this */
  var s = 256;
  for (var i = 0; i < 35; i++) {  /* lowered from 200 */
    var elem = document.createElement('img');
    elem.className = 'mover';
    elem.src = "images/pizza.png";
    elem.style.height = "100px";
    elem.style.width = "73.333px";
    elem.basicLeft = (i % cols) * s;
    elem.style.top = (Math.floor(i / cols) * s) + 'px';
    document.querySelector("#movingPizzas1").appendChild(elem);
  }

  // Items is set to an array rather than an HTMLCollection as iteration is faster on the former
  items = [].slice.call(document.querySelectorAll('.mover'));
  updatePositions();
});
```

### Optimisation: Iterate Over Arrays not HTMLCollections
In doing research for this project, I had read that iterating over an HTML Collection can be slow. Looking at the code above, you can see that I've created an array of items, which are all of the moving pizzas selected on line 61, rather than an HTMLCollection. This is also done in the `DOMContentLoaded` event since it didn't need to be happening every time `updatePositions()` was called.

### Optimisation: updatePositions()
This function contains a loop which was doing more than it needed to be doing. Here is the code for this function:

```
function updatePositions() {
  frame++;
  window.performance.mark("mark_start_frame");

  /* this doesn't need to be done on every call of this function */
  //var items = document.querySelectorAll('.mover');

  // Moved out of the loop to save repeating the calculation
  var top = document.body.scrollTop / 1250;

  // Same here
  var length = items.length;

  for (var i = 0; i < length; i++) {
    var phase = Math.sin(top + (i % 5));
    items[i].style.left = items[i].basicLeft + 100 * phase + 'px';
  }

  // User Timing API to the rescue again. Seriously, it's worth learning.
  // Super easy to create custom metrics.
  window.performance.mark("mark_end_frame");
  window.performance.measure("measure_frame_duration", "mark_start_frame", "mark_end_frame");
  if (frame % 10 === 0) {
    var timesToUpdatePosition = window.performance.getEntriesByName("measure_frame_duration");
    logAverageFrame(timesToUpdatePosition);
  }
}

// runs updatePositions on scroll
window.addEventListener('scroll', updatePositions);
```
Firstly you can see that the selection of the moving pizzas is no longer done here. This is explicitly shown by keeping the code but commenting it out. Secondly you can see that the calculation of the `top` variable has been made outside of the loop preventing it from being done repeatedly. You can also see that the length has been taken out of the loop so that `items.length` only occurs the once. I did attempt to get translate working to set the left of each item within the loop as I believe that this is faster than setting it in the manner shown, but I couldn't since the items have to include the left of themselves in the calculation and translateX works in a relative fashion. There's probably a way, but it turned out to be unnecessary.

## Pizza Resizing
The goal here was to get the pizza resizing down to less than 5ms. This wasn't actually that hard and here's what was done.

### `changePizzaSizes()`
This function had two items being repeatedly calculated within a loop. This was unnecessary. Here's a snippet of the new code:

```
 // Iterates through pizza elements on the page and changes their widths
  function changePizzaSizes(size) {
    // These only needs to be done the once, so grab first and perform calcs
    var dx = determineDx(document.querySelector(".randomPizzaContainer"), size);
    var newwidth = (document.querySelector(".randomPizzaContainer").offsetWidth + dx) + 'px';

    // Grabbing these as an array as iterating over an array is faster than doing the iteration for an HTMLCollection
    var pizzaElements = [].slice.call(document.querySelectorAll(".randomPizzaContainer"));
    for (var i = 0; i < document.querySelectorAll(".randomPizzaContainer").length; i++) {
      pizzaElements[i].style.width = newwidth;
    }
  }
```
You can see that I have pulled out the calculation of both `dx` and `newWidth` from the loop. I've also made a new variable named `pizzaElements` which is used to hold an array (not HTMLCollection) in order to iterate faster over each pizza.

I believe that this is all it took. Pizzas now resize in under 5ms!
