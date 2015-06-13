---
layout:     post
title:      "CSS in your browser with React-In-Stye"
date:       2015-06-10 09:50:45
categories: css sass scss react javascript minification
comments:   true
published:  true

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


  The magic part is that it is sent to the browser just like that, with your javascript file, so there is no need for a second request for css.

_2) You decide when the styles are applied._

Instead of having a css file, which contains all your files. You could apply the styles for you component when as they are loaded if you are using a cdn and loading individual components. You also could apply the styles right before you use them. It's all done in javascript. Apply the styles whenever works best for your project!

_3) Share you variables between your code and your styles._

With react in style, you can use JSON files or javascript objects to keep track of your styles. Need to set a timeout for something to happen when an animation finishes? Place that time in a style constant and use it in both your style and your code. From then on you can change that one number, and the animation time will reflect in your code and in your css.

Another benefit of this is that it's self documenting.

{% highlight javascript %}

// All usable in code as well!
// remember to append 'px', to your styles when you put the objects in react-in-style.
var StyleConstants = {
  swipeInAnimationTime: 0.3,
  swipeOutAnimationTime: 0.6,
  swipeStartingLocation: -400,
  swipeEndingLocation: 0,
  panelWidth: 390
}

{% endhighlight %}

You can also programatically manipulate the styles, for different situations. Functions, varaibles, and libraries are all at your disposal.

## Best practices

One of the worst parts of css is getting selector conflicts. Like sass, RIS scoped everyting to the selector you pass in. It's a good idea to give your  components a unique class name, like `.ew-my-component` (ew being my initials).

You should also only give each component the minimum amount of style it needs to function. in the case of our button, assuming our requirements were for  colors and active/hover states. You wouldn't want to apply any kind of positioning or width on this component. Leave positioning of a component to it's parent.

{% highlight javascript %}

// apply styles to the button
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
}, '.ew-colorful-button')

//  Style the button within a modal to add positioning
ReactInStyle({
  background: 'rgba(0, 0, 0, .95)',
  position: 'absolute',
  top: 0,
  right: 0,
  left: 0,
  bottom: 0,

  // If you need to override defaults, styles applied here would override the default styles because this selector ends up being more specific.
  '.ew-colorful-button' : {
    position: 'absolute',
    top: '20px',
    right: '20px'
  }
  // ... more styles for content or whatever
}, '.ew-lightbox-modal')
{% endhighlight %}

This way we have a completely reusable button, which is unlikely to polute the global varaibles, and has all the convenience of a native component, with all the flexibility of styles that you are used too.
