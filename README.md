# [SOLVED] How to fully animate the details HTML element with only CSS, no JavaScript

*Now it works in Safari and in Firefox on macOS too!*

**[See the final result live on CodePen](https://codepen.io/jgustavoas/pen/rNQyxWa?editors=1100)**

## Summary
- [Introduction](#introduction)
- [The issues](#issues)
- [Finding the solution](#solution)
- [The HTML](#html)
- [The CSS](#css)
- [Final words](#conclusion)
- [Notes](#notes)


## <a id="intro"></a>Introduction
Front-end development can be challenging when a webpage fails to render or work as intended in some major browser or operating system. It's a problem you can't ignore. It might affect many users and have a negative impact on your brand.

In [my previous repository](https://github.com/jgustavoas/fully-animated-details-element-using-pure-CSS-only), I showed a trick to animate `<details>` element with pure CSS, without any JavaScript. But that solution has two problems: 

1. It doesn't work in Safari and 
2. It doesn't work in Firefox on macOS.

I am glad to have solved those issues, and now the animation works in all major browsers, either on Windows, Linux, Android, macOS, or iOS. 

Plus, the final result is notably better and simpler than the original one.

In this post, I used the same design as the original to compare the changes in the code.


## <a id="issues"></a>The issues
### Safari browser
Safari has limited support for `<summary>` element¹ and `::marker` pseudo-element². In the former, `display: flex` does not work, and you need to use `summary::-webkit-details-marker { display: none }` to remove the default triangle icon³. As for the latter, that browser's support is limited to color and font-size, that is to say no animation. 😒

The issue regarding `::marker` also applies to `::before` and `::after` pseudo-elements. So, we won't see the decorative triangle icon rotating smoothly in Safari, just an abrupt rotation⁴. 👎

Worse, for some reason I couldn't figure out, the original trick using `<label>` and `<input type="checkbox">` does not change `<details>` element's height. I then had to discard it and invent a new one. 😞

My bigger surprise, though, was with Firefox.

### Firefox on macOS
 I already expected that the approach with `:has()` pseudo-class wouldn't work because Firefox still doesn't give default support for that useful CSS feature⁵ (isn't this embarrassing for Mozilla?). But also the adjacent sibling combinator (`+`) approach (which does work on Windows and Linux) has failed on macOS. 😰


### Expectation:
![Image of how the result should be](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/kbpbn9i9h3639x4d7ph3.png)


### Reality:
![Image of how it looks in Safari and Firefox on macOS](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gbps5kotzpmfvdon7nix.png)


## <a id="solution"></a>Finding the solution
I use Linux to develop, so I needed to work directly on a macOS to see the result immediately as I coded to make the tweaks.

I've got that, and after some trials, I found a solution, which consists of these main changes to the original HTML structure:

1. Starting `<details>` element without the `open` attribute (this was the trick then, but now it is not necessary). 👍

2. Discarding `<input type="checkbox">` and `<label>` elements because `<details>` itself will trigger the animation. 🤩

3. Placing `<div>` with the detailed content outside `<details>` element, becoming its sibling instead of its descendant (this is the new trick). 🤔


## <a id="html"></a>The HTML 
### Then
```html
<input type="checkbox" name="detail-one" id="detail-one" />
<details open>
    <summary>
        <label for="detail-one">Click to open and close smoothly with pure CSS</label>
    </summary>
    <div class="content">...</div>
</details>
```

### Now
```html
<details>
    <summary>
        <span>Click to open and close smoothly with pure CSS</span>
    </summary>    
</details>
<div class="content">...</div>
```

*I have replaced `<label>` with a `<span>`, but I kept the same style in CSS, so I could use and animate the triangle icon as a `::before` pseudo-element.*

*Due to Safari's poor support, you can only apply animation to some element or pseudo-element that is a child of `<summary>`, not to the `<summary>` itself. Otherwise, we could discard `<span>` too.*


## <a id="css"></a>The CSS
### `<details>`
#### Then
```css
details {
	max-width: 500px;
	box-sizing: border-box;
	margin-top: 5px;
	background: white;
	transition: max-height 400ms ease-out;

	max-height: 4rem; /* Set a max-height value just enough to show the summary content */
	overflow: hidden; /* Hide the rest of the content */
}
```
#### Now
```css
details {
	max-width: 500px;
	overflow: hidden;
}
```

### `<summary>`
#### Then
```css
summary {
	display: block; /* This hides the summary's ::marker pseudo-element */
}
```
#### Now
```css
summary {
	display: block; /* This hides the summary's ::marker pseudo-element */
}

summary::-webkit-details-marker {
	display: none; /* This also hides the ::marker pseudo-element, but in Safari */
}
```

### `<div class="content">`
#### Then
```css
div.content {
	padding: 0 10px;
	border: 2px solid #888;
}
```
#### Now
```css
div.content { 
	max-width: 500px;
	box-sizing: border-box;
	padding: 0 10px;
	max-height: 0;
	overflow: hidden;
	border: 2px solid transparent;
	transition: max-height 400ms ease-out, border 0ms 400ms linear;
}
```
*The initial value of max-height is 0, which changes when `<details>` opens (see the "Transitions" section next).*

### Transitions
#### Then
```css
input:checked + details,
details:has(input:checked) {
	max-height: 800px; /* Set a max-height value enough to show all the content */
}

input:checked + details label::before,
details:has(input:checked) label::before {
	rotate: 90deg;
	transition: rotate 200ms ease-out;
}
```
#### Now
```css
details[open] + div.content {
	max-height: 800px; /* Set a max-height value enough to show all the content */        
	border-color: #888;
	transition: max-height 400ms ease-out, border 0ms linear;
}

details[open] span::before {
	rotate: 90deg;
	transition: rotate 200ms ease-out;
}
```

*The technique with the adjacent sibling combinator (`+`) is the only one that works now. The technique with `:has()` pseudo-class isn't an option anymore.*


## <a id="conclusion"></a>Final words
In case you haven't noticed yet, this solution does not actually animate the height of `<details>` element. Indeed, it uses the open/closed state of that element to control the animation of the height of an adjacent `<div>` which holds the detailed content. 

Thus, the title *“How to fully animate the `<details>` element…”* is not accurate here because the detailed content is no longer a child of that element.

However, I think the benefits overcome this detail (pun not intended). The user can interact with the design seamlessly. And the developer gets a nice and clean code, rid of working issues, despite the limitations of some browsers.

Thank you for reading!


## <a id="notes"></a>Notes

1. [https://developer.mozilla.org/en-US/docs/Web/HTML/Element/summary#browser_compatibility](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/summary#browser_compatibility)

2. [https://developer.mozilla.org/en-US/docs/Web/CSS/::marker#browser_compatibility](https://developer.mozilla.org/en-US/docs/Web/CSS/::marker#browser_compatibility)

3. [https://developer.mozilla.org/en-US/docs/Web/HTML/Element/summary#default_style](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/summary#default_style)
   
4. [https://developer.mozilla.org/en-US/docs/Web/CSS/::before#browser_compatibility](https://developer.mozilla.org/en-US/docs/Web/CSS/::before#browser_compatibility)

5. [https://caniuse.com/css-has](https://caniuse.com/css-has)

