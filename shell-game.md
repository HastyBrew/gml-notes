# Shell Game Notes

## ROOM
**Size:** 800x600 <br />
**Background Color:** dark blue

## SPRITES

*Note: origins MUST be middle center*

### spr_cup

**Start Size:** 200x100 <br />
**Final Size:** 200x200 <br />

Make a brown circle, color in, resize to 200x100 to cut circle in half<br />
Do little line drawings to sort of look like a coconut

### spr_ball

**Size:** 50x50 <br />

Make a red circle in the middle, shade it to make a ball shape

## OBJECTS

### obj_cup

#### Create

```gml
depth = 2
```

### obj_controller

Drop the blank object in the room

#### Create

```gml
randomize()

sw = 200
start_x = (room_width/2) - sw

global.start_y = room_height/2
global.lift_height = global.start_y-50
lifting = true

global.lift_all = false
global.can_press = false
global.resetting = false

cup_ids = []

shuffle_num = floor(random_range(7,12))

for(var i = 0;i<3;i++) {

	cup_ids[i] = instance_create_layer(start_x+(sw*i),global.start_y,"Instances",obj_cup)
	
}

ball_index = irandom(array_length(cup_ids)-1)
balled_cup = cup_ids[ball_index]
balled_cup.balled = true

ball = instance_create_layer(balled_cup.x,balled_cup.y+25,"Instances",obj_ball)

global.lift_speed = 5
lift_delay = 30
lift_delay_total = 30

move_speed = 10

cup1 = noone
cup2 = noone

cup1_pos = 0
cup2_pos = 0

enum GAME_STATE {

	SETTING,
	LIFTING,
	SET_SHUFFLE,
	SHUFFLING,
	CHOOSING,
	RESETTING
	
}

game_state = GAME_STATE.SETTING

function random_neighbor(obj_x,obj_y,obj,sw) {
	
	var temp_arr = []
	
	var hit = instance_position(obj_x-sw,obj_y,obj)
	if hit {
		array_push(temp_arr,hit)
	}
	hit = instance_position(obj_x+sw,obj_y,obj)
	if hit {
		array_push(temp_arr,hit)
	}

	if array_length(temp_arr) > 0 {
		var random_index = array_length(temp_arr)-1
		return temp_arr[random_index]
	}
	else {
		return false	
	}
	
}
```


#### Step

```gml
switch(game_state) {

	case GAME_STATE.SETTING:
	
		if mouse_check_button_pressed(mb_left) {
			
			game_state = GAME_STATE.LIFTING	
		}
	
	
		break;
	
	case GAME_STATE.LIFTING:
	
		if lifting {
			if balled_cup.y > global.lift_height {
				balled_cup.y -= global.lift_speed
			}
			else {
				lifting = false	
			}
		}
		else {
			if lift_delay > 0 {
				lift_delay--	
			}
			else {
				if balled_cup.y < global.start_y {
					balled_cup.y += global.lift_speed	
				}
				else {
					game_state = GAME_STATE.SET_SHUFFLE
				}
			}
		}
		
		break;
		
	case GAME_STATE.SET_SHUFFLE:
	
		lift_delay = lift_delay_total
	
		if instance_exists(ball) {
			instance_destroy(ball)	
		}
	
	
		var cup_index = irandom(array_length(cup_ids)-1)
		cup1 = cup_ids[cup_index]
		cup1_pos = cup1.x
		
		
		cup2 = random_neighbor(cup1.x,cup1.y,obj_cup,sw)
		cup2_pos = cup2.x
		
		if instance_exists(cup1) and instance_exists(cup2) and cup1_pos > 0 and cup2_pos > 0 {
			
			if shuffle_num > 0 {
				shuffle_num--
				game_state = GAME_STATE.SHUFFLING
			}
			else {
				global.can_press = true
				shuffle_num = floor(random_range(7,12))
				game_state = GAME_STATE.CHOOSING	
			}
		}
	
		break;
	
	
	case GAME_STATE.SHUFFLING:

		if cup1.x > cup2_pos {
			cup1.x -= move_speed	
		}
		if cup1.x < cup2_pos {
			cup1.x += move_speed	
		}
		
		
		if cup2.x > cup1_pos {
			cup2.x -= move_speed	
		}
		if cup2.x < cup1_pos {
			cup2.x += move_speed	
		}
		
		if cup1.x == cup2_pos and cup2.x == cup1_pos {
			cup1 = noone
			cup2 = noone
			game_state = GAME_STATE.SET_SHUFFLE	
		}
		

		break;
		
	case GAME_STATE.CHOOSING:
	
		if !instance_exists(ball) {
			ball = instance_create_layer(balled_cup.x,balled_cup.y+25,"Instances",obj_ball)	
		}
		
		if global.lift_all {
			if lift_delay > 0 {
				lift_delay--	
			}
			else {
				game_state = GAME_STATE.RESETTING	
			}
		}
	
		break;
		
	case GAME_STATE.RESETTING:
		
		global.lift_all = false
		global.resetting = true
	
		break;
	
}
```

### obj_cup

#### Create

```gml
depth = 1
image_speed = 0

balled = false
chosen = false

lifting = false
```
#### Step

```gml
if instance_exists(obj_controller) {
	if obj_controller.game_state == GAME_STATE.CHOOSING {
	
		if point_in_rectangle(mouse_x,mouse_y,bbox_left,bbox_top,bbox_right,bbox_bottom) {
			if global.can_press {
				if mouse_check_button_pressed(mb_left) {
					lifting = true
					global.can_press = false
				}
			}
			
		}
		
	}
}

if lifting or global.lift_all {

	if y > global.lift_height {
		y -= global.lift_speed
	}
	else {
		global.lift_all = true
		lifting = false
	}
	
}

if global.resetting {
	if y < global.start_y {
		y += global.lift_speed	
	}
	else {
		if instance_exists(obj_controller) {	
			obj_controller.game_state = GAME_STATE.SET_SHUFFLE
			global.resetting = false
		}
	}
}

```
