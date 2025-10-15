This applies to all types of lerp(), as well as all types of slerp().

Note that this is **very** similar to an implementation of a linear tween. And that isn't a coincidence. All the Tween node does, is abstract this away, and repeat the operation over a fixed time. Fancy easings are achieved by multiplying the delta/weight with some curve.

Also note that, when you want to lerp over X time, or at Y units per frame. You **should** be using a Tween, or move_towards() instead. Not lerp. It's the wrong tool for the job.

Other uses for lerp are to easily calculate percentages. Want the middle between A and B? a.lerp(b, 0.5). Want 50 beyond A to B? Make it 1.5.

```GDScript
var _start: Vector2 = Vector.ZERO
var _end: Vector2 = Vector.ZERO
var _speed_scale: float = 1.0
var _progress: float = 0.0
var _is_lerping: bool = true

func _process(delta: float) -> void:
	if _is_lerping:
		_progress = min(_progress + (delta * _speed_scale), 1)
		position = _start.lerp(_end, progress)
		if _progress >= 1:
			_is_lerping = false


func start_lerping(to: Vector2, speed_scale: float) -> void:
	_start = position
	_end = to
	_speed_scale = speed_scale
	_progress = 0.0
	_is_lerping = true
```

"But this doesn't work with physics?!" Neither was lerping the position with delta.

You **can** and should lerp the velocity towards a target value, to achieve smooth but unrealistic acceleration and deceleration. You will find this in most Mario games for example. Or FPS characters.
