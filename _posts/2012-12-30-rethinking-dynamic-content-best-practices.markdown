---
layout: post
title: Rethinking Dynamic Content Best Practices
date: December 30, 2012
--- 

There's an awesome screencast detailing best practices with Dynamic Content
that was published in May 2010 [link]

A quick recap of the best practices:

1. Works fine with JavaScript disabled.
2. It is possible to "deep link" to specific content. 
3. The browsers back button and forward button work as expected.

## The Problem with URL hashes

For one individual user, the exisiting demo [link] meets the criteria just fine. 
Does this really work with JavaScript disabled?

Consider the following scenario:

1. I've got a fancy browser with Javascript enabled. I'm browsing the demo site, and I find a great product I'd like to share with a friend.

2. I copy the url 'http://example.com/#awesome-product', and send it to my friend.

3. My friend doesn't have javascript enabled. (s)he opens the link in her browser, and is confused that the awesome product doesn't load as expected.

4. (s)he gets confused/frustrated and swears to never visit example.com again.

THIS IS BAD UX!!

Today, we'll be improving the existing demo such that doesn't rely on the hash. 

If you'd like to follow along, download the original demo: [link]

or just get the final product: [link]

## Modernizr for progressive enhancement

If you're not using Modernizr yet, go get it. It's the easiest way (by far) for checking a browsers capabilites. 

Since we'll be playing with the HTML5 history api, we only need to check the 'History' checkbox. Download it!

include it in our example
	
{% highlight html %}
<script type='text/javascript' src='js/modernizr.js'></script>
{% endhighlight %}

Testing for HTML5 history support is super easy:
{% highlight javascript %}
$(function(){
	if(Modernizr.history){
		//history is supported; do magical things
	} else {
		//history is not supported; nothing fancy here
	}
});
{% endhighlight %}

We're going to take everything from the original demo, and only use it if Modernixr.histroy returns true (HTML5 history is supported).

{% highlight js %}
$(function(){
	if(Modernizr.history){
		//history is supported; do magical things

	var newHash      = "",
	        $mainContent = $("#main-content"),
	        $pageWrap    = $("#page-wrap"),
	        baseHeight   = 0,
	        $el;
	        
	    $pageWrap.height($pageWrap.height());
	    baseHeight = $pageWrap.height() - $mainContent.height();
	    
	    $("nav").delegate("a", "click", function() {
	        window.location.hash = $(this).attr("href");
	        return false;
	    });
	    
	    $(window).bind('hashchange', function(){
	    
	        newHash = window.location.hash.substring(1);
	        
	        if (newHash) {
	            $mainContent
	                .find("#guts")
	                .fadeOut(200, function() {
	                    $mainContent.hide().load(newHash + " #guts", function() {
	                        $mainContent.fadeIn(200, function() {
	                            $pageWrap.animate({
	                                height: baseHeight + $mainContent.height() + "px"
	                            });
	                        });
	                        $("nav a").removeClass("current");
	                        $("nav a[href="+newHash+"]").addClass("current");
	                    });
	                });
	        };
	        
	    });
	    
	    $(window).trigger('hashchange');
	} else {
		//history is not supported; nothing fancy here
	}
});
{% endhighlight %}

## Manipulate the history with HTML5 history API

And now, the conversion. Instead of changing the hash value whenever a nav a is clicked:
{% highlight js %}
$("nav").delegate("a", "click", function() {
    window.location.hash = $(this).attr("href");
    return false;
});
{% endhighlight %} 
we can change the url and history without a page refresh:
{% highlight js %}
$("nav").delegate("a", "click", function() {
	// just change the url and add a history entry
    history.pushState(null, null, $(this).attr("href"));
    return false;
});
{% endhighlight %} 

## handle browser back and forward button clicks