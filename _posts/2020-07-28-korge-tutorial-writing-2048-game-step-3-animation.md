---
layout: post
title: "KorGE Tutorial - Writing 2048 game. Step 3 - Animation"
author: rezmike
categories: [ Tutorials ]
image: assets/images/51935839105-1.png
---

In [the previous step](https://blog.korge.org/korge-tutorial-writing-2048-game-step-2-controls/) we have added
PositionMap class for the game state and initial code that handles interaction. In this step we are going to define the
main logic of the game - the movement and merging of blocks. We'll also add animation for that. So the resulting game
after this step will be quite playable and will look like this:

![](/assets/images/sample-1.gif)

## Preliminary checks

As you may remember, the last code lines we wrote in the previous step were `moveBlocksTo(direction)` function and
a `TODO` in it:

```kotlin
fun Stage.moveBlocksTo(direction: Direction) {
	println(direction)
	// TODO: we'll implement this function in the next step
}
```

Now, let's get back to it. First, remove lines with `println` and `TODO`. Second, add a few preliminary checks. Since
we're going to play animation of movement and merging, we need to check whether the animation is already running when a
gamer presses a key or swipes. After that we need to check if there are available moves or the game is over (in the
second case we'll show a special overlay offering to start the game over). So let's add two special
flags `isAnimationRunning` and `isGameOver` before the `main` function:

```kotlin
var isAnimationRunning = false
var isGameOver = false

suspend fun main() = ...
```

Next, let's add checks in `moveBlocksTo` function:

```kotlin
fun Stage.moveBlocksTo(direction: Direction) {
	if (isAnimationRunning) return
	if (!map.hasAvailableMoves()) {
		if (!isGameOver) {
			isGameOver = true
			showGameOver {
				isGameOver = false
				restart()
			}
		}
		return
	}
}
```

If the animation is running, we simply return from the function. Otherwise we check if there are available moves. Now
let's add two functions for that in `PositionMap` class:

```kotlin
class PositionMap /*...*/ {

	//...

	fun hasAvailableMoves(): Boolean {
		array.each { x, y, _ ->
			if (hasAdjacentEqualPosition(x, y)) return true
		}
		return false
	}

	private fun hasAdjacentEqualPosition(x: Int, y: Int) = getNumber(x, y).let {
		it == getNumber(x - 1, y) || it == getNumber(x + 1, y) || it == getNumber(x, y - 1) || it == getNumber(x, y + 1)
	}
}
```

Here we look at each position on the map and check if there is an adjacent position with the same number. If the
position is out of the field (like when `x == -1` or `y == 4`) or there is no number there (the cell is empty), then
function `getNumber` will return **-1**, therefore covering the case when there is a single empty cell on the field with
no empty cells next to it (no cells with the same number).

Well, the check of available cells is implemented. Let's get back to `moveBlockTo` function and see what we should do
next.

## Game over overlay

Now we need to add a special function `showGameOver` that will show a game over overlay and execute a special callback (
provided as a parameter `onRestart`) after a gamer clicks "Try again". This function should be called on a `Container` (
and it is called on our `Stage`), thus we can add other views to our stage:

```kotlin
fun Container.showGameOver(onRestart: () -> Unit) = container {
   ...
}
```

In the first line, there is a `container {…}` call. This way we add an overlay container to the `Stage`, this container
will be removed from the stage container when the gamer clicks "Try again". All other views added inside
the `showGameOver` function will actually be added to this overlay container, so their coordinates should be determined
relative to the container bounds.

Let's add content inside the overlay container. We'll use a `text` view for title and a `uiText` view for clickable
text "Try again". `UIText` is a special view in KorGE that lets you define different text formats (color, size and font)
for `normal`, `over` and `down` states. We also need to define an `onClick {…}` listener on the text "Try again" that
will remove the overlay container from the stage and call `onRestart` callback. The same action should be performed when
a gamer presses **Enter** or **Space** keys. The resulting function looks like that:

```kotlin
fun Container.showGameOver(onRestart: () -> Unit) = container {
	val format = TextFormat(
			color = RGBA(0, 0, 0),
			size = 40,
			font = Html.FontFace.Bitmap(font)
	)
	val skin = TextSkin(
			normal = format,
			over = format.copy(color = RGBA(90, 90, 90)),
			down = format.copy(color = RGBA(120, 120, 120))
	)

	fun restart() {
		this@container.removeFromParent()
		onRestart()
	}

	position(leftIndent, topIndent)

	roundRect(fieldSize, fieldSize, 5.0, color = Colors["#FFFFFF33"])
	text("Game Over", 60.0, Colors.BLACK, font) {
		centerBetween(0.0, 0.0, fieldSize, fieldSize)
		y -= 60
	}
	uiText("Try again", 120.0, 35.0, skin) {
		centerBetween(0.0, 0.0, fieldSize, fieldSize)
		y += 20
		onClick { restart() }
	}

	onKeyDown {
		when (it.key) {
			Key.ENTER, Key.SPACE -> restart()
			else -> Unit
		}
	}
}
```

You can comment the **if** condition that checks available moves in `moveBlocksTo` function and see how the game over
overlay looks like (it'll appear after you press any arrow key or swipe):

![](/assets/images/image-1.png)

## Game restart

After implementing game over overlay, we need to add logic that should be executed when the user clicks on the "Try
again" text. We already have `restart()` call in the `moveBlocksTo` function, let's create this function and add code in
it. We need to clear the field and add a new block there:

```kotlin
fun Container.restart() {
	map = PositionMap()
	blocks.values.forEach { it.removeFromParent() }
	blocks.clear()
	generateBlock()
}
```

Here we create a new `PositionMap` object and assign it to the `map` property, remove all `Block` views added to
the `blocks` mutable map as values, from the parent container, then we clear the `blocks` map (i.e. remove all map
entries and make the map empty) and generate a new Block on the currently empty field.

Now you can comment the **if** condition in `moveBlocksTo` function again and see that game restart works correctly.
After pressing "Try again" you will see a new block being added to the empty field:

![](/assets/images/sample.gif)

There is also a small addition to the `restartBlock` in the `main()` function — we need to restart the game after the
restart "button" is called:

```kotlin
 suspend fun main()... {
	...
	val restartBlock = container {
		...
		onClick {
			this@Korge.restart()
		}
	}
	...
}
```

## Changes of the position map

Let's return to our `moveBlocksTo` function. After checking that animation is not running and the map has available
moves, we need to change the map based on a user reaction. First we need to calculate a new map and all moves and merges
of the blocks on the field. Then, if the old map and the new map are different, we need to animate the changes:

```kotlin
fun Stage.moveBlocksTo(direction: Direction) {
	if (isAnimationRunning) return
	if (!map.hasAvailableMoves()) {
		//...
	}

	val moves = mutableListOf<Pair<Int, Position>>()
	val merges = mutableListOf<Triple<Int, Int, Position>>()

	val newMap = calculateNewMap(map.copy(), direction, moves, merges)

	if (map != newMap) {
		isAnimationRunning = true
		showAnimation(moves, merges) {
			// when animation ends
			map = newMap
			generateBlock()
			isAnimationRunning = false
		}
	}
}
```

Here we create two mutable lists: one for moves (pairs of an id of the moving block and a new position) and the other
for merges (triples of two ids of the merging blocks and a new position). Then we calculate a new map - we pass a copy
of the current map that will be changed during calculation, a direction of the movement and our two lists.

After the calculation we check that two maps are different (using `equals` operator defined in `PositionMap` class). If
so, we set `isAnimationRunning` to `true` and show an animation of moves and merges. When the animation ends, we assign
the new map to the current one, generate a new block and set `isAnimaitonRunning` flag to `false`.

Let's add a new function `copy()` to `PositionMap` that should create and return a new map with the same content:

```kotlin
class PositionMap(...) {
	...

	fun copy() = PositionMap(array.copy(data = array.data.copyOf()))
}
```

## Calculation of a new map

Now we need to add a function in which moves, merges and a new map should be calculated. Let's do it like that:

```kotlin
fun calculateNewMap(
		map: PositionMap,
		direction: Direction,
		moves: MutableList<Pair<Int, Position>>,
		merges: MutableList<Triple<Int, Int, Position>>
): PositionMap {
	val newMap = PositionMap()
	val startIndex = when (direction) {
		Direction.LEFT, Direction.TOP -> 0
		Direction.RIGHT, Direction.BOTTOM -> 3
	}
	var columnRow = startIndex

	fun newPosition(line: Int) = when (direction) {
		Direction.LEFT -> Position(columnRow++, line)
		Direction.RIGHT -> Position(columnRow--, line)
		Direction.TOP -> Position(line, columnRow++)
		Direction.BOTTOM -> Position(line, columnRow--)
	}

	for (line in 0..3) {
		var curPos = map.getNotEmptyPositionFrom(direction, line)
		columnRow = startIndex
		while (curPos != null) {
			val newPos = newPosition(line)
			val curId = map[curPos.x, curPos.y]
			map[curPos.x, curPos.y] = -1

			val nextPos = map.getNotEmptyPositionFrom(direction, line)
			val nextId = nextPos?.let { map[it.x, it.y] }
			//two blocks are equal
			if (nextId != null && numberFor(curId) == numberFor(nextId)) {
				//merge these blocks
				map[nextPos.x, nextPos.y] = -1
				newMap[newPos.x, newPos.y] = curId
				merges += Triple(curId, nextId, newPos)
			} else {
				//add old block
				newMap[newPos.x, newPos.y] = curId
				moves += Pair(curId, newPos)
			}
			curPos = map.getNotEmptyPositionFrom(direction, line)
		}
	}
	return newMap
}
```

First, we create a new `PositionMap` instance, define a start index based on the direction (it's an number of a column
or a row with which we start calculating) and define a current index of a column or a row (`columnRow` variable).

Second, we define an inner function `newPosition` that takes a `line` (row/column) number and returns the next position
based of the movement direction.

And third, we go through all the lines (from 0 to 3), get current position that has a block and get current block id,
get next position that has a block and get next id, then check whether the current number and the next number are the
same or not. If yes - we merge them: we clear the next position on the map (the current one will no longer be viewed
after the current check), define the current id in the new position on the new map and add a triple to the `merges`
list. If no - we move the current number: we define the current id in the new position on the new map and add a pair to
the `moves` list. Then we update the position of the current block and repeat the previous steps.

After going through all the positions is done, we return a new map.

Here we need to add a new function `getNotEmptyPositionFrom(direction, line)` in the `PositionMap` class that returns
a `Position` instance if the empty position is found, or `null` otherwise:

```kotlin
class PositionMap(...) {
	...

	fun getNotEmptyPositionFrom(direction: Direction, line: Int): Position? {
		when (direction) {
			Direction.LEFT -> for (i in 0..3) getOrNull(i, line)?.let { return it }
			Direction.RIGHT -> for (i in 3 downTo 0) getOrNull(i, line)?.let { return it }
			Direction.TOP -> for (i in 0..3) getOrNull(line, i)?.let { return it }
			Direction.BOTTOM -> for (i in 3 downTo 0) getOrNull(line, i)?.let { return it }
		}
		return null
	}
}
```

We also need to add an utility function `numberFor(blockId)` in the `main.kt` file:

```kotlin
...
var map = ...
var blocks = ...

fun numberFor(blockId: Int) = blocks[blockId]!!.number
```

## Animation of moves and merges

Now, the last feature we will add in this article and also the main feature of it – the animation! KorGE allows you to
use different types of animation. We will use [tween animations](https://korlibs.soywiz.com/korge/animation/#tweens)
with the [Animator DSL](https://korlibs.soywiz.com/korge/animation/#animator) because it will let us define different
animation elements (tweens) in a special order, so that some elements will be executed in parallel, some – in sequence,
and we will be able to define blocks of plain code (without tweening animation) that should be executed between them.

The animation is suspending, so we should use `launchImmediately` function and a `Stage` instance to create a coroutine
in which the animation will be running. The resulting `showAnimation` function is shown below:

```kotlin
fun Stage.showAnimation(
		moves: List<Pair<Int, Position>>,
		merges: List<Triple<Int, Int, Position>>,
		onEnd: () -> Unit
) = launchImmediately {
	animateSequence {
		parallel {
			moves.forEach { (id, pos) ->
				blocks[id]!!.moveTo(columnX(pos.x), rowY(pos.y), 0.15.seconds, Easing.LINEAR)
			}
			merges.forEach { (id1, id2, pos) ->
				sequence {
					parallel {
						blocks[id1]!!.moveTo(columnX(pos.x), rowY(pos.y), 0.15.seconds, Easing.LINEAR)
						blocks[id2]!!.moveTo(columnX(pos.x), rowY(pos.y), 0.15.seconds, Easing.LINEAR)
					}
					block {
						val nextNumber = numberFor(id1).next()
						deleteBlock(id1)
						deleteBlock(id2)
						createNewBlockWithId(id1, nextNumber, pos)
					}
					sequenceLazy {
						animateScale(blocks[id1]!!)
					}
				}
			}
		}
		block {
			onEnd()
		}
	}
}
```

Here we use `animateSequence` function to create an `Animator` instance. Inside it we define a `parallel` animation and
a block with `onEnd()` callback. Inside `parallel` block we go through the `moves` list and define a movement animation
via `moveTo` extension function in Animator, we also go through the `merges` list and define a `sequence` animation for
each merge triple. Inside `sequence` block we execute `parallel` movement animation of both blocks, then we exucute code
in `block` element where we delete both blocks and create a new one with the id of the first block, and then we use
a `sequenceLazy` animation element to animate scale of a block.

> **Note:** the code inside `sequence` is executed right away when the `showAnimation` function is called, the code inside `sequenceLazy` is executed only when the animation inside this animation element should be shown. That's why we use `sequenceLazy` to animate scale of a block – the block that we animate there is created only in the `block` element, it's not available when the whole animation starts. We don't use `sequenceLazy` everywhere because this animation function is quite expensive in comparison with `sequence`.

Well, let's add the missing `deleteBlock` function in the `main.kt` file. In this function we remove a block view with
the corresponding id from the `blocks` map and then remove this view from its parent container:

```kotlin
fun numberFor(blockId: Int) = ...
fun deleteBlock(blockId: Int) = blocks.remove(blockId)!!.removeFromParent()
```

And the last function we need to add is `animateScale(block)`.

```kotlin
fun Animator.animateScale(block: Block) {
	val x = block.x
	val y = block.y
	val scale = block.scale
	tween(
			block::x[x - 4],
			block::y[y - 4],
			block::scale[scale + 0.1],
			time = 0.1.seconds,
			easing = Easing.LINEAR
	)
	tween(
			block::x[x],
			block::y[y],
			block::scale[scale],
			time = 0.1.seconds,
			easing = Easing.LINEAR
	)
}
```

It uses `Animator` as a receiver and calls its `tween` function twice to animate position and scale at the same time.
These tweening animations will be executed sequentially because the `animateScale` function is called
inside `sequenceLazy` animation block.

In order to make this code compile, we need to add import for tweening properties either by
adding `import com.soywiz.korge.tween.*` at the beginning of the file, or manually by setting the mouse pointer on the
tweening parameters, pressing **Alt+Enter** and choosing **Import** option.

## The result

Well, we've added a lot of code in this step. If you run the game now, you will see that it's already quite playable:

![](/assets/images/sample-2.gif)

The whole project code after this step is shown [here](https://gist.github.com/RezMike/3979e24f0e90796767552cc89792ee20)
. In the next step we'll add scores and saving state via NativeStorage. Stay tuned!
