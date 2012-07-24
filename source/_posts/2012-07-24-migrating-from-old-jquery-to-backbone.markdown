---
layout: post
title: "Refactoring into Backbone"
date: 2012-07-24 15:24
comments: true
categories: javascript backbone

---

##Part 1

[Backbone.js](http://backbonejs.org/) has exploded on to the scene over the past several months. Todo lists and Web3.0 full-JavaScript web applications have popped up like weeds but there's not much information on integrating Backbone into an existing site.

In the typical style of progressive enhancement, most sites are still built to work statically with some jQuery magic layered on top to give it a slicker feel. Usually, the developers of these sites dismiss Backbone.js (and other similar frameworks) as being great for pure js front-ends, but not so useful for their work. "This page is just too simple" ~~they~~ I said.

While I've been using Backbone.js for a few months now to some interesting moonlight work, only relatively recently have I been integrating Backbone.js into my main work.

##A Brief Tour

For those unfamiliar, Backbone.js is a very lightweight, decoupled-ish MVC framework that provides 4 basic things:

1. Models
2. Collections
3. Views
4. Router

Along with some bonuses like using [Underscore.js](http://underscorejs.org/) and jQuery (Zepto compatible!) internally (which gives *yet another* bonus of forcing more JS devs to discover the wonder that is Underscore).

While building in and making event firing and handling core to the system makes Backbone's Models and Collections incredibly invaluble to highly interactive application UIs, the Backbone.View is the single most understated powerhouse in Backbone.

In a nutshell, all a Backbone.View really does is provide DOM element management with easy delegate event hooks. Yet even though it remains so simple and easy to replecate, the organization factor cannot be understated and thus becomes invaluable for refactoring existing code.

##First Steps

Take, for instance, a simple list of items that can be appended to and items deleted from. This is a goss oversimplification of a refactor I recently did but the example will hold.

Our HTML is structured such that we can interact with the application in the absence of JavaScript (or we at least preload templates/DOM to reduce page load times):

``` html
<html>
<!-- ... -->
<div id="widget">
	<ul>
		<li><span class="item" data-id="1">Item 1 <a href="/url/del">Delete</a></span></li>
		<!-- ... -->
	</ul>
</div>
<!-- ... -->
<script>
	$('#widget .item a').click(function(e){
	    var $this = $(this)
	      , $parent = $this.parent()
	      , id = $parent.data('id');
	    
	    $.ajax(/* … */);
	});
	
	//...
</script>
</body>
</html>
```

Beginning our refactor process is actually not too difficult. We can assume either there is more code to this page (say, an AJAX call to fill in the list dynamically) but not really necessary.

We can immediatley break out all of our jquery code into a new Backbone.View:

``` javascript scripts/foo.js
(function(namespace, $){
	
	namespace.Bar = Backbone.View.extend({
		events: {
			'click #widget .item a': 'clickItemDelete'
		}
		
      , initialize: function() {
	        
        }
      , render: function() {
	        //...
	        return this;
        }
      , clickItemDelete: function(e) {
	        var $this = $(e.currentTarget)
	          , $parent = $this.parent()
	          , id = $parent.data('id');
	        
	        $.ajax(/* … */);
        }
	});
	
})(window.Foo = window.Foo || {}, jQuery);
```

Notice our click handler immediately translated? Given that jQuery is used internally, *almost* everything remains the same execpt for it being broken into two pieces: the selector (and event) visit our events attribute while the body of the event goes to inhabit it's own method. 

**There is one caveat though.** Backbone prefers to keep track of the View object so it binds every event listener to the Backbone.View instance. So, unlike jQuery, `this` will be bound to the view instead of the element being acted upon. Easily enough, however, but `event.currentTarget` is essentually going to be your drop in replacement (which you can see in the above example I've already done so).

Knowing this any event listeners will be your easiest first target. For now we can dump the rest of our JavaScript code into the `render()` function.

After you've moved all your code out, you include whatever file contains your new Backbone.View and get thing started:

``` html
<html>
<!-- ... -->
<div id="widget">
	<ul>
		<li><span class="item" data-id="1">Item 1 <a href="/url/del">Delete</a></span></li>
		<!-- ... -->
	</ul>
</div>
<!-- ... -->
<script>
    $(function(){
        var $widget = $('#widget');
        
        var view = new Foo.Bar({
            el: $widget
        }).render();
    });
</script>
</body>
</html>
```

By overriding the element to our nearest logical parent, we avoid the unnecessary task of having to re-render any interfaces and take advantage of hte pre-assembled DOM. As well this lowers the task load in converting the JavaScript code and possible rewrite event selectors.

At this very moment what we have is all of our previous mixed and matched jQuery front-end code tucked away in a Backbone.View with our events actually split up and refactored properly. Some may wonder why bother if this is all it organizes, and while honestly your code organization should improve instantly (no more scattered event listeners) the … "most convincing" is yet to come.

From here we begin the longer task of evaluating the purpose of each gropuing of JS code. What purpose it serves determines when you organize it for the first pass:

* Is it trying to rending something to the DOM? Probably belongs in the `render()` method.
* Is it declaring variables and setting up requried pre-conditions? Probably belongs in the `initialize()` method.
* Is it utility functions (like url detection methods, etc)? Each of these should be provided their own method on the view itself *unless* it is a globally utilized function (in which case should live on its own). But it should already be separate anyways :)

Exactly how we approach this will be subject to my next post tomorrow.