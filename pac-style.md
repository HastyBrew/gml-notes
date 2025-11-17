# PACMAN-Style Game Notes

## **ROOM**

**Size:** 1000x1200
**Grids**: 25x25

## **SPRITES**
*Everything aligned center, except BG*

#### **spr_bg**<br />
- Imported from Ko-Fi
- Set background to spr_bg

#### **spr_player**<br />
- Size: 50x50
- Draw yellow circle, inset 4-5px
- Draw eye
- Duplicate frames and cut out mouth
- Copy the frames to make full animation

#### **spr_ghost**<br />
- Size: 50x50
- Draw red circle, inset
- Draw eyes, two black open circles, fill with white
- Duplicate, name spr_ghost_eyes
- Add frames, colors red, pink, blue, orange, light blue

#### **spr_ghost**<br />
- Delete everything but eyes
- Draw black circle pupils

#### **spr_gate**<br />
- Size: 50x50
- Color: magenta
- Alpha: 50%

#### **spr_node_cross**<br />
- Size: 50x50
- Color: green
- Alpha: 50%
- Duplicate to make spr_note_T and spr_node_L

#### **spr_node_T**<br />
2px line across top

#### **spr_node_L**<br />
2px line across top and right

#### **spr_pellet**<br />
6x6 white circle

#### **spr_powerup**<br />
20x20 white square

## **OBJECTS**

### **obj_controller**

#### **Create**

```gml
global.move_speed = 1.8

global.powered_up = false
global.powerup_timer = 700
global.powerup_timer_total = 700

win_delay = 20

ghosts = {
	
	red: {
		
		index: 0,
		ghost_type: GHOST_TYPE.TRACKING,
		countdown_timer_toal: 500
			
	},
	
	pink: {
		
		index: 1,
		ghost_type: GHOST_TYPE.RANDOM,
		countdown_timer_toal: 300
			
	},
	
	blue: {
		
		index: 2,
		ghost_type: GHOST_TYPE.TRACKING,
		countdown_timer_toal: 700
			
	},
	
	orange: {
		
		index: 3,
		ghost_type: GHOST_TYPE.RANDOM,
		countdown_timer_toal: 200
			
	},
}
```

#### **Game Start**

```gml
global.points = 0
```

### **obj_node_T**

#### **Create**

```gml
depth = 1

image_alpha = 0

can_move_up = false
can_move_left = false
can_move_down = false
can_move_right = false
var in

movement_arr = [0,1,1,1]

shift_index = 0
```

#### **Step**

```gml
if image_angle < 0 {
	shift_index = (360 + image_angle) / 90
}
else {
	shift_index = image_angle / 90
}

var index = 0 - shift_index
if index >= 0 {
	can_move_up = movement_arr[index]	
}
else {
	can_move_up = movement_arr[(array_length(movement_arr))+index]	
}


index = 1 - shift_index
if index >= 0 {
	can_move_left = movement_arr[index]	
}
else {
	can_move_left = movement_arr[(array_length(movement_arr))+index]	
}


index = 2 - shift_index
if index >= 0 {
	can_move_down = movement_arr[index]	
}
else {
	can_move_down = movement_arr[(array_length(movement_arr))+index]	
}


index = 3 - shift_index
if index >= 0 {
	can_move_right = movement_arr[index]	
}
else {
	can_move_right = movement_arr[array_length(movement_arr)+index]	
}
```

Duplicate for obj_node_L and obj_node_cross

### **obj_node_L**

#### **Create**
Change movement_arr
```gml
movement_arr = [0,1,1,0]
```

### **obj_node_cross**

#### **Create**
Change movement_arr
```gml
movement_arr = [1,1,1,1]
```
Test them

### **obj_pellet**

#### **Create**

```gml
depth = 7
```

### **obj_powerup**

#### **Create**

```gml
depth = 7
```

### **obj_player**

#### **Create**

