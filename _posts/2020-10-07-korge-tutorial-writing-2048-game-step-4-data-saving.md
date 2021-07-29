---
layout: post
title: "KorGE Tutorial - Writing 2048 game. Step 4 - Data saving"
author: rezmike
categories: [ Tutorials ]
image: assets/images/titles/korge-tutorial-writing-2048-game-step-4-data-saving.jpg
---

In [the previous step](https://blog.korge.org/korge-tutorial-writing-2048-game-step-3-animation/) we have added movement and
merging of blocks with animation. The game is already playable. But to make it complete, we need to add score points. In
this step we will do it, and along with that we will also add another feature – saving data of the current game
(game field and score). In the end, we'll have a complete 2048 game:

![](/assets/images/sample%20(1).gif)

## Score points

Let's see what we want from the score. First, the score should be updated every time a player merge blocks, and the
number of points should be increased by the sum of values of blocks that were merged. So if the player merge two blocks
with the number 8 on them, we should increase the score by 16 points. Second, when the current score starts to exceed the
best score, the best score should be updated accordingly.

Later, we'll add saving both scores in a storage and getting them from it – I'll describe the logic later. Now let's
just write code that calculates them and updates corresponding properties (fields).

## Score properties

Since we will access score variables from different functions in `main.kt` file, let's define these variables as
top-level properties. Now we should think what would be the best type for them. What is the use of these properties?
First, we need to define their initial values. Second, we need to update these values in the `main()` function where we
can get saved properties from `History`. Third, we need to update the values when blocks are merged. And forth, when the
properties are updated, we need to update the corresponding `text` views and we also need to save new values in the
storage.

One of the ways we can achieve this is by defining types of properties as `Int` and writing custom getters and setters
for them:

```kotlin
val score: Int
	get() = ...
	set(value) { ... }
val best: Int
	get() = ...
	set(value) { ... }
```

But if we do so, we will need to have access to the storage, `text` views, block merging and the fields themselves in
the same place. If you look at our project code, you'll see that it's not easy to implement this approach as we have all
these values in different places and they all can't be accessed in the same place.

The better way is to use `ObservableProperty` class provided by `korio` library. This class has a constructor with the
initial value, a `value` property to get the current value, and two very useful functions – `observe(handler)`
and `update(value)`. The first one allows you to add a new `hander` that will observe and handle new values of the property
when the property is updated. The second one allows you to update the property value. If we use this class for our score
properties, we'll be able to observe and update them from different places inside and outside our `main.kt` class. So
let's use it!

## ObservableProperty in action

First, we need to define the properties themselves. It's actually quite easy:

```kotlin
val score = ObservableProperty(0)
val best = ObservableProperty(0)

//...

suspend fun main() = ...
```

Here we have `score` property for the current score and `best` for the best score. We set initial values to 0 since we
can't get the values from the storage here (since, as you will see later, `NativeStorage` requires an instance
of `Views`).

Instead, we'll get the storage and update the best score in our `main()` function (but we'll do it a bit later):

```kotlin
suspend fun main() = Korge(...) {
	font = resourcesVfs["clear_sans.fnt"].readBitmapFont()

	//TODO: here we'll get the storage
	//TODO: and here we'll update the best score
	//best.update(...)

	...
}
```

Now we need to define the property observers. The first one will handle updates of the `score` property and update
the `best` property value. The second one will handle update of the `best` property and save its value in the storage.

```kotlin
suspend fun main() = Korge(...) {
	...
	//best.update(...)

	score.observe {
		if (it > best.value) best.update(it)
	}
	best.observe {
		//TODO: here we'll update the value in the storage
	}

	...
}
```

Now let's go to the `text` views that show the score. As you may remember, we have created them in one of the previous
articles. The first `text` view is for the best score value:

```kotlin
suspend fun main() = Korge(...) {
	...

	val bgBest = roundRect(...) {
		...
	}
	text("BEST", ...) {
		...
	}
	// here it is
	text("0", cellSize * 0.5, Colors.WHITE, font) {
		setTextBounds(Rectangle(0.0, 0.0, bgBest.width, cellSize - 24.0))
		alignment = TextAlignment.MIDDLE_CENTER
		alignTopToTopOf(bgBest, 12.0)
		centerXOn(bgBest)
	}

	...
}
```

