# Escape Room Tutorial Data

Here's the struct you'll need to complete the tutorial:

```gml
global.data = {
	bedside: {
		painting: {
			cursor: CURSOR_STATE.MAGNIFYING,
			command_text: "Look At Painting",
			dark_text: "I can't see anything",
			description: "It's a modern art painting. Is it good? I can't tell."
		},
		knobs: {
			cursor: CURSOR_STATE.HAND,
			command_text: "Use Knob",
			description: "I think they're just glued on for decoration."
		}
	},
	curtains: {
		closed: {
			cursor: CURSOR_STATE.HAND,
			command_text: "Open Curtains",
		},
		window: {
			cursor: CURSOR_STATE.MAGNIFYING,
			command_text: "Look At Window",
			description: "Ah, the beauty of nature. Mountains as far as the eye can see. Majestic."	
		},
		note: {
			cursor: CURSOR_STATE.HAND,
			command_text: "Pick Up Note",	
		}	
	},
	desk: {
		partyhat: {
			cursor: CURSOR_STATE.MAGNIFYING,
			command_text: "Look At Hat",
			dark_text: "I can't see anything",
			description: "It's a party hat from my birthday last night. I don't even remember wearing it."	
		},
		mirror: {
			cursor: CURSOR_STATE.MAGNIFYING,
			command_text: "Look At Mirror",
			dark_text: "I can't see anything",
			description: "No way, I played Disco Elysium and never looked in a mirror again."
		},
		key: {
			cursor: CURSOR_STATE.HAND,
			command_text: "Pick Up Key",	
		}
	},
	door: {
		lock: {
			cursor: CURSOR_STATE.HAND,
			command_text: "Use Lock",
			description: "It looks like it takes a key."
		},
		calendar: {
			cursor: CURSOR_STATE.MAGNIFYING,
			command_text: "Look At Calendar",
			dark_text: "I can't see anything",
			description: "It's a calendar showing the date is October 23. It's off by two days."
		},
		peephole: {
			cursor: CURSOR_STATE.MAGNIFYING,
			command_text: "Look At Peephole",
			dark_text: "I can't see anything",
			description: "It's a hallway."	
		}
	},
}
```

# Escape Room Game Notes

## **ROOMS**

**Size:** 800x750

- **Rm_Start**
- **Bedroom** *(create group)*
  -  **Rm_Bedside**
  -  **Rm_Curtains**
  -  **Rm_Desk**
  -  **Rm_Drawer**
- **Rm_End**

## **SPRITES**

**Imported from Ko-Fi**

*Use these file names ONLY, the _strip convention is used by GM*
- **spr_bg_strip6**
- **spr_digits_strip11**
- **spr_lights_strip4**
- **spr_cursor_strip4**
- **spr_items_strip2**
- **spr_note**
- **spr_btn**
- **spr_safeplate**

**Made in GM**

- **spr_overlay** Size: 800x600, Black
- **spr_placeholder** Size: 50x50, Magenta, Alpha: 50%

## **OBJECTS**

### **obj_cursor**

#### **Create**

```gml

image_speed = 0

enum CURSOR_STATE {
	
	REGULAR,
	MAGNIFYING,
	RIGHT_ARROW,
	HAND,
	LEFT_ARROW	
}

cursor_state = CURSOR_STATE.REGULAR
```

#### **Step**

```gml
x = mouse_x
y = mouse_y

if cursor_state == CURSOR_STATE.LEFT_ARROW {
	image_index = 2
	image_xscale = -1
}
else {
	image_index = cursor_state
	image_xscale = 1
}

cursor_state = CURSOR_STATE.REGULAR

if cursor_state == CURSOR_STATE.REGULAR {
	global.command_text = ""	
}

```

### **obj_overlay**

#### **Create**

```gml
image_alpha = 0.9
depth = 0

if global.curtains_open {
	instance_destroy()	
}
```

### **obj_initializer**

#### **Create**

