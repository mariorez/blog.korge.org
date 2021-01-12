---
layout: post
title: "KorGE Tutorial - Writing 2048 game. Step 2 - State and interaction"
author: rezmike
categories: [ Tutorials ]
image: assets/images/image.png
---

In [the previous step](https://blog.korge.org/korge-tutorial-writing-2048-game-step-1/) we have added all static
elements (views) we need in our games. Here we will implement dynamic blocks, add PositionMap for the game state, and
define controls that will let users move the blocks. At the end of this step we will have the following result:

![](/assets/images/image.png)

## Numbers

Let's start by defining numbers that the user can see in the game. At the moment, we only have the **main.kt** file in
our project. Let's create a new file, call it **Number.kt** and add enum class **Number** there. This enum will have two
properties: **value (Int)** and **color (RGBA)**. The **value** will define a number that we will show in blocks, the **
color** - a color that blocks with this number will be painted:

```kotlin
enum class Number(val value: Int, val color: RGBA) {
}
```

Since the field may contain a maximum of 16 blocks with different numbers (two of them may be two "4") and a user will
be able to merge them into one block with the next number, we define 17 values in our enum:

```kotlin
enum class Number(val value: Int, val color: RGBA) {
	ZERO(2, RGBA(240, 228, 218)),
	ONE(4, RGBA(236, 224, 201)),
	TWO(8, RGBA(255, 178, 120)),
	THREE(16, RGBA(254, 150, 92)),
	FOUR(32, RGBA(247, 123, 97)),
	FIVE(64, RGBA(235, 88, 55)),
	SIX(128, RGBA(236, 220, 146)),
	SEVEN(256, RGBA(240, 212, 121)),
	EIGHT(512, RGBA(244, 206, 96)),
	NINE(1024, RGBA(248, 200, 71)),
	TEN(2048, RGBA(256, 194, 46)),
	ELEVEN(4096, RGBA(104, 130, 249)),
	TWELVE(8192, RGBA(51, 85, 247)),
	THIRTEEN(16384, RGBA(10, 47, 222)),
	FOURTEEN(32768, RGBA(9, 43, 202)),
	FIFTEEN(65536, RGBA(181, 37, 188)),
	SIXTEEN(131072, RGBA(166, 34, 172));
}
```

## Blocks

Now let's add a special view called **Block** that will move across our field when a user interacts with it. To do it,
create a new file **Block.kt** and add a new class extending **Container** with one property **number** of type **
Number** (the enum created in the previous section):

```kotlin
class Block(val number: Number) : Container() {
}
```

In this class inside `init {}` we define the views of which this block will consist of: a **roundRect** as background
and a **text** with the number value:

```kotlin
class Block(val number: Number) : Container() {
	init {
		roundRect(cellSize, cellSize, 5.0, color = number.color)
		val textColor = when (number) {
			ZERO, ONE -> Colors.BLACK
			else -> Colors.WHITE
		}
		text(number.value.toString(), textSizeFor(number), textColor, font) {
			centerBetween(0.0, 0.0, cellSize, cellSize)
		}
	}
}
```

Here we use undefined values: **cellSize** and **font**. That's actually the variables we created in our **main**
function in **Step 1**. To make all needed variables available outside the **main** function, let's update the **
main.kt** file a little bit. Since we will need these variables as well as **fieldSize**, **leftIndent** and **
topIndent** later, outside the main function, we simply move them out and make them top-level. But as top-level, they
should be initialized with default values (we can't make them **lateinit**). For **Double** variables it's easy –
write **0.0** as default, but what about BitmapFont? Should we make it nullable even if we initialize it in the first
line of the **main** function? Well, no, because in Kotlin there is a special delegate for that – **
Delegates.notNull()**. So the resulting main function should look like this:

```kotlin
var cellSize: Double = 0.0
var fieldSize: Double = 0.0
var leftIndent: Double = 0.0
var topIndent: Double = 0.0
var font: BitmapFont by Delegates.notNull()

suspend fun main() = Korge(...) {
	font = resourcesVfs["clear_sans.fnt"].readBitmapFont()

	cellSize = views.virtualWidth / 5.0
	fieldSize = 50 + 4 * cellSize
	leftIndent = (views.virtualWidth - fieldSize) / 2
	topIndent = 150.0

	val bgField = ...
}
```

In the code of the class **Block**, there's also undefined function **textSizeFor**. It should return the size of the
text for the specified number. Let's add this function in **Block.kt** file:

```kotlin
private fun textSizeFor(number: Number) = when (number) {
	ZERO, ONE, TWO, THREE, FOUR, FIVE -> cellSize / 2
	SIX, SEVEN, EIGHT -> cellSize * 4 / 9
	NINE, TEN, ELEVEN, TWELVE -> cellSize * 2 / 5
	THIRTEEN, FOURTEEN, FIFTEEN -> cellSize * 7 / 20
	SIXTEEN -> cellSize * 3 / 10
}
```

Now let's define a special extension function that creates a **Block** view instance and add it to the container:

```kotlin
fun Container.block(number: Number) = Block(number).addTo(this)
```

## Creating new blocks

To manage all blocks on the game field, we need to specify two new variables in **main.kt**: **blocks** hash map
containing blocks with their ids, and **freeId** indicating the next available id for a new block:

```kotlin
val blocks = mutableMapOf<Int, Block>()

var freeId = 0
```

We also need to add several functions outside the **main** function. The first two are `columnX(number)`
and `rowY(number)` functions that take **number** of type **Int** as a column/row number and return real **x**/**y**
position for it:

```kotlin
fun columnX(number: Int) = leftIndent + 10 + (cellSize + 10) * number
fun rowY(number: Int) = topIndent + 10 + (cellSize + 10) * number
```

The next function is `createNewBlockWithId(id, number, position)`. It takes **id**, **number** and **position** for a
new block, creates this block and adds it to the **blocks** map:

```kotlin
fun Container.createNewBlockWithId(id: Int, number: Number, position: Position) {
	blocks[id] = block(number).position(columnX(position.x), rowY(position.y))
}
```

_Notice that we use `block()` function here, so a new block will be added to the receiver **Container** automatically._

_You can also notice that **Position** is not defined, we'll fix that a bit later._

And the last function we need to add now is `createNewBlock(number, position)` that takes **number** and **position**,
calculates new **id**, calls `createNewBlock(id, number, position)` function and returns this new **id**:

```kotlin
fun Container.createNewBlock(number: Number, position: Position): Int {
	val id = freeId++
	createNewBlockWithId(id, number, position)
	return id
}
```

## PositionMap

Now let's create a new file called **PositionMap.kt** and add enum class **Position** and class **PositionMap**
there. **Position** will have **x** and **y** properties indicating a position in **PositionMap**. **PositionMap** will
contain the current state of our game field – in the constructor, there will be a special two-dimensional **array**
from **kds** library containing block ids for each of 16 cells of the 4x4 game field (or **-1** if there is no block in
the current cell):

```kotlin
class Position(val x: Int, val y: Int)

class PositionMap(private val array: IntArray2 = IntArray2(4, 4, -1)) {
}
```

Let's add a few simple functions in our **PositionMap** that will be useful later. We will add more complex functions as
we write the game. So the functions are:

* `getOrNull(x, y)` – returns a **Position** object if there is a block in this position, otherwise returns null
* `getNumber(x, y)` – returns an **Int** value indicating the ordinal number of the **Number** enum element for the
  block in this position, or **-1** if there is no block
* `get(x, y)` – an operator function that returns a block id for this position
* `set(x, y, value)` – an operator function that sets a block id for this position
* `forEach(action)` – a function that calls this **action** for each block id in **array**
* `equals(other)` – checks whether the **other** object is **PositionMap** and whether positions of **this** and **
  that** map are equal.
* `hashCode()` – delegates calculating hashCode to **array**

And here's the implementation of these functions:

```kotlin
class PositionMap(private val array: IntArray2 = IntArray2(4, 4, -1)) {

	private fun getOrNull(x: Int, y: Int) = if (array.get(x, y) != -1) Position(x, y) else null

	private fun getNumber(x: Int, y: Int) = array.tryGet(x, y)?.let { blocks[it]?.number?.ordinal ?: -1 } ?: -1

	operator fun get(x: Int, y: Int) = array[x, y]

	operator fun set(x: Int, y: Int, value: Int) {
		array[x, y] = value
	}

	fun forEach(action: (Int) -> Unit) { array.forEach(action) }

	override fun equals(other: Any?): Boolean {
		return (other is PositionMap) && this.array.data.contentEquals(other.array.data)
	}
	override fun hashCode() = array.hashCode()
}
```

Now we just need to create an instance of **PositionMap** in **main.kt**, and we can move on to the next section:

```kotlin
var map = PositionMap()
val blocks = ...
```

## Generating new blocks

After the start of the game and after block movement, we need to generate a new block on the game field. Let's create a
special function **generateBlock** with **Container** receiver that will do that. It should get a random free position
from our **PositionMap** (if such a position exists), select a number for the new block (in 90% of cases - **"2"** or **
Number.ZERO**, in 10% - **"4"** or **Number.ONE**), create this block and add it to **map**:

```kotlin
fun Container.generateBlock() {
	val position = map.getRandomFreePosition() ?: return
	val number = if (Random.nextDouble() < 0.9) Number.ZERO else Number.ONE
	val newId = createNewBlock(number, position)
	map[position.x, position.y] = newId
}
```

Now we need to write **getRandomFreePosition** function in **PositionMap**. In this function, we count the quantity of
free positions (if this quantity is 0, then there are no free positions and we return null), choose a number for a
random free position and find it in the map:

```kotlin
class PositionMap(...) {
	...

	fun getRandomFreePosition(): Position? {
		val quantity = array.count { it == -1 }
		if (quantity == 0) return null
		val chosen = Random.nextInt(quantity)
		var current = -1
		array.each { x, y, value ->
			if (value == -1) {
				current++
				if (current == chosen) {
					return Position(x, y)
				}
			}
		}
		return null
	}

	...
}
```

Let's return to the **main** function, and in the end of it we call `generateBlock()` to generate a new block when the
game start:

```kotlin
fun main() = Korge(...) {
	...

	generateBlock()
}
```

Now you can run the game and see that everything works fine. When the game starts, a new block is generated at some
position on the field. If you rerun the game, the position may be different because it's selected randomly. So the
result may look like this:

![](/assets/images/image%20(1).png)

## User interaction

In KorGE, user interaction is based on the concept of events. For example, if a user presses a key or a mouse button,
the engine generates a special event for that, so you can listen to it and perform some actions. There are various types
of events:

* onKeyDown, onKeyUp, onKeyTyped
* onOver, onOut, onDown, onUp
* onMove, onClick, onMouseDrag, onSwipe
* and many more...

And there are also different ways how you listen to events. Here we'll use just one of them (via DSL function), if you
want to see more – feel free to check [this page](https://korlibs.soywiz.com/korge/input/) of the KorGE documentation.

But before we define event listeners, let's create a special enum class **Direction** (I decided to put it in **
PositionMap.kt** file, you can choose your own place). This enum will have 4 values: **LEFT**, **RIGHT**, **TOP** and **
BOTTOM**. They define the directions in which a user can move blocks.

```kotlin
enum class Direction {
	LEFT, RIGHT, TOP, BOTTOM
}
```

Well, now we can define **onKeyDown**. Just add these lines in the end of the main function:

```kotlin
fun main() = Korge(...) {
	...

	onKeyDown {
		when (it.key) {
			Key.LEFT -> moveBlocksTo(Direction.LEFT)
			Key.RIGHT -> moveBlocksTo(Direction.RIGHT)
			Key.UP -> moveBlocksTo(Direction.TOP)
			Key.DOWN -> moveBlocksTo(Direction.BOTTOM)
			else -> Unit
		}
	}
}
```

Inside **onKeyDown** block, we get a **KeyEvent** as **it**. **KeyEvent** has special properties like **type** (in this
case - **Key.Type.DOWN**), **id**, **key**, **keyCode** and **character**. So inside **when**, we check the **key** a
user pressed, and if it's one of the arrow keys, we call `moveBlockTo(direction)` function with the appropriate **
Direction** value (we'll define this function a bit later).

Let's also define another event listener – **onSwipe**. This one is more complex. It will listen to mouse events and
check if a user swipes via mouse. A swipe is a gesture when the mouse is pressed, moved a few pixels at some direction
and then released. The **onSwipe** listener lets us define a **SwipeDirection** – one of 4 possible movement directions,
and a **threshold** – the quantity of pixels a mouse should be moved to generate this event (once the event is
generated, it won't be generated again until a user releases the mouse). So let's add an **onSwipe** listener the **
threshold** of **20.0** in the main function right after **onKeyDown {}** and define the same check as in the previous
listener.

```kotlin
fun main() = Korge(...) {
	...

	onKeyDown {
		when (it.key) {
			Key.LEFT -> moveBlocksTo(Direction.LEFT)
			Key.RIGHT -> moveBlocksTo(Direction.RIGHT)
			Key.UP -> moveBlocksTo(Direction.TOP)
			Key.DOWN -> moveBlocksTo(Direction.BOTTOM)
			else -> Unit
		}
	}

	onSwipe(20.0) {
		when (it.direction) {
			SwipeDirection.LEFT -> moveBlocksTo(Direction.LEFT)
			SwipeDirection.RIGHT -> moveBlocksTo(Direction.RIGHT)
			SwipeDirection.TOP -> moveBlocksTo(Direction.TOP)
			SwipeDirection.BOTTOM -> moveBlocksTo(Direction.BOTTOM)
		}
	}
}
```

Now let's add the missing function `moveBlocksTo(direction)`. Since this part of the tutorial is already quite large, we
will not define the full body of this function – we'll do it in the next part. For now, let's limit ourselves to logging
the direction of the future movement. That way you can test the events and see if everything works fine.

```kotlin
fun Stage.moveBlocksTo(direction: Direction) {
	println(direction)
	// TODO: we'll implement this function in the next step
}
```

If you run the game, swipe in different directions and type different arrow keys, you will see something like that in
the output:

```kotlin
>Task: runJvm
// some KorGE logs
...
// your logs:
TOP
LEFT
BOTTOM
RIGHT
TOP
LEFT
BOTTOM
RIGHT
LEFT
```

The whole code written in this step is shown [here](https://gist.github.com/RezMike/6adce5df5f2a3108eb648ac59cdda924).
In [the next step](https://blog.korge.org/korge-tutorial-writing-2048-game-step-3-animation/) we'll define logic for
block movement and add animations for it. Stay tuned!
