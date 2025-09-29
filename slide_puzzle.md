# Slide Puzzle Game Notes

## **ART CREDITS**

**Figures On The Beach** *- Auguste Renoir*

**The Bathing Pool** *- Hubert Robert*

**Man with a Beard** *- Style of Rembrandt*

**Olive Trees** *- Vincent Van Gogh*

**My Bunkie** *- Charles Schreyvogel*

## **ROOM**

**Size:** 100x100

 **Background color: Dark Burgundy**

## **SPRITES**

**spr_renoir** (imported from Ko-Fi)

## **FONTS**

- **fnt_title:**  Roboto, Black, 30pt
- **fnt_author:** Roboto, Black Italic, 30pt

## **OBJECTS**

### **obj_controller**

#### **Create**

```gml
randomize()

// GLOBALS

global.move_speed = 30
global.shuffle_speed = 60

global.shuffle_counter = floor(random_range(20,50))
global.correct_tiles = 0

global.game_finished = false

text_alpha = 0

// CREATE THE BOARD

slide_arr = [noone]

columns = 0
rows = 0

offset = 50
square_size = 300

mt = noone

for (var i = 0; i < 9; i++) {
	if i == 0 {
		mt = instance_create_layer(offset,offset,"Instances",obj_slide_empty)
		rows++
	}
	else {
		slide_arr[i] = instance_create_layer(offset+(square_size*rows),offset+(square_size*columns),"Instances",obj_slide_pic)
		slide_arr[i].image_index = i
		
		rows++
		if rows > 2 {
			columns++
			rows = 0
		}
	}
}
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



#### **Draw**

```gml

draw_set_alpha(text_alpha)

draw_set_color(c_white)

draw_set_font(fnt_title)
draw_text(80,80,"Figures On The Beach")

draw_set_font(fnt_author)
draw_text(80,130,"Auguste Renoir")

draw_set_alpha(1)
```

### **obj_slide_empty**

#### **Create**

```gml

depth = 7
image_speed = 0
image_index = 0
image_alpha = 0

initial_x = x
initial_y = y

prev_x = x
prev_y = y

new_x = x
new_y = y

pic_slide_done = false

enum SHUFFLE_STATE {
	
	SET_SHUFFLE,
	SLIDING,
	DONE
	
}

shuffle_state = SHUFFLE_STATE.SET_SHUFFLE

neighbor = noone

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

switch(shuffle_state) {

	case SHUFFLE_STATE.SET_SHUFFLE:
	
		pic_slide_done = false
	
		neighbor = random_neighbor(obj_slide_pic,prev_x,prev_y)

		prev_x = x
		prev_y = y
		
		neighbor.new_x = x
		neighbor.new_y = y
		
		new_x = neighbor.x
		new_y = neighbor.y
		
		shuffle_state = SHUFFLE_STATE.SLIDING
	
		break;
		
	case SHUFFLE_STATE.SLIDING:
	
		if neighbor.x < neighbor.new_x {
			neighbor.x += global.shuffle_speed
		}
		else if neighbor.x > neighbor.new_x {
			neighbor.x -= global.shuffle_speed
		}
		else if neighbor.y < neighbor.new_y {
			neighbor.y += global.shuffle_speed
		}
		else if neighbor.y > neighbor.new_y {
			neighbor.y -= global.shuffle_speed
		}
		else {
			pic_slide_done = true
		}
	
		if x < new_x {
			x += global.shuffle_speed	
		}
		else if x > new_x {
			x -= global.shuffle_speed
		}
		else if y < new_y {
			y += global.shuffle_speed
		}
		else if y > new_y {
			y -= global.shuffle_speed	
		}
		else {
			if pic_slide_done {
				if global.shuffle_counter > 0 {
					global.shuffle_counter--
					shuffle_state = SHUFFLE_STATE.SET_SHUFFLE
				}
				else {
					shuffle_state = SHUFFLE_STATE.DONE	
				}
				
			}
		}
	
		break;
	
}

if global.game_finished and shuffle_state != SHUFFLE_STATE.SLIDING {
	if image_alpha < 1 {
		image_alpha += 0.01	
	}
}
```

### **obj_slide_pic**

#### **Create**

```gml

depth = 5
image_speed = 0
image_index = 0
image_alpha = 1

sliding = false
moving_vert = choose(true,false)

initial_x = x
initial_y = y

prev_x = x
prev_y = y

new_x = x
new_y = y


enum SLIDE_STATE {
	
	SHUFFLING,
	SLIDING,
	READY,
	FINISHED
	
	
}

slide_state = SLIDE_STATE.SHUFFLING
empty_slide_done = false
```

#### **Step**

```gml

var touching_vertical = collision_rectangle(bbox_left-1,bbox_top,bbox_right+1,bbox_bottom,obj_slide_empty,false,false)
var touching_horizontal = collision_rectangle(bbox_left,bbox_top-1,bbox_right,bbox_bottom+1,obj_slide_empty,false,false)

var mt = obj_slide_empty

switch(slide_state) {

	case SLIDE_STATE.SHUFFLING:
	
		if global.shuffle_counter <= 0 {
			slide_state = SLIDE_STATE.READY	
		}
	
		break;
		
	case SLIDE_STATE.SLIDING:
		
		moving_vert = choose(true,false)
		if mt.x < mt.new_x {
			mt.x += global.move_speed	
		}
		else if mt.x > mt.new_x {
			mt.x -= global.move_speed	
		}
		else if mt.y < mt.new_y {
			mt.y += global.move_speed	
		}
		else if mt.y > mt.new_y {
			mt.y -= global.move_speed	
		}
		else {
			empty_slide_done = true	
		}
	
		if x < new_x {
			x += global.move_speed	
		}
		else if x > new_x {
			x -= global.move_speed	
		}
		else if y < new_y {
			y += global.move_speed	
		}
		else if y > new_y {
			y -= global.move_speed	
		}
		else {
			if empty_slide_done {
				slide_state = SLIDE_STATE.READY
			}
		}
	
		break;
		
	case SLIDE_STATE.READY:
	
		if global.correct_tiles > 7 {
			slide_state = SLIDE_STATE.FINISHED	
		}
		else {
			global.correct_tiles = 0	
		}
	
		empty_slide_done = false
	
		if mouse_check_button_released(mb_left) and point_in_rectangle(mouse_x,mouse_y,x,y,x+sprite_width,y+sprite_height) {
			
			if touching_horizontal or touching_vertical {
				
				new_x = obj_slide_empty.x 
				new_y = obj_slide_empty.y 
				
				obj_slide_empty.new_x = x
				obj_slide_empty.new_y = y
				with(obj_slide_pic) {
					if new_x == initial_x and new_y == initial_y {
						global.correct_tiles ++	
					}
				}
				show_debug_message(global.correct_tiles)
				slide_state = SLIDE_STATE.SLIDING
				
				
			}
			
		}
	
		break;
		
	case SLIDE_STATE.FINISHED:
	
		global.game_finished = true
	
		break;
		
}

if instance_exists(obj_slide_empty) {
	if obj_slide_empty.shuffle_state == SHUFFLE_STATE.DONE {
		if image_alpha < 1 {
			image_alpha += 0.1	
		}
	}
}
```