```gml
window_set_cursor(cr_none)

global.direction_arr = [Rm_Bedside,Rm_Curtains,Rm_Desk,Rm_Door]
global.direction_index = 0

global.inventory = []

global.curtains_open = false

global.safe_open = false

cursor = instance_create_layer(mouse_x,mouse_y,"Instances",obj_cursor)

global.data = {
	bedside: {
		painting: {
			cursor: CURSOR_STATE.MAGNIFYING,
			command_text: "Look At Painting",
			dark_text: "I can't see anything",
			description: "It's a modern art painting. Is it good? I can't tell."	
		},
		knobs: {
			cursor: CURSOR_STATE.HAND,
			command_text: "Use Knob",
			description: "I think they're just glued on for decoration."	
		}	
	},
	curtains: {
		closed: {
			cursor: CURSOR_STATE.HAND,
			command_text: "Open Curtains",	
		},
		window: {
			cursor: CURSOR_STATE.MAGNIFYING,
			command_text: "Look At Window",
			description: "Ah, the beauty of nature. Mountains as far as the eye can see. Majestic."		
		},
		note: {
			cursor: CURSOR_STATE.HAND,
			command_text: "Pick Up Note",		
		}	
	},
	desk: {
		partyhat: {
			cursor: CURSOR_STATE.MAGNIFYING,
			command_text: "Look At Hat",
			dark_text: "I can't see anything",
			description: "It's a party hat from my birthday last night. I don't even remember wearing it."	
		},
		mirror: {
			cursor: CURSOR_STATE.MAGNIFYING,
			command_text: "Look At Mirror",
			dark_text: "I can't see anything",
			description: "No way, I played Disco Elysium and never looked in a mirror again."
		},
		key: {
			cursor: CURSOR_STATE.HAND,
			command_text: "Pick Up Key",	
		}
	},
	door: {
		lock: {
			cursor: CURSOR_STATE.HAND,
			command_text: "Use Lock",
			description: "It looks like it takes a key."
		},
		calendar: {
			cursor: CURSOR_STATE.MAGNIFYING,
			command_text: "Look At Calendar",
			dark_text: "I can't see anything",
			description: "It's a calendar showing the date is October 23. It's off by two days."
		},
		peephole: {
			cursor: CURSOR_STATE.MAGNIFYING,
			command_text: "Look At Peephole",
			dark_text: "I can't see anything",
			description: "It's a hallway."	
		}
	},	
}
```

#### **Draw**

```gml
draw_set_font(fnt_title)
draw_set_color(c_white)

var sw = string_width("Esacpe")

draw_text((room_width-sw)/2,200,"Esacpe")

draw_set_font(fnt_header)
var ssw = string_width("Click to Start")
draw_text((room_width-ssw)/2,400,"Click to Start")
```

#### **Step**

```gml
if mouse_check_button_released(mb_left) {
	room_goto(Rm_Bedside)
}
```

### **obj_controller**

#### **Create**

```gml
global.bg_alpha = 0

global.command_text = ""
global.description_text = ""

global.menu_up = false

cursor = instance_create_layer(mouse_x,mouse_y,"Instances",obj_cursor)

if !global.curtains_open {
	instance_create_layer(0,0,"Instances",obj_overlay)	
}

description_timer = 100
description_timer_total = 100
```

#### **Step**

```gml
if global.bg_alpha < 1 {
	global.bg_alpha += 0.05
}

if global.description_text != "" {
	if description_timer > 0 {
		description_timer--	
	}
	else {
		description_timer = description_timer_total
		global.description_text = ""
	}
}

if instance_exists(obj_note_big) or instance_exists(obj_safeplate) {
	global.menu_up = true
	global.command_text = "Put Away Note"
}
else {
	global.menu_up = false	
}
```

### **obj_bg**

#### **Create**

```gml
depth = 10

image_speed = 0

if global.curtains_open and global.direction_index == 1 {
	image_index = 4
}
else if global.safe_open and global.direction_index == 2 {
	image_index = 5	
}
else {
	image_index = global.direction_index
}

image_alpha = global.bg_alpha
```

#### **Step**

```gml
image_alpha = global.bg_alpha
```

### **obj_bg_tester**

#### **Create**

```gml
image_alpha = 0

instance_destroy()
```

### **obj_text_command**

#### **Create**

```gml
depth = 5

selected = false

command_text = ""
```

#### **Step**

```gml
if global.command_text == "" {
	selected = false
	command_text = "Look At"
}
else {
	selected = true
	command_text = global.command_text
}
```

#### **Draw**

```gml
draw_set_font(fnt_header)

if selected {
	draw_set_color(c_white)
}
else {
	draw_set_color(c_gray)	
}

draw_text(x,y+10,command_text)

draw_set_color(c_white)
draw_text_ext(x,y+40,global.description_text,30,400)
```

### **obj_text_items**

#### **Create**

```gml
depth = 5
```

#### **Draw**

```gml
draw_set_font(fnt_header)
draw_set_color(c_white)
draw_text(x,y+10,"ITEMS")
```

### **obj_wall**

#### **Create**

```gml
image_alpha = 0
```

#### **Step**

