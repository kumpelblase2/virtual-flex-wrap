# virtual-flex-wrap

`virtual-flex-wrap` is a Vuejs component to allow for displaying a virtualized grid of elements using flexbox row wrapping and is very much inspired by many of existing components to display a virtualized collection elements.
However, it seemed weird that there are only two categories: simple lists or grid-like display with absolute positioning. 
Neither of these fit what I was looking for which thus resulted in `virtual-flex-wrap`.

This is both an attempt to use flexbox for the layout of such a component and also to provide a rather simple solution to this kind of problem.
As a result, there are some limitations for using this:
1. The number of columns is the same for each row
2. The height of each row is the same
3. You need to take care of sizing your components

These are very much fundamental to the design of this and as a result you might need to look for something else if even one does not fit your use-case.

You can see an example of how to use under `dev` which you can start using `npm run serve`.

## Props

`items`: The list of items that you want to display. Each of these will be passed to the specified component to be displayed. __Required__

`columnCount`: The number of columns per row. __Required__

`itemHeight`: The height of the items, i.e. the height of the rows. __Required__

`cellComponent`: The component used to render an item. __Required__

`keyProvider`: A function to provide a key for each item. By default, it will use the `id` property on the item.

`rowBufferCount`: How many rows to buffer before and after to allow for fluent scrolling. Default: 1.

`visibilityCheckInterval`: Interval in ms between checks if the user scrolled too far. Default: 100.

`scrollContainer`: Container to use for detecting if the user scrolled outside of the visible area. Default: this component
