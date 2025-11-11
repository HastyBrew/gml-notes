# Matching Game

## **ROOM**
Size: 1150x1150
Color: Pastel Blue

## **FONTS**

- **fnt_title:** Jersey 20, size 30
- **fnt_text:** Jersey 20, size 30

## **SPRITES**

### **spr_card_back**
Size: 200x300
1. Dark Purple BG, Light Blue Border
2. Duplicate for **spr_square_icons**
3. Create diamond pattern over the border


### **spr_card_icon**
Take Border, Make SIX More Frames<br />
Make:
- Blank
- Green Square
- Yellow Circle
- Purple Triangle
- Orange Smiley Face
- White Lightning
- Red Star

## **OBJECTS**

### **obj_controller_card**

#### **Create**

```gml
global.flipped_cards = 0

global.game_won = false


global.chosen_arr = []

global.delay_on = false

global.delay_timer = 50
global.delay_timer_total = 50

card_width = 200
card_height = 300
buffer = 50

columns = 0
rows = 0

x_offset = 100 + (card_width/2)
y_offset = 50 + (card_height/2)

solved_amount = 0

instance_arr = [noone]

card_arr = []

for (var i = 1;i<7;i++) {
	array_push(card_arr,i)
	array_push(card_arr,i)	
}

card_arr = array_shuffle(card_arr)

for (var i = 1; i < 13; i++) {
	instance_arr[i] = instance_create_layer(x_offset+(buffer*rows)+(card_width*rows),y_offset+(buffer*columns)+(card_height*columns),"Instances",obj_card)
	instance_arr[i].card_num = i
	instance_arr[i].icon_num = card_arr[i-1]
	
	rows++
	
	if rows > 3 {
		columns++
		rows = 0
	}
}

```
#### **Step**

```gml
if array_length(global.chosen_arr) >= 2 and global.delay_timer > 0 {
	
	global.delay_on = true
	global.delay_timer--
	

	
}
else if array_length(global.chosen_arr) >= 2 and global.delay_timer <= 0 {

	if instance_arr[global.chosen_arr[0]].icon_num == instance_arr[global.chosen_arr[1]].icon_num {
	
		instance_arr[global.chosen_arr[0]].card_state = CARD_STATE.SOLVED
		instance_arr[global.chosen_arr[1]].card_state = CARD_STATE.SOLVED
		solved_amount += 2
		global.delay_on = false
		global.delay_timer = global.delay_timer_total
		global.chosen_arr = []
		
		
	}
	else {
		instance_arr[global.chosen_arr[0]].card_state = CARD_STATE.FLIPPING_BACK
		instance_arr[global.chosen_arr[1]].card_state = CARD_STATE.FLIPPING_BACK
		global.delay_on = false
		global.delay_timer = global.delay_timer_total
		global.chosen_arr = []
	}
	
}

if solved_amount >= array_length(instance_arr)-1 {
	global.game_won = true
}
```

### **obj_card**

#### **Create**

```gml
image_speed = 0

sprite_index = spr_card_back


enum CARD_STATE {
	
	NULL,
	HIDDEN,
	FLIPPING,
	FLIPPED,
	FLIPPING_BACK,
	SOLVED,
	ALL_FLIP
	
}

card_state = CARD_STATE.HIDDEN

card_num = -1
icon_num = -1



Left Released

if card_state = 1 and !global.delay_on {
	card_state = 2
	array_push(global.chosen_arr,card_num)
}
```

#### **Step**

```gml
switch(card_state) {
	
	case CARD_STATE.HIDDEN:
	
		image_xscale = 1
		sprite_index = spr_card_back
		image_index = 0
	
		break;
		
		
	case CARD_STATE.FLIPPED:
		
		image_xscale = 1
		sprite_index = spr_card_icons
		image_index = icon_num

	
		break;
	
	case CARD_STATE.FLIPPING:
	
		if image_xscale > 0 {
			image_xscale -= 0.15
		}
		else {
			card_state = CARD_STATE.FLIPPED	
		}
	
		
		break;
		
	case CARD_STATE.FLIPPING_BACK:
	 
		if image_xscale > 0 {
			image_xscale -= 0.15
		}
		else {
			if !global.game_won {
				card_state = CARD_STATE.HIDDEN
			}
			else {
				room_restart()
			}
		}
	
		
		break;
}

if global.game_won = true {
	card_state = CARD_STATE.FLIPPING_BACK	
}
```
