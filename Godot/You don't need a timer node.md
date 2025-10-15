While Timer can be a useful node at times. Often it is nicer to type one out yourself.

You get control and clarity, don't need an additional node, and can keep your code self contained.

```GDScript
const WAIT_TIME: float = 1.0
var _t: float = 0.0


func _process(delta) -> void:
	_t += delta
	if _t >= WAIT_TIME:
		_t -= WAIT_TIME
		do_something()


func do_something() -> void:
	print("Timer tick.")
```

# Fun Timer Trivia
Keep in mind that there is a common pitfall in timer implementation. Both the Time node and the above code have the same problem: They count delta.

Delta is not a linear value, but represents time **steps**. The average delta at 60fps is 16.6666~ MS. This means that the **smallest** time step you can wait for, should be, 16MS. And that is almost correct. A wait time of 1.0 / 60.0 will result in your time executing, almost, every frame. (It might in fact run every frame.)

But why wouldn't it? **Lets propose you want to run code every 100 MS.**

| Tick | Delta  | Callback        |
| ---- | ------ | --------------- |
| 0    | 0.0    | True            |
| 1    | 16.66  |                 |
| 2    | 33.32  |                 |
| 3    | 49.98  |                 |
| 4    | 66.64  |                 |
| 5    | 83.3   |                 |
| 6    | 99.96  | MISSED by 0.4ms |
| 7    | 116.62 | True            |
| 8    | 133.28 |                 |
| 9    | 149.94 |                 |
| 10   | 166.6  |                 |
| 11   | 183.26 |                 |

Instead of running "during the 6ths tick" your timer "misses", and runs a whole frame later!

**How do we fix this?**
By pretending that, each tick is actually the time of the next frame, and simulating additional sub-ticks.

```GDScript
const MAX_TICKS: int = 300 # Process Ticks
const WAIT_TIME: float = 0.100 # In seconds
var tick: int = 0 # Elapsed Process Ticks
var time: float = 0.0 # Accumulated Delta

var result: Array[int] = [] # Array of ticks on which a func was called

func _process(delta: float) -> void:
	tick += 1
	time += delta
	
	if time >= WAIT_TIME:
		time -= WAIT_TIME
		count()
	if time + (delta * 0.5) >= WAIT_TIME:
		time -= WAIT_TIME + (delta * 0.5)
		count()
	
	if tick >= 300:
		set_process(false)
		print("Method 3:")
		print(result)


func count() -> void:
	result.append(tick)
```

**This is not, the right way to do it.** But you should be able to see what's going on here. We pretend if a little extra time has passed. Equal to half a tick.

| Tick | Delta  | Callback                 |
| ---- | ------ | ------------------------ |
| 0    | 0.0    | True                     |
| 1    | 16.66  |                          |
| 2    | 33.32  |                          |
| 3    | 49.98  |                          |
| 4    | 66.64  |                          |
| 5    | 83.3   |                          |
| 6    | 99.96  | True (in the half frame) |
| 7    | 116.62 | Previously True          |
| 8    | 133.28 |                          |
| 9    | 149.94 |                          |
| 10   | 166.6  |                          |
| 11   | 183.26 |                          |

Huraa, we're now calling our function on the "correct" tick. And are **preventing drift.**

| Tick | Delta  | Callback                 |
| ---- | ------ | ------------------------ |
| 0    | 0.0    | True                     |
| 1    | 16.66  |                          |
| 2    | 33.32  |                          |
| 3    | 49.98  |                          |
| 4    | 66.64  |                          |
| 5    | 83.3   |                          |
| 6    | 99.96  | True (in the half frame) |
| 7    | 116.62 | Previously True          |
| 8    | 133.28 |                          |
| 9    | 149.94 |                          |
| 10   | 166.6  |                          |
| 11   | 183.26 |                          |
| 12   | 199.92 | True (in the half frame) |
| 13   | 216.58 | Previously True          |
| 14   | 233.24 |                          |

If it's not evident: Drift refers to the fact that the timing offset from missing ticks "adds" up. Eventually leading to the **number of ticks between calls** changing, as the error wraps around to one whole tick worth of delta.

At 100ms this is almost irrelevant. But there are scenarios in games in which this comes up. Propose you have a character that attacks 2.5 times a second. And a game that runs at a tick rate of 30. That means you need to sync 2.5 times, with time steps of 33.33. Suffice to say, Neither 1, nor 2.5, cleanly divides by 0.033.

This btw is why in the video game Arknights, the character Thorns, will attack less times in total at higher game speeds. Since the game simply increases the delta for each tick, rather than simulating additional ticks. **Playing at higher speeds results in more ticks being missed.** And thus auto deploy can fail, due to the significant loss in DPS.

Changelog:
1. Updated the code example with tested code.