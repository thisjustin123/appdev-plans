### This is the breakdown for Eatery Detail's Refactor, flowed to the brim

## 1) Problem Statement

Our main issue with the current eatery detail screen is that the logic for which eatery to show is hidden in the screen UI code. The high level goal is to dig that code up into the viewmodel to basically have the following behavior:

1. The screen is initialized showing the current or next/nearest meal to now.
2. The user can then select a different meal to show.

Sounds pretty simple, but there are some roadblocks. Namely...

### `eateryFlow`

Because of some logistics with how backend works, we have to emit the eatery selected by the user as a Flow. This is because that eatery may not have been fully loaded yet. The flow here represents updates for when that eatery is loading and then finishes loading.

This makes it difficult for us to initialize the eatery detail meal to, say, point (1), since the eatery may not be successfully loaded on screen launch.

## 2) Flow Solution

I have created 3 flows as the solution here. Sounds like a lot, and it is. But it should work. Here's the breakdown for each:

### `_curMeal`

**Purpose**: Constantly emits the current or next/nearest meal to the current time, mapped from `eateryFlow`, regardless of what else the screen is doing right now.

**Implementation**: Map from `eateryFlow`. If success, get the current or next/nearest meal. Otherwise, continue sending null.

### `_userSelectedMeal`

**Purpose**: Emits the meal that the user has expressly selected by the bottom sheet.

**Implementation**: When the user submits a new event selection, call some new function in the view model to set this flow value over to what the user selected.

And finally, how it all fits together:

### `mealToShow`

**Purpose**: Emits the meal that the screen should actually show. Note that this is the only non-private flow here.

**Implementation**: If `_userSelectedMeal` is non-null, indicating that the user has indeed selected a different meal, emit that meal. Otherwise, emit the `_curMeal`.

## 3) Idea

The idea here is that we have separate notions for the default meal to show (`_curMeal`) and the meal the user has selected (`_userSelectedMeal`). We use flow combination to combine these two flows and emit whichever is appropriate to show given the scenario.

This avoids the issue of initialization by making `_curMeal` a flow that responds automatically to updates from `eateryFlow` while still exposing the exact same interface to the screen as `mealToShow`. Note that `_curMeal` and `userSelectedMeal` are PRIVATE.

A final REALLY important point here is in the *spec comments.* There are a few cases where we can assume a key value emitted is non-null. Before getting started, please convince yourself that these are actually true! And of course, ask any questions you may have.