```gml
if collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_cursor,0,0) and !global.menu_up {
	if x < room_width/2 {
		obj_cursor.cursor_state = CURSOR_STATE.LEFT_ARROW
		if mouse_check_button_released(mb_left) {
			if global.direction_index > 0 {
				global.direction_index--
				room_goto(global.direction_arr[global.direction_index])
			}
			else {
				global.direction_index = array_length(global.direction_arr)-1
				room_goto(global.direction_arr[global.direction_index])
			}
		}	
	}
	else {
		obj_cursor.cursor_state = CURSOR_STATE.RIGHT_ARROW	
		if mouse_check_button_released(mb_left) {
			if global.direction_index < array_length(global.direction_arr)-1 {
				global.direction_index++
				room_goto(global.direction_arr[global.direction_index])
			}
			else {
				global.direction_index = 0
				room_goto(global.direction_arr[global.direction_index])
			}
    }
	}
}
```

### **obj_painting**
*This code is the same for several objects, only the data is changed*

#### **Create**

```gml
image_alpha = 0

data = global.data.bedside.painting
```

#### **Step**

```gml
if collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_cursor,0,0) and !global.menu_up {

	obj_cursor.cursor_state	= data.cursor
	
	global.command_text = data.command_text
	
	if mouse_check_button_released(mb_left) {
		if global.curtains_open {
			global.description_text = data.description
		}
		else {
			global.description_text = data.dark_text
		}
	}
}
```
### **obj_partyhat**
Duplicate **obj_painting**, change data to:

```gml
data = global.data.desk.partyhat
```

### **obj_mirror**
Duplicate **obj_painting**, change data to:

```gml
data = global.data.desk.mirror
```

### **obj_calendar**
Duplicate **obj_painting**, change data to:

```gml
data = global.data.door.calendar
```

### **obj_knob**
Duplicate **obj_painting**, change Create Event to:

#### **Create**

```gml
image_alpha = 0

data = global.data.bedside.knobs

if !global.curtains_open {
	instance_destroy()	
}
```

### **obj_peephole**
Duplicate **obj_knob**, change data to:

```gml
data = global.data.door.peephole
```

### **obj_curtains**

#### **Create**

```gml
image_alpha = 0

data = global.data.curtains.closed

if global.curtains_open {
	data = global.data.curtains.window
}
else {
	data = global.data.curtains.closed	
}
```

#### **Step**

```gml
if collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_cursor,0,0) and !global.menu_up {

	obj_cursor.cursor_state	= data.cursor
	
	global.command_text = data.command_text
	
	if mouse_check_button_released(mb_left) {
		if !global.curtains_open {
			global.curtains_open = true
			room_restart()
		}
		else {
			global.description_text = data.description	
		}
	}
}
```

### **obj_note_big**

#### **Create**

```gml
depth = 1
```

#### **Step**

```gml
if mouse_check_button_released(mb_left) {
	instance_destroy()	
}

if instance_exists(obj_cursor) {
	obj_cursor.cursor_state = CURSOR_STATE.MAGNIFYING	
}
```

### **obj_item_room_note**

#### **Create**

```gml
depth = 1
```

#### **Step**

```gml
if collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_cursor,0,0) {

	obj_cursor.cursor_state	= data.cursor
	
	global.command_text = data.command_text
	
	if mouse_check_button_released(mb_left) {
		if !array_contains(global.inventory,"note") {
			array_push(global.inventory,"note")
			room_restart()
		}
	}
}
```

### **obj_item_menu_note**

#### **Create**

```gml
image_speed = 0
depth = 5

if array_contains(global.inventory,"note") {
	image_alpha = 1
}	
else {
	image_alpha = 0
	instance_destroy()
}
```

#### **Step**

```gml
if collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_cursor,0,0) {

	obj_cursor.cursor_state	= CURSOR_STATE.HAND
	
	global.command_text = "Read Note"
	
	if mouse_check_button_released(mb_left) {
		instance_create_layer(room_width/2,0,"Instances",obj_note_big)
	}	
}
```

### **obj_digits_arrows**

#### **Create**

```gml
depth = 2
```

### **obj_safe_lights**

#### **Create**

```gml
image_speed = 0
depth = 2

light_type = -1
```

#### **Step**

```gml
if light_type == 1 {
	image_index = 0	
}
else {
	image_index = 2	
}
```

### **obj_digits**

#### **Create**

```gml
depth = 2
image_speed = 0

num = 0

image_index = num

btn_upper = instance_create_layer(x,y-125,"Instances",obj_digits_arrows)

btn_lower = instance_create_layer(x,y+125,"Instances",obj_digits_arrows)
btn_lower.image_yscale = -1
```

#### **Step**