```gml
depth = 3

image_speed = 0.8

enum PLAYER_STATE {

	MOVING_LEFT,
	MOVING_RIGHT,
	MOVING_UP,
	MOVING_DOWN,
	STOPPED,
	HIT
	
}

player_state = PLAYER_STATE.MOVING_RIGHT

moving_vert = false
moving_horiz = true

node = noone

tl = 2

restart_timer = 50

shrink_size = 1
```

#### **Step**

```gml
var hitting_t = collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_node_T,0,0)
var hitting_l = collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_node_L,0,0)
var hitting_cross = collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_node_cross,0,0)


if hitting_t {
	node = hitting_t	
}
else if hitting_l {
	node = hitting_l	
}
else if hitting_cross {
	node = hitting_cross	
}
else {
	node = noone	
}


switch(player_state) {
	
		case PLAYER_STATE.MOVING_LEFT:
		
			moving_horiz = true
			moving_vert = false
		
			x -= global.move_speed
			image_angle = 0
			image_xscale = -1
			
			if node {
				if x > node.x-tl and x < node.x+tl and !node.can_move_left {
					x = node.x
					player_state = PLAYER_STATE.STOPPED
				}
			}
		
			break;
			
		case PLAYER_STATE.MOVING_RIGHT:
		
			moving_horiz = true
			moving_vert = false
		
			x += global.move_speed
			image_angle = 0
			image_xscale = 1
			
			if node {
				if x > node.x-tl and x < node.x+tl and !node.can_move_right {
					x = node.x
					player_state = PLAYER_STATE.STOPPED
				}
			}
		
			break;
	
		case PLAYER_STATE.MOVING_UP:
		
			moving_horiz = false
			moving_vert = true
		
			y -= global.move_speed
			image_angle = 90
			image_xscale = 1
			
			if node {
				if y > node.y-tl and y < node.y+tl and !node.can_move_up {
					y = node.y
					player_state = PLAYER_STATE.STOPPED
				}
			}
		
			break;
			
		case PLAYER_STATE.MOVING_DOWN:
		
			moving_horiz = false
			moving_vert = true
		
			y += global.move_speed
			image_angle = 90
			image_xscale = -1
	
			if node {
				if y > node.y-tl and y < node.y+tl and !node.can_move_down {
					y = node.y
					player_state = PLAYER_STATE.STOPPED
				}
			}
			
			break;
			
		case PLAYER_STATE.STOPPED:
			
			if node {
				
				x = node.x
				y = node.y
				
			}
		
			break;
			
		case PLAYER_STATE.HIT:
		
			image_angle += 10
			
			image_xscale = shrink_size
			image_yscale = shrink_size
			
			shrink_size -= 0.015
			
			if restart_timer > 0 {
				restart_timer--	
			}
			else {
				global.points = 0
				room_restart()	
			}
		
			break;
	
}
if player_state != PLAYER_STATE.HIT {
	if keyboard_check(vk_left) {

		if moving_horiz and !node {
			player_state = PLAYER_STATE.MOVING_LEFT	
		}
		else if node {
			if y > node.y-tl and y < node.y+tl and node.can_move_left {
				y = node.y
				player_state = PLAYER_STATE.MOVING_LEFT
			}
		}
	
	}


	if keyboard_check(vk_right) {

		if moving_horiz and !node {
			player_state = PLAYER_STATE.MOVING_RIGHT
		}
		else if node {
			if y > node.y-tl and y < node.y+tl and node.can_move_right {
				y = node.y
				player_state = PLAYER_STATE.MOVING_RIGHT
			}
		}
	
	}


	if keyboard_check(vk_up) {

		if moving_vert and !node {
			player_state = PLAYER_STATE.MOVING_UP
		}
		else if node {
			if x > node.x-tl and x < node.x+tl and node.can_move_up {
				x = node.x
				player_state = PLAYER_STATE.MOVING_UP
			}
		}
	
	}


	if keyboard_check(vk_down) {

		if moving_vert and !node {
			player_state = PLAYER_STATE.MOVING_DOWN
		}
		else if node {
			if x > node.x-tl and x < node.x+tl and node.can_move_down {
				x = node.x
				player_state = PLAYER_STATE.MOVING_DOWN
			}
		}
	
	}
}

if player_state == PLAYER_STATE.STOPPED {
	image_speed = 0
	image_index = 5
}
else {
	image_speed = 0.8	
}

if (x<-10) x = room_width + 10
if (x>room_width+10) x = -10
```

