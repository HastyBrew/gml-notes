# GAMEMAKER SHUFFLE LOAD ICON

## **GENERAL NOTES**

I made this for a slide puzzle game and just reused the logic for a simple loading icon. It's meant to give the player feedback when room logic is running in the background. 

It can be added as an instance to the room or through a controller object. You'll need to create a sprite as well as the object, more on those specifics below.

## **ROOM**

Any size


## **SPRITE**

Can be any size, full sprite size must be square, origin must be center. 

If the sprite is a square but the image isn't, make sure to select Mode: Full Image, Type: Rectangle in Collison Mask

**Name the sprite:** spr_loadicon


## **OBJECT**

### **obj_loadicon**

#### **Create**

```gml
depth = 0

sprite_index = spr_loadicon
image_index = 0
image_speed = 0

image_alpha = 0

// BOARD VARS

set_board = false

tile_size = sprite_width

initial_x = x
initial_y = y

prev_x = x
prev_y = y

new_x = x
new_y = y

rows = 0
columns = 0

offset_x = initial_x - tile_size
offset_y = initial_y - tile_size

board_arr = [noone]

// SLIDE VARS

slide_done = false
neighbor = noone

speed_change = 10 //change this to change speed

move_speed = tile_size / speed_change

// ENUM SETTING

enum LOADICON_STATE	{
	
	NULL,
	MAKING_BOARD,
	SET_SHUFFLE,
	SLIDING,
	FINISHED
	
}

loadicon_state = LOADICON_STATE.NULL

// BOARD MAKING

if instance_number(obj_loadicon) < 2 {
	set_board = true
	loadicon_state = LOADICON_STATE.MAKING_BOARD
}

if set_board {

	for (var i = 0; i < 9; i ++) {
	
		if i == 0 {
			new_x = offset_x
			new_y = offset_y
			rows++
		}
		else {
			board_arr[i] = instance_create_layer(offset_x+(tile_size*rows),offset_y+(tile_size*columns),"Instances",obj_loadicon)
			board_arr[i].image_index = 0 // change depending on your index
			board_arr[i].image_alpha = 1
			
			rows++
			
			if rows > 2 {
				columns++
				rows = 0
			}
			
		}
	
	}
	
}

function random_neighbor(obj,px,py) {
	
    var temp_arr = []

	// LEFT
    var n = instance_position(x - sprite_width, y, obj)
	
	if n != noone {
		if n.x != px {
			array_push(temp_arr, n)
		}
	}
	
	// RIGHT
	n = instance_position(x + sprite_width, y, obj)
	
	if n != noone {
		if n.x != px {
			array_push(temp_arr, n)
		}
	}
	
	// UP
    n = instance_position(x, y-sprite_height, obj)
	
	if n != noone {
		if n.y != py {
			array_push(temp_arr, n)
		}
	}
	
	// DOWN
	n = instance_position(x, y+sprite_height, obj)
	
	if n != noone {
		if n.y != py {
			array_push(temp_arr, n)
		}
	}
	
	n = instance_position(px, py, obj)

	// ESC & RETURN
    if (array_length(temp_arr) == 0) return n
	
    return temp_arr[irandom(array_length(temp_arr) - 1)]
}


```

#### **Step**

```gml
switch(loadicon_state) {

	case LOADICON_STATE.MAKING_BOARD:
	
		if array_length(board_arr) >= 9 {
			x = new_x
			y = new_y
			loadicon_state = LOADICON_STATE.SET_SHUFFLE
		}
	
		break;
		
	case LOADICON_STATE.SET_SHUFFLE:
	
		slide_done = false
		
		neighbor = random_neighbor(obj_loadicon,prev_x,prev_y)
		
		prev_x = x
		prev_y = y
		
		neighbor.new_x = x
		neighbor.new_y = y
		
		new_x = neighbor.x
		new_y = neighbor.y
		
		loadicon_state = LOADICON_STATE.SLIDING

		break;
		
	case LOADICON_STATE.SLIDING:
	
		if neighbor.x < neighbor.new_x {
			neighbor.x += move_speed	
		}
		else if neighbor.x > neighbor.new_x {
			neighbor.x -= move_speed	
		}
		else if neighbor.y < neighbor.new_y {
			neighbor.y += move_speed	
		}
		else if neighbor.y > neighbor.new_y {
			neighbor.y -= move_speed	
		}
		else {
			slide_done = true
		}
	
		if x < new_x {
			x += move_speed	
		}
		else if x > new_x {
			x -= move_speed	
		}
		else if y < new_y {
			y += move_speed	
		}
		else if y > new_y {
			y -= move_speed	
		}
		else {
			if slide_done {
				loadicon_state = LOADICON_STATE.SET_SHUFFLE	
			}
		}
		
		break;
	
}
```



#### **Draw GUI**

```gml
draw_self()
```