```gml
image_index = num

if collision_rectangle(btn_upper.bbox_left,btn_upper.bbox_top,btn_upper.bbox_right,btn_upper.bbox_bottom,obj_cursor,0,0) {
	obj_cursor.cursor_state = CURSOR_STATE.HAND
	global.command_text = "Increase Digit"
	
	if mouse_check_button_released(mb_left) {
		num++
		if num > 9 {
			num = 0	
		}
		else if num < 0 {
			num = 9
		}
	}
}

if collision_rectangle(btn_lower.bbox_left,btn_lower.bbox_top,btn_lower.bbox_right,btn_lower.bbox_bottom,obj_cursor,0,0) {
	obj_cursor.cursor_state = CURSOR_STATE.HAND
	global.command_text = "Decrease Digit"
	
	if mouse_check_button_released(mb_left) {
		num--
		if num > 9 {
			num = 0	
		}
		else if num < 0 {
			num = 9
		}
	}
}
```

### **obj_safeplate**

#### **Create**

```gml
depth = 4

digit1 = instance_create_layer(x+150,y+250,"Instances",obj_digits)
digit2 = instance_create_layer(x+275,y+250,"Instances",obj_digits)
digit3 = instance_create_layer(x+400,y+250,"Instances",obj_digits)
digit4 = instance_create_layer(x+525,y+250,"Instances",obj_digits)

light1 = instance_create_layer(x+500,y+55,"Instances",obj_safe_lights)
light1.light_type = 1

light2 = instance_create_layer(x+575,y+55,"Instances",obj_safe_lights)
light2.light_type = 1

solved_puzzle = false
safe_open_timer = 35
```

#### **Step**

```gml
if !collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_cursor,0,0) {
	if mouse_check_button_released(mb_left) {
		
		if instance_exists(obj_digits) {
			instance_destroy(obj_digits)	
		}
		if instance_exists(obj_digits_arrows) {
			instance_destroy(obj_digits_arrows)	
		}
		if instance_exists(obj_safe_lights) {
			instance_destroy(obj_safe_lights)	
		}
		instance_destroy()
	}
}

if instance_exists(obj_digits) {
	if digit1.num == 1
		and digit2.num == 0
		and digit3.num == 2
		and digit4.num == 4 {
			solved_puzzle = true	
	}
}
	
if solved_puzzle {
	light2.image_index = 3
	if safe_open_timer > 0 {
		safe_open_timer--	
	}
	else {
		global.safe_open = true
		room_restart()	
	}
}
```

### **obj_safe_in_room**

#### **Create**

```gml
image_alpha = 0

if !global.curtains_open or global.safe_open {
	instance_destroy()	
}
```

#### **Create**

```gml
if collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_cursor,0,0) and !global.menu_up {

	obj_cursor.cursor_state = CURSOR_STATE.HAND
	global.command_text = "Use Safe"
	
	if mouse_check_button_released(mb_left) {
		instance_create_layer((room_width-681)/2,100,"Instances",obj_safeplate)	
	}
	
}
```

### **obj_item_room_key**

#### **Create**

```gml
image_speed = 0
image_index = 1

depth = 3

if !global.safe_open or array_contains(global.inventory,"key") {
	image_alpha = 0
	instance_destroy()
}
else {
	image_alpha = 1	
}

data = global.data.desk.key
```

#### **Step**

```gml
if collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_cursor,0,0) {

	obj_cursor.cursor_state	= data.cursor
	
	global.command_text = data.command_text
	
	if mouse_check_button_released(mb_left) {
		if !array_contains(global.inventory,"key") {
			array_push(global.inventory,"key")
			room_restart()
		}
	}
}
```

### **obj_item_menu_key**

#### **Create**

```gml
image_speed = 0
depth = 5

image_index = 1

if array_contains(global.inventory,"key") {
	image_alpha = 1
}	
else {
	image_alpha = 0
	instance_destroy()
}
```

#### **Step**

```gml
if collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_cursor,0,0) {

	obj_cursor.cursor_state	= CURSOR_STATE.HAND
	
	global.command_text = "Use Key"
	
	if mouse_check_button_released(mb_left) {
		global.description_text = "Get me to a door!"
	}
}
```

### **obj_lock**

#### **Create**

```gml
image_alpha = 0

data = global.data.bedside.knobs

if !global.curtains_open {
	instance_destroy()	
}
```

#### **Step**

```gml
if collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_cursor,0,0) and !global.menu_up {

	obj_cursor.cursor_state	= data.cursor
	
	global.command_text = data.command_text
	
	if mouse_check_button_released(mb_left) {
		if global.curtains_open {
			if array_contains(global.inventory,"key") {
				room_goto(Rm_End)
			}
			else {
				global.description_text = data.description
			}
		}
		else {
			global.description_text = data.dark_text
		}
	}
}
```

### **obj_text_end**
*(I made this but didn't end up using it in the tutorial, I forgot to take obj_initializer out of Rm_End and add this object, but I liked the way that looked better.)*

#### **Draw**

```gml
draw_set_font(fnt_title)
draw_set_color(c_black)
var title_length = string_width("Congratulations!")
draw_text((room_width-title_length)/2,100,"Congratulations!")
```