#### **Collision > obj_pellet**

```gml
global.points++
instance_destroy(other)
```

#### **Collision > obj_powerup**

```gml
global.powered_up = true
global.powerup_timer = global.powerup_timer_total
instance_destroy(other)
```

Drop in room, test with nodes
Drop in the rest of the nodes
Drop in pellets

### **obj_gate**

#### **Create**

```gml
image_speed = 0
image_alpha = 0
```

Drop in center, 6x2 grid-spaces wide

### **obj_ghost_eyes**

#### **Create**

```gml
depth = 5
```

### **obj_node_ghost**

#### **Create**

```gml
image_speed = 0
image_alpha = 0
```

### **obj_ghost**

#### **Create**

```gml
image_index = 0
image_speed = 0
depth = 6

enum GHOST_TYPE {
	
	NULL,
	RANDOM,
	FIXED,
	RUNNING,
	TRACKING
	
}

ghost_type = GHOST_TYPE.NULL

enum GHOST_STATE {

	BOXED,
	MOVING_TO_PATH,
	CHOOSING_PATH,
	MOVING_LEFT,
	MOVING_RIGHT,
	MOVING_UP,
	MOVING_DOWN,
	BACK_TO_START
	
}

ghost_state = GHOST_STATE.BOXED

opp_dir = GHOST_STATE.MOVING_RIGHT

eyes = instance_create_layer(x,y,"Instances",obj_ghost_eyes)

moving_vert = false
moving_horiz = true

floating_right = choose(true,false)

move_direction = 0

node = noone
tl = 2

origin_x = x
origin_y = y

movement_arr = []

node_reset = true

node_delay = 0
node_delay_total = 2

countdown_timer_total = random_range(500,1000)
countdown_timer = countdown_timer_total

move_speed = global.move_speed * 0.75
```

#### **Step**

