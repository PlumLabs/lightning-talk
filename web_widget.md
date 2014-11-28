# Web Widget

### What is it?
* A widget is a stand-alone application that can be embedded into third party sites by any user on a page.
* It's a small application that can be installed and executed within a web page by an end user.
* It's a "chunk of a web page"

### Options to create one Widget (Iframe vs. Javascript)
[Iframe]:
advantage will work when the visitor has Javascript disabled. Besides, it is isolated from the client's website, meaning you can't access his document and he can't access yours.

Javascript (FTW): customers to adjust the widget's style to match their

### How they looks then?
In wonderland for example if I have a REST resource like the following

```
GET http://www.site.com/user/1/votes
```

and it returns

```
{ chicho: 3, edu: 3, rodri: 3, ptak: 3}
```
then from the site `http://www.origin-site.com` I would like to add something like the following to use it

```javascript
 $.ajax({
    url: "http://www.site.com/user/1/votes",
    type: "GET",
    dataType: "json",
    timeout: 5000,
    success: function(rs) {
      $.extend(user.data.voting, rs.user_voting);
    }
});
```
But that's not possible because the [Same Origin Policy], it's a browser security to prevent Cross-site Request Forgery attacks ([CSRF])

> An origin is defined by:
> 1. Protocolo
> 2. Host (Dominio/Dirección de IP)
> 3. Puerto
>
> If something of above doesn't match then browser detect as an external >resource and wont load the resource.
> e.g: in http://localhost:3000 try to get:
> - https://localhost:3000/algo – Different protocol
> - http://localhost:1100/algo – Different port
> - http://www.google.com/algo – Different host
> - http://localhost:3000/algo – Correct

### JSONP to rescue
It's JSON + Padding. So far we know that we can't use AJAX because it's an external site, but we know some tags where is allowed to use external references.

```
<script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/jquery/1.8.1/jquery.min.js"></script>
```

In the above tag there is no check of Origin Policy, so could we use it to load our json in the followig way?

```
<script type="text/javascript" src="http://www.site.com/user/1/votes"></script>
```

the answer is NOP, because browser doesn't know what to do with a json there. But what if response is a function instead a simple JSON?

```
our_callback({
    chicho: 3,
    edu: 3,
    rodri: 3, ptak: 3
});
```

then we need to create the function which will get the JSON data as a parameter:

```
function our_callback(json_data) {
    // do stuff with json_data
}
```

jQuery is so good with us that it does all the ugly work for us, so we could do all the above using native jQuery in the following way:

```
$.ajax({
  url: "http://www.site.com/user/1/votes?callback=",
  dataType: 'jsonp'
}).done(function(response) {
  console.dir(response);
});
```

### Deliver widgets
So how we can use all this to give to our clients an easy way to add our widget in their pages?

We should give to them some code to add our widget in their pages, something like this

```javascript
<script type='text/javascript'>
MySiteWidget = {};
MySiteWidget.limit = 5;
MySiteWidget.default_query = "users";
</script>
<script src='//www.mysite.com/widget.js' type='text/javascript'></script>
<div class='mysite-widget'></div>
```

There are a lot of variants of this `widget.js` script in internet, but all of them use the same technique...an anonymous function and a call to that function. The variables we create in our functions won’t interfere with the rest of the page.

```
var foo = "Hello World!";
document.write("Before our anonymous function foo means '" + foo + '".');

(function() {
    // The following code will be enclosed within an anonymous function
    var foo = "Goodbye World!";
    document.write("Inside our anonymous function foo means '" + foo + '".');
})(); // We call our anonymous function immediately

document.write("After our anonymous function foo means '" + foo + '".');
```

this `widget.js` file will need to load all libraries needed by our widget because we can’t rely on having jQuery loaded on the host page, we’re going to load jQuery dynamically using JavaScript:

```
(function() {

// Localize jQuery variable
var jQuery;

/******** Load jQuery if not present *********/
if (window.jQuery === undefined || window.jQuery.fn.jquery !== '1.4.2') {
    var script_tag = document.createElement('script');
    script_tag.setAttribute("type","text/javascript");
    script_tag.setAttribute("src",
        "http://ajax.googleapis.com/ajax/libs/jquery/1.4.2/jquery.min.js");
    if (script_tag.readyState) {
      script_tag.onreadystatechange = function () { // For old versions of IE
          if (this.readyState == 'complete' || this.readyState == 'loaded') {
              scriptLoadHandler();
          }
      };
    } else { // Other browsers
      script_tag.onload = scriptLoadHandler;
    }
    // Try to find the head, otherwise default to the documentElement
    (document.getElementsByTagName("head")[0] || document.documentElement).appendChild(script_tag);
} else {
    // The jQuery version on the window is the one we want to use
    jQuery = window.jQuery;
    main();
}

/******** Called once jQuery has loaded ******/
function scriptLoadHandler() {
    // Restore $ and window.jQuery to their previous values and store the
    // new jQuery in our local jQuery variable
    jQuery = window.jQuery.noConflict(true);
    // Call our main function
    main();
}

/******** Our main function ********/
function main() {
jQuery(document).ready(function($) {
        var css_link = $("<link>", {
            rel: "stylesheet",
            type: "text/css",
            href: "<%= URI.join(root_url, path_to_stylesheet("widget.css")).to_s %>"
        });
        css_link.appendTo('head');

        var jsonp_url = <%= raw widgets_url(format: "json").to_json %>;
        $.ajax({
          url:       jsonp_url,
          data:      MySiteWidget,
          dataType:  "jsonp",
          success:   function(data) {
            // modify this part
            $.each(data, function(i,d) {
              $('.mysite-widget').append(d.template)
            })
          }
        });
    });
}

})(); // We call our anonymous function immediately
```

### CORS
 JSONP is controversial, it's a *work-around* to Same Origin Policy. CORS (Cross-Origin Resource Sharing) is a solution to this problem, the goal is add a property in HTTP response header to allow access to the server as a Cross Domain.

License
----
MIT


Resources
----
* http://alexmarandon.com/articles/web_widget_jquery/
* http://www.stefanwienert.de/blog/2013/10/15/intro-on-making-a-javascript-widget/
* http://fernetjs.com/2012/09/jsonp-cors-y-como-los-soportamos-desde-nodejs/
* http://en.wikipedia.org/wiki/Cross-origin_resource_sharing

**Free Software, Hell Yeah!**

[Iframe]:https://developer.mozilla.org/en/docs/Web/HTML/Element/iframe
[Same Origin Policy]:https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy
[CSRF]:http://en.wikipedia.org/wiki/Cross-site_request_forgery
