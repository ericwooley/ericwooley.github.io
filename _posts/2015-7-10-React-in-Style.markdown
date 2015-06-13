---
layout:     post
title:      "CSS in your browser with React-In-Stye"
date:       2015-06-10 09:50:45
categories: css sass scss react javascript minification
comments:   true
published:  false 

---
**A few quick notes**

1. React-In-Style is not only for react, maybe it is poorly named, but I don't want to change it now. It does however, mesh extremely well with reacts compenent based flow.
2. There is currently no way to autoprefix, it's a difficult issue, and I have a few ideas to handle it. Until then consider something like [Prefix-Free](http://leaverou.github.io/prefixfree)

# Get to the good stuff!

Alright, fine.

## Introducing React-In-Style (RIS)


First off, let me introduce React-In-Style or RIS for short. RIS is differnt than other css solutions because you start with javascript objects, or even json. Don't worry, it still allows for nested selectors and media queries.

RIS is not the first one to think that in browser javascript to css is better. Check out [Pete Hunts Take on css for react components.](https://github.com/petehunt/jsxstyle)

### Advantages of styles as js
The advantages of sending your javascript styles down to the page, and processing it there, is tremendous.

_1) The amount of code transfered down is much less than css, and especially sass._

  Consider the following css style
{% highlight css %}

button {
	border: none;
	border-radius: 0;
	background-color: blue;
}
button:hover {
	border: 1px black inset;
	background-color: green;
	box-shadow: 2px 2px 2px 2px rgba(0,0,0,.8);
}
button:active {
	border: 1px black inset;
	backgrond-color: red;
	box-shadow: 4px 4px 4px 4px rgba(0,0,0,.8);
}
{% endhighlight %}

  We can clean this up with scss!

{% highlight scss %}

button {
	border: none;
	border-radius: 0;
	background-color: blue;

  &:hover, &:active {
    border: 1px black inset;
  }
	&:hover {
	    background-color: green;
	    box-shadow: 2px 2px 2px 2px rgba(0,0,0,.8);
  }
  &:active {
    backgrond-color: red;
    box-shadow: 4px 4px 4px 4px rgba(0,0,0,.7);
  }
}
{% endhighlight %}

  The scss version is much cleaner IMO, less repeated code which is great for readability for us humans,  but what about loading size?
  It turns into this:

{% highlight scss %}

button {
  border: none;
  border-radius: 0;
  background-color: blue;
}
button:hover, button:active {
  border: 1px black inset;
}
button:hover {
  background-color: green;
  box-shadow: 2px 2px 2px 2px rgba(0, 0, 0, 0.8);
}
button:active {
  backgrond-color: red;
  box-shadow: 4px 4px 4px 4px rgba(0, 0, 0, 0.7);
}

{% endhighlight %}

  It actualy grew in size. That sucks. What if we were to use RIS instead.

{% highlight javascript %}


ReactInStyle({
  border: 'none',
  borderRadius: '0',
  backgroundColor: 'blue',

  '&:hover, &:active' {
    border: '1px black inset',
  }
  '&:hover' {
      backgroundColor: green',
      boxShadow: '2px 2px 2px 2px rgba(0,0,0,.8)'
  }
  '&:active' {
    backgrondColor: red',
    boxShadow: '4px 4px 4px 4px rgba(0,0,0,.7)'
  }
}, 'button')

{% endhighlight %}


  The magic part is that it is sent tothe browser just like that, with your javascript file, so there is no need for a second request for css.

_2) You decide when the styles are applied_