```gml
eyes.x = x
eyes.y = y


if ghost_state != GHOST_STATE.BOXED and ghost_state != GHOST_STATE.MOVING_TO_PATH {
	
	if global.powered_up {
		if global.powerup_timer > 250 {
			image_index = 4
		}
		else if floor(global.powerup_timer/20) mod 2 {
			image_index = 4
		}
		else {
			image_index = ghost_data.index
		}
	}
	else {
		image_index = ghost_data.index
	}
	
}


var touching_gate = collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_gate,0,0)

var hitting_t = collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_node_T,0,0)
var hitting_l = collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_node_L,0,0)
var hitting_cross = collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_node_cross,0,0)

if hitting_t {
	node = hitting_t	
}
else if hitting_l {
	node = hitting_l	
}
else if hitting_cross {
	node = hitting_cross	
}
else {
	node = noone	
}

switch(ghost_state) {
	
	case GHOST_STATE.BOXED:
	
		image_index = ghost_data.index
		ghost_type =  ghost_data.ghost_type
		
		if touching_gate {
			if floating_right {
				image_xscale = 1
				eyes.image_xscale = 1
				x += move_speed / 2	
			}
			else {
				image_xscale = -1
				eyes.image_xscale = -1
				x -= move_speed	/ 2 
			}
		}
		else {
			if floating_right {
				x -= move_speed
				floating_right = false	
			}
			else {
				x += move_speed
				floating_right = true
			}
		}
		
		if countdown_timer > 0 {
			countdown_timer--	
		}
		else {
			if instance_exists(obj_node_ghost) {
				if x < obj_node_ghost.x + tl and x > obj_node_ghost.x - tl {
					ghost_state = GHOST_STATE.MOVING_TO_PATH
				}
			}
		}
	
	
		break;
		
	case GHOST_STATE.MOVING_TO_PATH:
	
		if instance_exists(obj_node_ghost) {
			if y > obj_node_ghost.y {
				y --	
			}
			else if y <= obj_node_ghost.y {
				y = obj_node_ghost.y
				ghost_state = choose(GHOST_STATE.MOVING_LEFT,GHOST_STATE.MOVING_RIGHT)
			}
		}
	
		break;
		
	case GHOST_STATE.MOVING_LEFT:
		
			opp_dir = GHOST_STATE.MOVING_RIGHT
		
			moving_horiz = true
			moving_vert = false
			
			movement_arr = []
		
			x -= move_speed
			image_xscale = -1
			eyes.image_xscale = -1
			
			
			if node and node_delay <= 0 {
				if x > node.x-tl and x < node.x+tl {
					x = node.x
					ghost_state = GHOST_STATE.CHOOSING_PATH
				}
			}
		
			break;
			
		case GHOST_STATE.MOVING_RIGHT:
		
			opp_dir = GHOST_STATE.MOVING_LEFT
		
			moving_horiz = true
			moving_vert = false
			
			movement_arr = []
			
		
			x += move_speed
			image_xscale = 1
			eyes.image_xscale = 1
			
			
			if node and node_delay <= 0 {
				if x > node.x-tl and x < node.x+tl {
					x = node.x
					ghost_state = GHOST_STATE.CHOOSING_PATH
				}
			}
		
			break;
	
		case GHOST_STATE.MOVING_UP:
		
			opp_dir = GHOST_STATE.MOVING_DOWN
		
			moving_horiz = false
			moving_vert = true
			
			movement_arr = []
		
			y -= move_speed
			
			
			if node and node_delay <= 0 {
				if y > node.y-tl and y < node.y+tl {
					y = node.y
					ghost_state = GHOST_STATE.CHOOSING_PATH
				}
			}
		
			break;
			
		case GHOST_STATE.MOVING_DOWN:
		
			opp_dir = GHOST_STATE.MOVING_UP
		
			moving_horiz = false
			moving_vert = true
			
			movement_arr = []
		
			y += move_speed
			
	
			if node and node_delay <= 0 {
				if y > node.y-tl and y < node.y+tl {
					y = node.y
					ghost_state = GHOST_STATE.CHOOSING_PATH
				}
			}
			
			break;
			
		case GHOST_STATE.BACK_TO_START:
		
			if y < origin_y {
				y += global.move_speed
			}
			else if y > origin_y {
				y -= global.move_speed
			}
			
			if y < origin_y +(tl*2) and y > origin_y-(tl*2) {
				y = origin_y	
			}
			
			if x < origin_x {
				x += global.move_speed
			}
			else if x > origin_x {
				x -= global.move_speed
			}
			
			if x < origin_x +(tl*2) and x > origin_x-(tl*2) {
				x = origin_x
			}
			
			if x == origin_x and y == origin_y {
				countdown_timer = countdown_timer_total
				ghost_state = GHOST_STATE.BOXED	
			}
			
			
		
			break;
			
		case GHOST_STATE.CHOOSING_PATH:
		
			switch(ghost_type) {
			
				case GHOST_TYPE.RANDOM:
				
					if node {
						if (node.can_move_left) array_push(movement_arr,GHOST_STATE.MOVING_LEFT)
						if (node.can_move_right) array_push(movement_arr,GHOST_STATE.MOVING_RIGHT)
						if (node.can_move_up) array_push(movement_arr,GHOST_STATE.MOVING_UP)
						if (node.can_move_down) array_push(movement_arr,GHOST_STATE.MOVING_DOWN)
					}
					
					node_delay = node_delay_total
					ghost_state = movement_arr[irandom(array_length(movement_arr)-1)]
				
					break;
					
				case GHOST_TYPE.TRACKING:
			
				if node and instance_exists(obj_player) {
					if node.can_move_left and obj_player.x < x {
						array_push(movement_arr,GHOST_STATE.MOVING_LEFT)	
					}
					if node.can_move_right and obj_player.x > x {
						array_push(movement_arr,GHOST_STATE.MOVING_RIGHT)	
					}
					if node.can_move_up and obj_player.y < y {
						array_push(movement_arr,GHOST_STATE.MOVING_UP)	
					}
					if node.can_move_down and obj_player.y > y {
						array_push(movement_arr,GHOST_STATE.MOVING_DOWN)	
					}
					node_delay = node_delay_total
					if array_length(movement_arr) > 0 {
						ghost_state = movement_arr[irandom(array_length(movement_arr)-1)]
					}
					else {
						ghost_state = opp_dir
					}
				}
				
					break;
					
				case GHOST_TYPE.RUNNING:
				
					if node and instance_exists(obj_player) {
					if node.can_move_left and obj_player.x > x {
						array_push(movement_arr,GHOST_STATE.MOVING_LEFT)	
					}
					if node.can_move_right and obj_player.x < x {
						array_push(movement_arr,GHOST_STATE.MOVING_RIGHT)	
					}
					if node.can_move_up and obj_player.y > y {
						array_push(movement_arr,GHOST_STATE.MOVING_UP)	
					}
					if node.can_move_down and obj_player.y < y {
						array_push(movement_arr,GHOST_STATE.MOVING_DOWN)	
					}
					node_delay = node_delay_total
					if array_length(movement_arr) > 0 {
						ghost_state = movement_arr[irandom(array_length(movement_arr)-1)]
					}
					else {
						ghost_state = opp_dir
					}
				}
					break;
			
			
			}
		
			break;
	
}

if node_delay > 0 {
	node_delay--	
}

if (x<-10) x = room_width + 10
if (x>room_width+10) x = -10

if (eyes.x<-10) x = room_width + 10
if (eyes.x>room_width+10) x = -10

if ghost_state == GHOST_STATE.BACK_TO_START {
	image_alpha = 0	
}
else {
	if image_alpha < 1 {
		image_alpha += 0.05	
	}
}
```

