---
layout: post
title: "Evaluate the javascript from server side"
date: 2015-08-10 20:28
comments: true
categories: ruby, javascript
---
A problem I run into recently is I am trying to send an AJAX request to fetch grouped options for a select input, the select input is dynamically generated, and I don't want to assign an unique random ID to it, and pass the ID to the server as the DOM selector, I do believe there's a better way of doing this, here's what I come up:


```ruby

# options/index.js.erb

var groupedOptionsHTML = "<%= j helper_method_generate_grouped_options %>";


```




```javascript

var newSelectInput = generateSelectInput();

$.ajax({
  type: "GET",
  dataType: "script",
  url: "http://localhost:3000/options",
  success: function(data, xhr, status) {
  	eval(data);

  	newSelectInput.html(groupedOptionHTML);
  }
})

```

As you can see, the key to retrieve the variable `groupedOptionsHTML` is to evaluate the data returned using `eval(data)`. Please drop a line if you have any better idea to solve this problem, cheers.