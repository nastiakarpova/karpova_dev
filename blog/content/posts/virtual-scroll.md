+++
title = 'Virtual scroll with functional JavaScript'
summary = "A detailed step-by-step guide on how to build a virtual scroll using JavaScript and a functional library, Ramda."
date = 2023-09-26T15:28:07+02:00
draft = false
+++

*A detailed step-by-step guide on how to build a virtual scroll using pure 
JavaScript and a functional library, Ramda.*

Virtual scroll is a powerful technique that enables web developers to 
present large datasets in a user-friendly and efficient manner. By 
rendering only the visible content and dynamically managing the dataset as 
the user scrolls, virtual scrolling enhances performance, improves the 
user experience, and reduces DOM clutter. Whether you're building a 
data-intensive application or simply looking to optimize your user 
interface, mastering virtual scrolling is a valuable skill that will 
elevate your web development projects.

Below are the steps to build and understand a virtual scroll. I will break 
down the problem into smaller chunks, which will help us all to really 
grasp what is going on behind the scenes of *virtual scrolling*.

Here are the sub-problems we will tackle:
1. Display number of  items using buttons
2. Show items in their “corresponding” place
3. Add the scroll functionality

## 1. Display number of items using buttons
We will work with a list of 10 items. This acts as data that we fetched 
from the database.

```javascript
  
	const items = ["Item 0", "Item 1", "Item 2", "Item 3", 
		"Item 4", "Item 5", "Item 6", "Item 7", "Item 8", "Item 9"];
  	
```
To break down the problem, I started with two buttons for incrementing and 
decrementing a number. That number corresponds to an index in our tiny 
dataset of 10 items.
So, we will add two buttons into the html with an onclick event with two 
different callback functions:

```html
  
	<div>
		<button onclick="increment()">Increment</button>
		<button onclick="decrement()">Decrement</button>
	</div>
  
```
Those functions will do a lot of work for calculating which items from 
**items** we should display, when the user clicks either of the buttons. 
To keep track of that, we need a count, the number of items, and a number 
of items we want to show.
```javascript
  
	let count = 0;
	const shownItemsCount = 3;
	const totalItemsCount = R.length(items);
  
```
We also want to have a cap for count, since we only have 10 items in this 
case, and we treat count as index, the maximum count should be 9. But 
because of how the items will be rendered, the maximum count can be 
totalItemsCount - shownItemsCount. In our case, it is 7. 
The same goes for values below zero, they don’t exist in the array. If we 
don’t do this measure, what will happen is the displayed items will 
disappear, one by one, when we reach the limit of the array of either side 
by clicking the buttons.

Finally, those functions will re-render the page, so the correct items can 
be shown.

Those are the functions:
```javascript
  
	function increment() {
		(count < totalItemsCount - shownItemsCount) ? count++ : count;
		renderList();
	}
  
```
```javascript
  
	function decrement() {
		(count > 0) ? count-- : count;
		renderList();
	}
  
```
Let’s go over what happens in the **renderList** function. Here, we want 
to take a portion of items from the array and get it ready for display. 
Then, render that data onto the page.
```javascript
  
	function renderList() {
		const visibleItems = R.pipe(
			R.slice(count, count + shownItemsCount),
			R.map((x) => `<div>${x}</div>`)
		)(items)

		document.getElementById("list-container").innerHTML = R.join("", 
	visibleItems);
	}
  
```
Now we got a virtual scroll…with buttons. :)

## 2. Show items in their “corresponding” place

Next step is show the items in their corresponding place, as if they are 
invisible until you “scroll” to their location. For this, we need to know 
the height of the container that can hold all of the items, together with 
the height of a single row that displays an item from **items**. 

```javascript
  
	const rowHeight = 20;
	const totalListHeight = rowHeight * totalItemsCount;
  
```
Another variable we need is an offset, by which we need to visually move 
the shown items down. For example, if we increment the **count** to 3, 
that means we will be displaying “Item 3”, “Item 4”, and “Item 5”, and we 
want to offset the reserved place for “Item 0”, “Item 1”, and “Item 2”. 
Hence,

```javascript
  
	let offsetY = count * rowHeight;
  
```

That offset variable will become very handy with CSS **transform** 
property and **translateY** transformation function, which does exactly 
what we want: move an element vertically.

So, we will rewrite the **renderList** function:

```javascript
  
	function renderList() {
		offsetY = count * rowHeight;

		const visibleItems = R.pipe(
			R.slice(count, count + shownItemsCount),
			R.map((x) => `<div style="height: ${rowHeight}px;
			transform: translateY(${offsetY}px)">${x}</div>`)
			)(items)
			
		document.getElementById("list-container").innerHTML = `
		<div style="height: ${totalListHeight}px;">
			${R.join("", visibleItems)}
		</div>`;
	}
  
```
Perfect! Now it works with an offset, so we are very close to make it work 
with a scroll event. Let’s dive in!

## 3. Add the scroll functionality

First, let’s replace click event (and corresponding buttons) with a scroll 
event. We attach it to the same old **list-container** div. We also want 
to set the ability to scroll with CSS. Plus, we need a viewport div with a 
fixed height.
```css
  
	#list-container {
		overflow: scroll;
	}

	#viewport {
		height: 60px;
	}
  
```
Since, there are no buttons, we are not going to use the callback 
functions **increment** and **decrement**. However, we will take our 
learnings and create a new function **moveItems**. Because we are not 
clicking but scrolling, calculating count will require a bit more math 
than adding or subtracting 1 to it.

We want to know where we are when we scroll. And one important property 
that will help us immensely with calculating our position from the top of 
the list-container to the place we have scrolled to is **scrollTop**. It 
is attached to **list-container**, so we can use it with a 
**scrollEventHandler**. 
```javascript
  
	function scrollEventHandler(event) {
		const scrollTop = listElement.scrollTop;
		moveItems(scrollTop);
	}
  
```
To calculate **count**, we have to divide the distance scrolled by a 
constant row height. And the same restrictions apply here: we don’t want 
to go below 0, and above 7.
```javascript
  
	function moveItems(scrollTop) {
		count = Math.floor(scrollTop / rowHeight);
		count = R.min(totalItemsCount - shownItemsCount, count);
		count = R.max(0, count);
		renderList();
	}
  
```
And there you have it! A virtual scroll, implemented with vanilla 
JavaScript (and Ramda).

To make it a better user experience for our ~~fussy~~ sophisticated users, 
we can add buffer to the **visibleItems**. That will produce a smoother 
scroll, without empty spaces on either sides of the rendered list.
Also, let's make it a bit more pretty and give it a more catchy data than 
Item 1, Item 2...

After a bit of tweaking, here is my result. All the code can be found on 
my GitHub [repo](https://github.com/nastiakarpova/virtual-scroll). 