We need to set the initial string value in this `text` view to the one that the `best` property has. Then, we need to
add an observer that will update this string value when the property updates. The previous code block with the added
changes is here:

```kotlin
suspend fun main() = Korge(...) {
	...

	val bgBest = roundRect(...) {
		...
	}
	text("BEST", ...) {
		...
	}
	// here is a new change
	text(best.value.toString(), cellSize * 0.5, Colors.WHITE, font) {
		setTextBounds(Rectangle(0.0, 0.0, bgBest.width, cellSize - 24.0))
		alignment = TextAlignment.MIDDLE_CENTER
		alignTopToTopOf(bgBest, 12.0)
		centerXOn(bgBest)
		// and here is another one
		best.observe {
			text = it.toString()
		}
	}

	...
}
```

The second `text` view is for the current score value:

```kotlin
suspend fun main() = Korge(...) {
	...

	val bgScore = roundRect(...) {
		...
	}
	text("SCORE", ...) {
		...
	}
	// here it is
	text("0", cellSize * 0.5, Colors.WHITE, font) {
		setTextBounds(Rectangle(0.0, 0.0, bgScore.width, cellSize - 24.0))
		alignment = TextAlignment.MIDDLE_CENTER
		alignTopToTopOf(bgScore, 12.0)
		centerXOn(bgScore)
	}

	...
}
```

And here are the same changes that we've added to the previous `text` view:

```kotlin
suspend fun main() = Korge(...) {
	...

	val bgScore = roundRect(...) {
		...
	}
	text("SCORE", ...) {
		...
	}
	// here is a new change
	text(score.value.toString(), cellSize * 0.5, Colors.WHITE, font) {
		setTextBounds(Rectangle(0.0, 0.0, bgScore.width, cellSize - 24.0))
		alignment = TextAlignment.MIDDLE_CENTER
		alignTopToTopOf(bgScore, 12.0)
		centerXOn(bgScore)
		// and here is another one
		score.observe {
			text = it.toString()
		}
	}

	...
}
```

Now, let's write code that will update score after the merges happen.

## Updating score after merges

The place where we should write new code is here:

```kotlin
fun Stage.moveBlocksTo(direction: Direction) {
	...

	if (map != newMap) {
		isAnimationRunning = true
		showAnimation(moves, merges) {
			map = newMap
			generateBlock()
			isAnimationRunning = false
			// here
		}
	}
}
```

The code is actually very simple. We define a new variable `points` with the number of points that should be added to
the current score, we go through the `merges` list and add the block's value to the `points` variable. Then we just add
collected `points` to the current score and update it.

```kotlin
fun Stage.moveBlocksTo(direction: Direction) {
	...

	if (map != newMap) {
		isAnimationRunning = true
		showAnimation(moves, merges) {
			map = newMap
			generateBlock()
			isAnimationRunning = false

			// new code here
			var points = 0
			merges.forEach {
				points += numberFor(it.first).value
			}
			score.update(score.value + points)
		}
	}
}
```

With Kotlin standard library, we can simplify this code a little:

```kotlin
fun Stage.moveBlocksTo(direction: Direction) {
	...

	if (map != newMap) {
		isAnimationRunning = true
		showAnimation(moves, merges) {
			map = newMap
			generateBlock()
			isAnimationRunning = false

			// new code here
			val points = merges.sumBy { numberFor(it.first).value }
			score.update(score.value + points)
		}
	}
}
```

Choose any option you like. :-)

## Updating score on restart

We also should update score when the user chooses to restart the game. Since we already have a special function for
restarting, it's quite simple to achieve:

```kotlin
fun Container.restart() {
	map = PositionMap()
	blocks.values.forEach { it.removeFromParent() }
	blocks.clear()
	// new code line here
	score.update(0)
	generateBlock()
}
```

