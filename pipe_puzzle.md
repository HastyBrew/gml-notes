# Pipe Puzzle Game

## **ROOM**
Size: 800x800
Color: Dark Blue

## **SPRITES**

### **spr_pipe_master**
Original Size: 25x25

Frames:
1. Straight Vertical Gray Line
2. Corner Gray Line (Up and Right)
3. Straight Vertical Gray Line Line leading to Gray Box

**Important!** Upgrade spr_pipe_master to 100x100 before duplicating

Duplicate spr_pipe_master to make

1. spr_pipe_completed
2. spr_pipe_corner
3. spr_pipe_decoy
4. spr_pipe_ends
5. spr_pipe_straight

For spr_pipe_corner, spr_pipe_ends, and spr_pipe_straight, delete all frames but 1,2, and 0 respectively <br />
For spr_pipe_decoy, add a magenta first frame

## **OBJECTS**

### **obj_pipe_controller**

#### **Create**

```gml
randomize()

global.can_press = true
global.game_won = false

global.overlay_alpha = 0

global.pipe_ids = []
```
Drop in room

### **obj_pipe_overlay**

#### **Create**

```gml
depth = 2

image_speed = 0
image_index = 0

image_alpha = 0
```

#### **Step**

```gml
image_alpha = global.overlay_alpha
```

### **obj_pipe_ends**

#### **Create**

```gml
image_speed = 0
image_index = 0

depth = 5

pipe_overlay = noone
```

#### **Step**

```gml
if global.game_won {
	if !instance_exists(pipe_overlay) {
		pipe_overlay = instance_create_layer(x,y,"Instances",obj_pipe_overlay)
		pipe_overlay.image_index = 2
		pipe_overlay.image_angle = image_angle
	}
}
```

### **obj_pipe_corner**

#### **Create**

```gml
array_push(global.pipe_ids,id)
image_speed = 0
image_index = 0

depth = 5

if image_angle < 0 {
	image_angle += 360
}
else if image_angle > 360 {
	image_angle -= 360	
}

og_image_angle = image_angle
new_angle = image_angle


move_arrays = [90,180,270,0]
random_index = irandom(3)

random_move = move_arrays[random_index]

is_rotating = false

is_correct = false

pipe_overlay = noone

image_angle = random_move
```

#### **Step**

```gml
if image_angle == og_image_angle {
	is_correct = true	
}
else {
	is_correct = false	
}

if !global.game_won {
	if !is_rotating {
		if point_in_rectangle(mouse_x,mouse_y,bbox_left,bbox_top,bbox_right,bbox_bottom) {

			if mouse_check_button_released(mb_left) {
				
				new_angle = image_angle + 90
				if (new_angle > 360) new_angle -= 360
					
					
				is_rotating = true
		
			}
	
		}
	}
	else {
		if image_angle < new_angle {
			image_angle += 5
		}
		else {
			if image_angle == 360 {
				new_angle = 0
				image_angle = new_angle
			}
			image_angle = new_angle
			is_rotating = false
		}
	}
}
else {
	if !instance_exists(pipe_overlay) {
		pipe_overlay = instance_create_layer(x,y,"Instances",obj_pipe_overlay)
		pipe_overlay.image_index = 1
		pipe_overlay.image_angle = image_angle
	}
}
```

### **obj_pipe_straight**

#### **Create**

*Copy/Paste Create Event from obj_pipe_corner*

```gml
array_push(global.pipe_ids,id)
image_speed = 0
image_index = 0

depth = 5

if image_angle < 0 {
	image_angle += 360
}
else if image_angle > 360 {
	image_angle -= 360	
}

og_image_angle = image_angle
new_angle = image_angle


move_arrays = [90,180,270,0]
random_index = irandom(3)

random_move = move_arrays[random_index]

is_rotating = false

is_correct = false

pipe_overlay = noone

image_angle = random_move
```

#### **Step**

*Almost the same Step Event from obj_pipe_corner, changes to beginning if statement and pipe_overlay.image_index*

```gml
if image_angle == og_image_angle {
	is_correct = true	
}
else if image_angle == og_image_angle + 180 or image_angle == og_image_angle - 180 {
	is_correct = true	
}
else {
	is_correct = false	
}


if !global.game_won {
	if !is_rotating {
		if point_in_rectangle(mouse_x,mouse_y,bbox_left,bbox_top,bbox_right,bbox_bottom) {

			if mouse_check_button_released(mb_left) {
				
				new_angle = image_angle + 90
				if (new_angle > 360) new_angle -= 360
					
					
				is_rotating = true
		
			}
	
		}
	}
	else {
		if image_angle < new_angle {
			image_angle += 5
		}
		else {
			if image_angle == 360 {
				new_angle = 0
				image_angle = new_angle
			}
			show_debug_message("OG angle " + string(og_image_angle))
			show_debug_message("Image Angle " + string(image_angle))
			image_angle = new_angle
			is_rotating = false
		}
	}
}
else {
	if !instance_exists(pipe_overlay) {
		pipe_overlay = instance_create_layer(x,y,"Instances",obj_pipe_overlay)
		pipe_overlay.image_angle = image_angle
	}
}
```
### **obj_pipe_decoy**

#### **Create**

*Copy/Paste Create Event from obj_pipe_corner*

```gml
image_speed = 0
image_index = choose(1,2)

move_arrays = [90,180,270,0]
random_index = irandom(3)

random_move = move_arrays[random_index]

is_rotating = false

new_angle = image_angle
image_angle = random_move
```

#### **Step**

*Copy/Paste code snippet from other Step Events*

```gml
if !global.game_won {
	if !is_rotating {
		if point_in_rectangle(mouse_x,mouse_y,bbox_left,bbox_top,bbox_right,bbox_bottom) {

			if mouse_check_button_released(mb_left) {
	
				new_angle = image_angle + 90
				if (new_angle > 360) new_angle -= 360
					
					
				is_rotating = true
		
			}
	
		}
	}
	else {
		if image_angle < new_angle {
			image_angle += 5
		}
		else {
			if image_angle > 270 {
				new_angle = image_angle - 360	
			}
			image_angle = new_angle
			is_rotating = false
		}
	}
}
```

### **obj_pipe_controller (again)**

#### **Step**


```gml
if array_length(global.pipe_ids) > 0 {
	
	var corrects = 0
	
	for (var i = 0;i<array_length(global.pipe_ids);i++) {
		if global.pipe_ids[i].is_correct {
			corrects++
			if corrects >= array_length(global.pipe_ids) {
				global.game_won = true	
			}
		}
		else {
			corrects = 0	
		}
	}
}

if global.game_won {
	if global.overlay_alpha < 1 {
		global.overlay_alpha += 0.2	
	}
}
```
