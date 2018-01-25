## [IntersectionObserver: defer, lazy-load](#top)

- [1. Observer vs. Event](#Observer-vs-Event)
- [2. generic structure of an Observer](#generic)
- [3. carousel layout with image lazy-load](#carousel)

**new members of the Observers family**

- [IntersectionObserver](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API):  The Intersection Observer API provides a way to asynchronously observe changes in the intersection of a target element with an ancestor element or with a top-level document's viewport
	- IntersectionObserver is an async non-blocking API
	- IntersectionObserver replaces our expensive listeners on scroll or resize events
	- IntersectionObserver does all the expensive calculations like getClientBoundingRect() for you so that you don’t need to
	- IntersectionObserver follows the structural pattern of other Observers out there so, theoretically, should be easy to understand if you’re familiar with how other Observers work
- [PerformanceObserver(as part of Performance Timeline Level 2 specification)](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserver) 

<h3 id="Observer-vs-Event">1. Observer vs. Event</h3>

- Event
	- reacts synchronously
	- affect main thread's responsiveness
- Observer
	- reacts asychronously
	- Observers can watch something happening on a page, like DOM changes
	- Observers can also watch for page’s lifecycle events
	- Observers can also run some callback functions

**notes of using Observers**

| Reason | solution |
| :------------- | :------------- |
| several observables being passed to a callback function at the same time | the callback function should expect not a single entry but an Array of entries (even though sometimes the Array will contain only one entry in it) |
|pre-computed properties|used to calculate ourselves using expensive (from performance standpoint) methods and properties when using regular events|

[back to top](#top)

<h3 id="generic">2. generic structure of an Observer</h3>

```javascript
let observer = new YOUR-TYPE-OF-OBSERVER(function (entries) {  // entries: Array of observed elements, not a single entry.
  entries.forEach(entry => {
      // do something with each particular entry
  });
});
// Now we should tell our Observer what to observe
observer.observe(WHAT-TO-OBSERVE);
```

[back to top](#top)

<h3 id="IntersectionObserver">3. IntersectionObserver</h3>

**IntersectionObserver** requires a configuration with three main elements: 

| elements |defination |by default|
| :------------- | :------------- |:------------- |
| root|the root element used for the observation. It defines the basic “capturing frame” for observable elements |viewport of browser by default, <br>can be defined any element, like `document.getElementById('your-element')`|
|rootMargin|Defines margin around root element that extends or shrinks the “capturing frame”|margin values in CSS, like `50px 20px 10px 40px`|
|threshold|defines the percentage of such intersection at which Observer should react|`threshold: 0`|

![](https://i.imgur.com/EeZh7TF.png)

```javascript
// structure
const config = {
  root: null,    // avoiding 'root' or setting it to 'null' sets it to default value: viewport
  rootMargin: '0px',
  threshold: 0.5
};
let observer = new IntersectionObserver(function(entries) {
    //...
}, config);
//sample
const img = document.getElementById('image-to-observe');
observer.observe(image);    //IntersectionObserver for one element
//IntersectionObserver for several elements, to iterate and observe each of them separately
const images = document.querySelectorAll('img');
images.forEach(image => {
    observer.observe(image);
});
```

- the code above means that the entry for IntersectionObserver element became initialized and is now controlled by your IntersectionObserver
- but IntersectionObserver will be fired for all observed elements once they are registered, but it doesn't mean that they all intersect our 'capturing frame'

**[IntersectionObserverEntry interface](https://w3c.github.io/IntersectionObserver/#intersection-observer-entry)**

```java
[Constructor(IntersectionObserverEntryInit intersectionObserverEntryInit)]
interface IntersectionObserverEntry {
  readonly attribute DOMHighResTimeStamp time;
  readonly attribute DOMRectReadOnly rootBounds;
  readonly attribute DOMRectReadOnly boundingClientRect;
  readonly attribute DOMRectReadOnly intersectionRect;
  readonly attribute boolean isIntersecting;
  readonly attribute double intersectionRatio;
  readonly attribute Element target;
};
```

|pre-defined and pre-calculated set of properties|defination|
| :------------- | :------------- |
|rootBounds|A rectangle for the “capturing frame” (root + rootMargin)|
|boundingClientRect|A rectangle for the observed element itself|
|intersectionRect|An area of the “capturing frame” intersected by the observed element|

![](https://i.imgur.com/TZxCFM3.png)

IntersectionObserverEntry interface other properties

- `isIntersecting(boolean type)`:  whether the observed element is currently intersecting the “capturing frame” or not, can be used to find out whether the observed element is just entering the “capturing frame” or is already leaving it
	- Microsoft Edge 15, isIntersecting property was not implemented, returning undefined despite full support for IntersectionObserver otherwise. This has been fixed in July 2017 though and is available since Edge 16.
- `intersectionRatio(double type)`:  is similar to threshold property of Observer's config
- `target`:  the original element that had been passed to observe() function of your Observer. Just like event.target

[back to top](#top)

<h3 id="carousel">3. carousel layout with image lazy-load</h3>

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <link rel="stylesheet prefetch" href="https://cdnjs.cloudflare.com/ajax/libs/tachyons/4.9.0/tachyons.min.css">
  <style>
  .screen { min-height: 100vh; text-align:center; text-transform: uppercase; position: relative; }
  .screen h1 {
    margin:0;
    padding:0;
    white-space: nowrap;
    line-height: 100vh;
    color: #fff;
    font-size: 3.5em;
  }
  #first-screen { background: rgb(0,106,221); }
  #second-screen { background: rgb(255,25,129); }
  #fourth-screen { background: #FFDF19; }
  #monitor {
    position:fixed;
    top: 1em;
    right: 1em;
    padding-left: 1em;
    padding-right: 1em;
    background: #000;
    font-family: monospace;
    text-align: center;
    line-height: 2.8em;
  }
  #isIntersecting { color: #33ff00; opacity: 0; transition: opacity .2s linear; }
  img {
    display: block;
    max-width: none;
    width: 100%;
    height: 100%;
    min-height: 100px;
    -o-object-fit: cover;
    object-fit: cover;
  }
  #navbar {
  position: fixed;
  top: 0;
  right: 0;
  left: 0;
  z-index: 10;
  background-color: #222;
  color: #fff;
}
#navbar ul {
  padding: 0;
  margin: 0;
  display: -ms-flexbox;
  display: -webkit-flex;
  display: flex;
  -ms-flex-direction: row;
  -webkit-flex-direction: row;
  flex-direction: row;
  -ms-flex-align: center;
  -webkit-align-items: center;
  align-items: center;
  -ms-flex-pack: center;
  -webkit-justify-content: center;
  justify-content: center;
}
#navbar li {
  list-style: none;
  padding: 1em;
  text-transform: uppercase;
  background: #222;
  color: #fff;
  transition: all .2s linear;
}
#navbar #first.active { background: rgb(0,106,221); }
#navbar #second.active { background: rgb(255,25,129); }
#navbar #third.active { background: #fff; color: #222; }
#navbar #fourth.active { background: #FFDF19; }
  </style>
</head>
<body>
<div class="screen" id="first-screen"> <h1>First screen</h1></div>
<div class="screen" id="second-screen"> <h1>Second Screen</h1></div>
<div class="screen pa3 pa4-ns w-100 w-80-ns center" id="third-screen">
  <main class="cf pa2">
    <div class="fl w-100 w-50-ns ph2">
      <a href="" class="pv2 grow db no-underline black"><img class="db w-100" src="https://farm5.staticflickr.com/4616/39798634951_aa270731f6_k_d.jpg"></a>
     <a href="" class="pv2 grow db no-underline black"><img class="db w-100" src="https://farm8.staticflickr.com/7364/9797687423_89d0b3040b_z_d.jpg"></a>
    </div>
    <div class="fl w-50 w-25-ns ph2">
      <a href="" class="pv2 grow db no-underline black"><img class="db w-100" src="https://farm9.staticflickr.com/8713/16979768317_44c27d64c9_z_d.jpg"></a>
      <a href="" class="pv2 grow db no-underline black"><img class="db w-100" src="https://farm2.staticflickr.com/1717/24895045945_d99447e642_b_d.jpg"></a>
      <a href="" class="pv2 grow db no-underline black"><img class="db w-100" src="https://farm5.staticflickr.com/4745/38898563945_d3974144c0_z_d.jpg"></a>
    </div>
    <div class="fl w-50 w-25-ns ph2">
            <a href="" class="pv2 grow db no-underline black"><img class="db w-100" src="https://farm2.staticflickr.com/1502/24195472655_bc7e4f3582_z_d.jpg"></a>
      <a href="" class="pv2 grow db no-underline black"><img class="db w-100" src="https://farm6.staticflickr.com/5691/23446613226_6ac1a5b1a2_z_d.jpg"></a>
      <a href="" class="pv2 grow db no-underline black"><img class="db w-100" src="https://farm2.staticflickr.com/1594/24342615229_4d407695ef_z_d.jpg"></a>
      <a href="" class="pv2 grow db no-underline black"><img class="db w-100" src="https://farm4.staticflickr.com/3707/9288847355_3aa5800e92_z_d.jpg"></a>
    </div>
  </main>
</div>
<div class="screen" id="fourth-screen"> <h1>Fourth Screen</h1> </div>
<nav id="navbar">
  <ul>
    <li id="first" data-ref="first-screen">First screen</li>
    <li id="second" data-ref="second-screen">Second screen</li>
    <li id="third" data-ref="third-screen">Third screen</li>
    <li id="fourth" data-ref="fourth-screen">Fourth screen</li>
  </ul>
</nav>
<!-- <div id="monitor">
    <div id="isIntersecting">
      <span class="placeholder">0</span>
      images loaded
      </div>
</div> -->
<script>
// nav tab随页面变色
  const sections = document.querySelectorAll('div.screen');
  const configNav = {
    rootMargin: '-50px 0px -55%'
  };
  let observerNav = new IntersectionObserver(function (entries, self) {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        intersectionHandler(entry); 
      }
    });
  }, configNav);
  sections.forEach(section => {
    observerNav.observe(section);
  });
  function intersectionHandler(entry) {
    const id = entry.target.id;
    const currentlyActive = document.querySelector('nav li.active');
    const shouldBeActive = document.querySelector('nav li[data-ref=' + id + ']');
    if (currentlyActive) {
      currentlyActive.classList.remove('active');
    }
    if (shouldBeActive) {
      shouldBeActive.classList.add('active');
    }
  }
  // image lazyLoad
  const images = document.querySelectorAll('[data-src]');
  const config = {
    rootMargin: '0px 0px 50px 0px',
    threshold: 0
  };
  let loaded = 0;
  let observer = new IntersectionObserver(function (entries, self) {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        // console.log(`Image ${entry.target.src} is in the viewport!`);
        preloadImage(entry.target);
        // Stop watching and load the image
        self.unobserve(entry.target);    //unobserve(): Remove the IntersectionObserverRegistration record
      }
    });
  }, config);
  
  images.forEach(image => { observer.observe(image); });

  function preloadImage(img) {
    const src = img.getAttribute('data-src');
    if (!src) { return; }
    img.src = src;
    _updateMonitoring();
  }
  // Just for the monitoring purpose. Isn't needed in real projects
  function _updateMonitoring() {
    const container = document.getElementById('isIntersecting');
    const placeholder = container.querySelector('.placeholder')
    loaded += 1;
    placeholder.innerHTML = loaded;
    container.style.opacity = 1;
  }
</script>
</body>
</html>
```

> [How To Defer, Lazy-Load and Act with IntersectionObserver](https://frontendfoc.us/link/35305/f1b4c54f25)