The basic part with the scores is implemented. If you run the game now, you'll see that the scores are updated as
expected.

![](/assets/images/image-1%20(1).png)

But there is one flaw we need to fix – if you close the game, you'll never see your best score again.

Now, let's go to the next part of this article – data saving, where we'll fix this flaw.

## Greet NativeStorage!

To save some data and to restore it later, KorGE provides a special class – `NativeStorage`. Its implementation differs
on different targets (since it's _native storage_), but the goal of this class is the same on all of them – to allow you
to write some simple data (key-value pairs) during the game run and to read it after the game is closed and reopened.

You shouldn't create an instance of `NativeStorage` by yourself. Instead, you should use `Views.storage` extension
property (that's why you can't initialize it before `main` function). You can get an instance of `NativeStorage` wherever you have access to the `Stage`,
since `Stage` has `views` property. For example, you can get `NativeStorage` inside `main()` function. That's how we'll
get it in our project:

```kotlin
suspend fun main() = Korge(...) {
	font = resourcesVfs["clear_sans.fnt"].readBitmapFont()

	// new code line here
	val storage = views.storage

	//TODO: here we'll update the best score
	//best.update(...)

	score.observe {
		if (it > best.value) best.update(it)
	}
	best.observe {
		//TODO: here we'll update the value in the storage
	}
}
```

Well, now we have a storage where we'll save our best score and from where we'll restore it. Let's do it right away!

## Saving and restoring scores

First, let's restore the best score by getting saved value from the storage and updating the score:

```kotlin
suspend fun main() = Korge(...) {
	font = resourcesVfs["clear_sans.fnt"].readBitmapFont()

	val storage = views.storage
	// new code line here
	best.update(storage.getOrNull("best")?.toInt() ?: 0)

	score.observe {
		if (it > best.value) best.update(it)
	}
	best.observe {
		//TODO: here we'll update the value in the storage
	}
}
```

Second, let's save the best score in the storage by setting the current value with the same key ("best") that we used
above.

```kotlin
suspend fun main() = Korge(...) {
	font = resourcesVfs["clear_sans.fnt"].readBitmapFont()

	val storage = views.storage
	best.update(storage.getOrNull("best")?.toInt() ?: 0)

	score.observe {
		if (it > best.value) best.update(it)
	}
	best.observe {
		// new code line here
		storage["best"] = it.toString()
	}
}
```

`NativeStorage` allows you to save and restore only `String` values, that's why we had to call `toInt()` and `toString` in
the code above.

Well, that's it with the best score! If you run the game now, you'll see that the best score is restored after game
reopening.

Now, let's go to the last part of this chapter. Here we'll add a new class `History` that will let us save and restore
current game state and the whole history of all movements, so we'll be able to undo all the movements we added, even if
we added them during one of the previous launches of the game.

## History of all movements

So, let's create a new file `History.kt` and add our `History` class there:

```kotlin
class History {
}
```

How should we define a history of a 2048 game? We have 4х4 field with 16 blocks which update their positions on every
successful user move. It turns out that we should save the field state (block positions) on every move. Let's call this
state an `Element` of our history:

```kotlin
class History {

	class Element {
	}
}
```

This `Element` should know about positions of blocks at some point of the history. It also should know about the score
at that point. Since block positions are always 16, let's create an `IntArray` with 16 values in the `Element` class.
Those values will represent the ids of the blocks placed on each one of 16 positions. Another property in `Element` will
contain the score.

```kotlin
class History {

	class Element(val numberIds: IntArray, val score: Int)
}
```

Now, let's get back to the `History` class. How we should create an instance of `History`? When we launch the game, we
need to restore the game state. In order to do that, we can get a `String?` value from the `NativeStorage`. This value
is nullable, because there may be no value saved in the storage. After we get the value, we need to restore the
history. So let's pass this value to the `History` where we'll restore the state. We also need to add a special callback
that will be called when we update current `History`. The resulting `History` constructor should look like this:

```kotlin
class History(from: String?, private val onUpdate: (History) -> Unit) {
	...
}
```

From this `from` string property we should get our saved history. Since the history is conceptually just a list
of `Element` objects, let's define a property that will contain it.

```kotlin
class History(from: String?, private val onUpdate: (History) -> Unit) {

	class Element(val numberIds: IntArray, val score: Int)

	private val history = mutableListOf<Element>()
}
```

Now we need to initialize this `history` property with the values that `from` string should contain. But first we have
to think about the format that our history should be represented in. Since the history is just a list of `Element`s and
each `Element` is just a bunch of `Int` values (17 values, by the way), the simplest way to save them in a string would
be to convert each `Int` value to a string and separate those values with special characters. Let's use a comma `,` to
separate `Int` values in an `Element`, and a semicolon `;` to separate `Element` string representations. This format
will be easy to parse - we'll have to split the `from` string by `;` into several strings, and then split those strings
by `,` into 17 strings that can be converted to `Int`s. After that, we'll be able to create `Element` instances and fill
the `history` property. Here's the resulting code that does all that:

```kotlin
class History(from: String?, private val onUpdate: (History) -> Unit) {

	class Element(val numberIds: IntArray, val score: Int)

	private val history = mutableListOf<Element>()

	init {
		from?.split(';')?.fastForEach {
			val element = elementFromString(it)
			history.add(element)
		}
	}

	private fun elementFromString(string: String): Element {
		val numbers = string.split(',').map { it.toInt() }
		if (numbers.size != 17) throw IllegalArgumentException("Incorrect history")
		return Element(IntArray(16) { numbers[it] }, numbers[16])
	}
}
```

Now let's add the reverse action in the `History` – a special `toString()` function that will convert a `History`
instance to its string representation:

```kotlin
class History(...) {

	...

	override fun toString(): String {
		return history.joinToString(";") {
			it.numberIds.joinToString(",") + "," + it.score
		}
	}
}
```

And the last part of the `History` class implementation remains – we need to add a few functions that will let us check
and change the game history. Those functions will represent the following actions:

* add a new `Element` (number ids and score) to the `history`
* undo the last move with returning a new current history `Element`
* clear the whole `history` (when restarting a game)
* check if the history is empty

Let's also add a new property `currentElement` that will return the last element in the history. This property will come
in handy later.

So here is the new code:

```kotlin
class History(...) {

	...

	val currentElement: Element get() = history.last()

	...

	fun add(numberIds: IntArray, score: Int) {
		history.add(Element(numberIds, score))
		onUpdate(this)
	}

	fun undo(): Element {
		if (history.size > 1) {
			history.removeAt(history.size - 1)
			onUpdate(this)
		}
		return history.last()
	}

	fun clear() {
		history.clear()
		onUpdate(this)
	}

	fun isEmpty() = history.isEmpty()
}
```

Well, that's all with the `History` class. Now let's move to the `main.kt` file where we'll use the code we just wrote.

## Use of History in main code

First, we need to define a `History` variable. Then we need to initialize it in the `main()` function.

```kotlin
...

var map = PositionMap()
val blocks = mutableMapOf<Int, Block>()
//new line here
var history: History by Delegates.notNull()

...

suspend fun main() = Korge(...) {
	...

	val storage = views.storage
	//and a few new lines here
	history = History(storage.getOrNull("history")) {
		storage["history"] = it.toString()
	}
	...
}
```

There we create a new `History` instance with a string from the `storage` and a callback that will save a string
representation of that `History` in the `storage` on every `History` update.

In the `main()` function, there is a special container called `undoBlock`. When a user clicks on it, we need to undo the
last move he/she made by restoring the previous state of the field. We can get this state from the `history.undo()`
call.

```kotlin
suspend fun main() = Korge(...) {
	...

	val undoBlock = container {
		val background = ...
		...
		//new lines here
		onClick {
			this@Korge.restoreField(history.undo())
		}
	}

	generateBlock()
	...
}
```

We need to create a new function `Container.restoreField(History.Element)` in which the field will be cleared, its state
and the current score will be updated to the previous values, and blocks on the field will be replaced by the old ones.
The following code does exactly that.

```kotlin
fun Container.restoreField(history: History.Element) {
	map.forEach { if (it != -1) deleteBlock(it) }
	map = PositionMap()
	score.update(history.score)
	freeId = 0
	val numbers = history.numberIds.map {
		if (it >= 0 && it < Number.values().size)
			Number.values()[it]
		else null
	}
	numbers.forEachIndexed { i, number ->
		if (number != null) {
			val newId = createNewBlock(number, Position(i % 4, i / 4))
			map[i % 4, i / 4] = newId
		}
	}
}
```

In the `main()` function we also need to restore the field if its state was saved during previous game launch. If it
wasn't, we need to generate a new block. Right now we do only the last action. Here is the fix:

```kotlin
suspend fun main() = Korge(...) {
	...

	val undoBlock = ...

	//old code
	//generateBlock()

	//new code
	if (!history.isEmpty()) {
		restoreField(history.currentElement)
	} else {
		generateBlock()
	}

	onKeyDown {
		...
	}
	...
}
```

In the `Container.restart()` function we need to clear the history, so after closing and reopening the game the old game
state can't be restored (instead, the new game state should be restored).

```kotlin
fun Container.restart() {
	map = PositionMap()
	blocks.values.forEach { it.removeFromParent() }
	blocks.clear()
	score.update(0)
	//new code line here
	history.clear()
	generateBlock()
}
```

And the last change we need to make. In order to always have the latest state saved, we should update our history
whenever a new block is generated. Therefore the state will be saved when a user moves or merges blocks and when the
game is being opened or restarted. We already have `Container.generateBlock()` function. Let's add a new line there and
rename this function to `Container.generateBlockAndSave()`.

> A small tip: you can rename a function by positioning cursor on it and pressing `Shift + F6`

The updated function should look like this:

```kotlin
fun Container.generateBlockAndSave() {
	val position = map.getRandomFreePosition() ?: return
	val number = if (Random.nextDouble() < 0.9) Number.ZERO else Number.ONE
	val newId = createNewBlock(number, position)
	map[position.x, position.y] = newId
	history.add(map.toNumberIds(), score.value)
}
```

Here we use an undefined function `PositionMap.toNumberIds()`. Let's add it to the `PositionMap` class. It should create
and return an IntArray containing ids of numbers placed on the field.

```kotlin
class PositionMap(...) {

	...

	fun toNumberIds() = IntArray(16) { getNumber(it % 4, it / 4) }
}
```

## Conclusion

Well, this is probably the last step in the tutorial about creating your own 2048 game in KorGE. In these 5 steps, we
have developed a simple classic 2048 game with all the features I think this game should have. We have also considered
some of the many features the KorGE has. I hope, this journey was interesting for you and you have learnt something new
for yourself. If you haven't seen some of the steps, here is a full list of them:

* [KorGE Tutorial - Writing 2048 game. Step 0 - Introduction](https://blog.korge.org/korge-tutorial-writing-2048-game-step-0/)
* [KorGE Tutorial - Writing 2048 game. Step 1 - Views](https://blog.korge.org/korge-tutorial-writing-2048-game-step-1/)
* [KorGE Tutorial - Writing 2048 game. Step 2 - State and interaction](https://blog.korge.org/korge-tutorial-writing-2048-game-step-2-controls/)
* [KorGE Tutorial - Writing 2048 game. Step 3 - Animation](https://blog.korge.org/korge-tutorial-writing-2048-game-step-3-animation/)
* [KorGE Tutorial - Writing 2048 game. Step 4 - Data saving](https://blog.korge.org/korge-tutorial-writing-2048-game-step-4-data-saving/)

The code written in all the steps is available [here](https://gist.github.com/RezMike/539e442fa49d5329115a60597e3d438a).
You can find the original project [on GitHub](https://github.com/RezMike/2048).
And [here](https://rezmike.github.io/2048/) you can play the 2048 game online!

And, of course, a short gameplay video with our completed 2048 game:

<iframe src="https://www.youtube.com/embed/ruAJmzIaoWk?feature=oembed" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen="" name="fitvid0"></iframe>
