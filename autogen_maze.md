# Auto-Generated Maze Notes

## **ROOM**

**Size:** 850x1000

 **Background color:** Dark Gray

## **SPRITES**

**spr_square** 50x50
Frames: Light Gray, Black, Red

**spr_player** 30x30, Blue

## **OBJECTS**

- **obj_controller**  drop in room
- **obj_path** spr_square
- **obj_border** spr_square, Solid
- **obj_player** spr_player, Solid

### **obj_controller**

#### **Create**

```gml
global.move_speed = 5

global.maze_done = false

instance_create_layer((irandom(7)+1)*100,room_height-50,"Instances",obj_path)
```

#### **Step**

```gml

if instance_exists(obj_slide_empty) {
	if global.game_finished and obj_slide_empty.image_alpha >= 1 {
		if text_alpha < 1 {
			text_alpha += 0.05
		}
	
	}
}
```

### **obj_border**

#### **Create**

```gml
image_speed = 0
image_index = 1
image_alpha = 1


Step

if global.maze_done {
	if image_alpha < 1 {
		image_alpha += 0.05
	}
}
```

### **obj_player**

#### **Create**

```gml
depth = 2
```

#### **Step**

```gml

if keyboard_check(vk_left) and x > 0 {

	x -= global.move_speed	

}
else if keyboard_check(vk_right) and x < room_width - sprite_width {

	x += global.move_speed	

}

if keyboard_check(vk_up) {

	y -= global.move_speed	

}
else if keyboard_check(vk_down) and y < room_height - sprite_height {

	y += global.move_speed	

}

if y < 0 {
	room_restart()	
}
```

#### **Collision > obj_border**
Create Event and leave blank
```gml
// leave blank
```

### **obj_path**

#### **Create**

```gml

depth = 5

image_speed = 0
image_index = 0

image_alpha = 1

randomize()

// ENUM

enum PATH_STATE {
	
	NULL,
	SEARCHING,
	SETTING,
	BACKTRACKING,
	SETTING_BORDER,
	DONE
	
}

total_columns = room_width / sprite_width
total_rows = room_height / sprite_height

rows = 0
columns = 0

border_arr = []


path_state = PATH_STATE.NULL

// VARS

if instance_number(obj_path) < 2 {
    master = true
	path_state = PATH_STATE.SEARCHING
	image_index = 2
} else {
    master = false
}

ender = false

prev_x = x
prev_y = y

new_x = x
new_y = y

position_arr = []

neighbor = []



// FUNCTION

function random_neighbor(obj,px,py) {
	
    var temp_arr = []
	
	var dx = x - px
	var dy = y - py

	// LEFT
    var nx = instance_position(x - sprite_width, y, obj)
	var nx2 = instance_position(x - (sprite_width*2), y, obj)
	
	if nx == noone and nx2 == noone {
		var tx = x - sprite_width * 2
		var ty = y
		if tx != px and tx >= 0 {
			
			array_push(temp_arr, [tx,ty])
			
			if dx < 0 {
				array_push(temp_arr, [tx, ty])
				array_push(temp_arr, [tx, ty])
			}
		}
	}
	
	// RIGHT
    nx = instance_position(x + sprite_width, y, obj)
	nx2 = instance_position(x + (sprite_width*2), y, obj)
	
	if nx == noone and nx2 == noone {
		var tx = x + sprite_width * 2
		var ty = y
		if tx != px and tx <= room_width {
			
			array_push(temp_arr, [tx,ty])
			
			if dx < 0 {
				array_push(temp_arr, [tx, ty])
				array_push(temp_arr, [tx, ty])
			}
		}
	}
	
	// UP
    nx = instance_position(x, y - sprite_height, obj)
	nx2 = instance_position(x, y - (sprite_height*2), obj)
	
	if nx == noone and nx2 == noone {
		var tx = x
		var ty = y - sprite_height * 2
		if ty != py and ty >= 0 {
			
			array_push(temp_arr, [tx,ty])
			
		}
	}
	
	// DOWN
    nx = instance_position(x, y + sprite_height, obj)
	nx2 = instance_position(x, y + (sprite_height*2), obj)
	
	if nx == noone and nx2 == noone {
		var tx = x
		var ty = y + sprite_height * 2
		if ty != py and ty <= room_height {
			
			array_push(temp_arr, [tx,ty])
			
			if dy < 0 {
				array_push(temp_arr, [tx, ty])
				array_push(temp_arr, [tx, ty])
				array_push(temp_arr, [tx, ty])
			}
		}
	}

	// ESC & RETURN
    if (array_length(temp_arr) == 0) return noone
	
    return temp_arr[irandom(array_length(temp_arr) - 1)]
}
```

#### **Step**

```gml

if master {
	switch(path_state) {
	
		case PATH_STATE.SEARCHING:
		
				neighbor = random_neighbor(obj_path,prev_x,prev_y)
			
				if neighbor == noone {
					path_state = PATH_STATE.BACKTRACKING	
				}
				else {
					path_state = PATH_STATE.SETTING	
				}
		
				break;
	
		case PATH_STATE.SETTING:
	
			instance_create_layer(x,y,"Instances",obj_path)
			
			var mx = x
			var my = y
			
			if neighbor[0] != x {
				mx = x + sign(neighbor[0] - x) * sprite_width;
			}
			
			if neighbor[1] != y {
				my = y + sign(neighbor[1] - y) * sprite_height;
			}
			
			instance_create_layer(mx, my, "Instances", obj_path)
		
			prev_x = x
			prev_y = y
			
			position_arr[array_length(position_arr)] = [prev_x, prev_y];
		
			x = neighbor[0]
			y = neighbor[1]
		
			path_state = PATH_STATE.SEARCHING
	
			break;
		
		case PATH_STATE.BACKTRACKING:
	
			if array_length(position_arr) > 0 {
	
				var last = array_pop(position_arr)
		
				prev_x = x
				prev_y = y
	
				x = last[0]
				y = last[1]
		
				path_state = PATH_STATE.SEARCHING
			}
			else {
				path_state = PATH_STATE.DONE	
			}
	
			break;
			
		case PATH_STATE.SETTING_BORDER:	
		
			for (var i = 0;i<340;i++) {
				if i == 339 {
					global.maze_done = true	
				}
				if !position_meeting(sprite_width*rows,sprite_height*columns,obj_path) {
					border_arr[i] = instance_create_layer(sprite_width*rows,sprite_height*columns,"Instances",obj_border)
					border_arr[i].image_index = 1
				}		
				rows++
				if rows > 17 {
					columns++
					rows = 0
				}
			}
			
			path_state = PATH_STATE.NULL
			
		
			break;
			
		case PATH_STATE.NULL:
			if global.maze_done {
				instance_destroy()	
			}
			break;
			
			
		case PATH_STATE.DONE:
			
			var end_tile = instance_create_layer((irandom(6)+1)*100,0,"Instances",obj_path)
			instance_create_layer(end_tile.x+sprite_width,end_tile.y,"Instances",obj_path)
			end_tile.image_index = 2
			end_tile.depth = 4
			instance_create_layer(x+10,y+10,"Instances",obj_player)
			path_state = PATH_STATE.SETTING_BORDER
		
			break;
	
	}
	

}
else if path_state == PATH_STATE.NULL{
		
	if global.maze_done {
		instance_destroy()	
	}
			
}
```

