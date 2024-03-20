# 1) Column -> LazyColumn

First, we should probably convert this screen to a `LazyColumn` to acquire the `.firstVisibleItemIndex` parameter. As a refresher, you can do:

```kotlin
val visibleItemIndex = remember(listState) { listState.firstVisibleItemIndex }
```

to get an automatically recomposing `visibleItemIndex` that contains the integer index of which item is scrolled at the top.

One main issue with how the screen is currently set up, though, is that it's nested inside of a `ModalBottomSheetLayout` as the content. This composable is basically a wrapper that allows a bottom sheet to be pulled up in front of the content.

If we nest a Lazy Column here, though, there will be SO many nests. As such, I recommend pulling the Lazy Column content body ALL out into its own composable. You'll likely want to put this in the `.ui.screens` package as its own file.

It's up to you to decide how you want parameters to be taken into this new screen. Think about if you should be just passing in data values or an entire viewmodel. For example, you could theoretically pass in the whole viewmodel into the screen, or you could just pass in what it needs--like the eatery object, current menu to show, etc.

# 2) Show the Header

Play around with the `visibleItemIndex` above to see when a good spot is for the header to show up. You can make the header conditionally render just with some `animateFloatAsState` opacity--just make sure the header isn't always floating there and clickable when it's faded away. There is likely some `enabled` modifier or other; you can probably ask ChatGPT LOL.

The header itself should be implemented simply just as a separate component layered on top of the `LazyColumn` with a `Box` that contains both. There may be a better library to use, but this seems totally fine for our purpose.

# 3) Make the Header cause a scroll when a menu subsection is clicked

The header should be a separate component that takes in an anonymous function to fire. This function supplies an index for which menu item was clicked, and then should animate scroll to that menu item.

You do not have to visually make the header animate over to a new header item at this point--the button press should only make the scroll occur.

# 4) Animate Header Selection

Finally, what's possibly the most annoying part:

The header should take in an index for which menu item should be animated selected (with the little black indicator bubble behind it; see iOS for what I mean). When this index changes, the header should animate the bubble behind that text to appear.

The annoying part is making it automatically change between these indices when the part of the menu is scrolled to. Basically, you'll need to utilize `visibleItemIndex` or something very similar to determine when each header is scrolled to. The main annoying thing about this is that there's a *dynamic* number of items in this lazy column, so you can't really hardcode this. 

As a result, you might have to make some weird workaround. Here's a workaround ChatGPT suggests:

In the UI component to track:
```kotlin
val targetPosition = remember { mutableStateOf(IntOffset.Zero) }

// ...

Box(
    modifier = Modifier
        .onGloballyPositioned { layoutCoordinates ->
            targetPosition.value = layoutCoordinates.localToRoot(IntOffset.Zero)
        }
        .offset {
            targetPosition.value
        }
) {
    targetComponent()
}
```

In the file somewhere:
```kotlin
LaunchedEffect(visibleItemIndex) {
    if (visibleItemIndex != -1) {
        if (targetPosition.value.y >= thresholdTop && targetPosition.value.y <= thresholdBottom) {
            onTargetScrolledTo()
        }
    }
}
```

You can read through this a bit, but the idea is that `LaunchedEffect` fires whenever `visibleItemIndex` changes. The code will then check if the scrolled to the position of the component whose position is supposedly held by `targetPosition`. It's a messy implementation but should work. 

Instead of just having one explicit `if` here, you have to dynamically check to see which one was scrolled to; and then when it's scrolled to, change the index that you pass into the header to cause the selection to animate.