#### **Collision > obj_player**

```gml

if ghost_state != GHOST_STATE.BACK_TO_START {
	if !global.powered_up {
		obj_player.player_state = PLAYER_STATE.HIT	
	}
	else {
		ghost_state = GHOST_STATE.BACK_TO_START	
	}
}
```


### **obj_controller (again)**

#### **Create**

```gml
if instance_exists(obj_gate) {
	
	ghost = instance_create_layer(obj_gate.x-75,obj_gate.y,"Instances",obj_ghost)
	ghost.ghost_data = ghosts.red
	
	ghost = instance_create_layer(obj_gate.x-25,obj_gate.y,"Instances",obj_ghost)
	ghost.ghost_data = ghosts.pink
	
	ghost = instance_create_layer(obj_gate.x+25,obj_gate.y,"Instances",obj_ghost)
	ghost.ghost_data = ghosts.blue
	
	ghost = instance_create_layer(obj_gate.x+75,obj_gate.y,"Instances",obj_ghost)
	ghost.ghost_data = ghosts.orange
}
```

#### **Step**

```gml
if win_delay > 0 {
	win_delay--	
}
else {

	if !instance_exists(obj_pellet)	{
		room_restart()	
	}
	
}

if global.powered_up {
	if global.powerup_timer > 0 {
		global.powerup_timer--	
	}
	else {
		global.powered_up = false	
	}
}
```

