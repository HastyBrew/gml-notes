# Simple Branching Dialogue Paths Notes

## **ROOMS**

**Size:** 800x1300

- **Rm_Main**<br />
Contains: obj_controller,obj_bg
- **Rm_Shop**<br />
Contains: obj_obj,obj_shop_title

## **FONTS**

- **fnt_title:** Jersey 20, 60
- **fnt_header:** Jersey 20, 30

## **SPRITES**

- **spr_shopkeep_strip2**: *imported from Ko-Fi*
- **spr_placeholder**: 50x50, Magenta
- **spr_brow**: 60x20, Black, Orientation Center
- **spr_cursor**: 30x30, small right-facing triangle

## **OBJECTS**

### **obj_bg**

#### **Create**

```gml
depth = 8

image_alpha = 0

image_speed = 0

if room == Rm_Main {
	image_index = 0	
}
else {
	image_index = 1	
}
```

#### **Step**

```gml
if (image_alpha < 1) image_alpha += 0.05
```

### **obj_cursor**

#### **Create**

```gml
depth = 2
```

### **obj_choice**

#### **Create**

```gml
depth = 2

text_string = ""

Draw

draw_set_font(fnt_text)
draw_set_color(c_white)
draw_text(x,y,text_string)
```

### **obj_brows**

#### **Create**

```gml
depth = 2
```

### **obj_controller**

#### **Create**

```gml
enum CONVO_STATE {
	
		STARTING,
		SETTING,
		SET,
		CHOOSING
}


convo_state = CONVO_STATE.STARTING

enum BROW_STATE {

	REGULAR,
	SURPRISED,
	MAD
	
}

brow_state = BROW_STATE.REGULAR


chosen_choice = 0

top_text = ""
choices = []
current_choices = []
current_label = noone
new_label = noone


title_text = "The Shopkeep"
dialogue_text = ""
choice1_text = ""
choice2_text = ""

brow_y = 147

brow1 = instance_create_layer(355,brow_y,"Instances",obj_brows)
brow1.image_angle = 13

brow2 = instance_create_layer(452,brow_y,"Instances",obj_brows)
brow2.image_angle = -20

convos = {
	
	shopkeep: {
	
		welcome: {
			top_text: "Welcome to my Shop! Haven't seen you 'round these parts before ay buckaroo.",
			choices: ["Nice to see you"],
			brow_state: BROW_STATE.REGULAR,
			next_action: noone
			
		},
		
		"Nice to see you": {
			top_text: "What can I do ya fer?",
			choices: ["What's for sale?", "Got any gossip?"],
			brow_state: BROW_STATE.REGULAR,
			next_action: noone
			
			
		},
		
		"What's for sale?": {
			top_text: "Let's have a look",
			choices: ["Look at stuff"],
			brow_state: BROW_STATE.REGULAR,
			next_action: Rm_Shop
			
			
		},
		
		"Got any gossip?": {
			top_text: "Oh there's plenty goin round round these parts. Ol' Brhynjoff got stuck in a minecraft again.",
			choices: ["Minecraft? Like the game?", "Those names sound familiar"],
			brow_state: BROW_STATE.REGULAR,
			next_action: noone
			
			
		},
		
		"Those names sound familiar": {
			top_text: "Well I'm sure they do, sonny, what with them being references for the sake of humor and the like.",
			choices: ["Got any REAL gossip?", "See you later"],
			brow_state: BROW_STATE.MAD,
			next_action: noone
			
			
		},
		
		"Got any REAL gossip?": {
			top_text: "If you're looking for a real game experience I'm 'fraid I can't help you. Just this screen.",
			choices: ["See you later"],
			brow_state: BROW_STATE.SURPRISED,
			next_action: noone
			
			
		},
		
		"Minecraft? Like the game?": {
			top_text: "No, like the old craft mine out back. Brhynjoff got caught digging down and ran into a Creeper",
			choices: ["OK, now I'm even more confused", "See you later"],
			brow_state: BROW_STATE.MAD,
			next_action: noone
			
			
		},
		
		"OK, now I'm even more confused": {
			top_text: "He was playing with a blackjack, but a reverse one. A jack black, if you will",
			choices: ["Can I go now?", "See you later"],
			brow_state: BROW_STATE.SURPRISED,
			next_action: noone
			
			
		},
		
		"Can I go now?": {
			top_text: "No. This is the only screen in this game",
			choices: ["See you later"],
			brow_state: BROW_STATE.REGULAR,
			next_action: noone
			
			
		},
		
		"See you later": {
			top_text: "Well alright then",
			choices: ["Nice to meet you"],
			brow_state: BROW_STATE.REGULAR,
			next_action: Rm_Main
			
			
		}
	}
}
```

#### **Step**

```gml
// CURSOR CREATION

if instance_exists(obj_choice) {
	if !instance_exists(obj_cursor) {
		instance_create_layer(obj_choice.x-30,obj_choice.y,"Instances",obj_cursor)	
	}
}
else {

	if instance_exists(obj_cursor) {
		instance_destroy(obj_cursor)	
	}
		
}

if instance_exists(obj_cursor) and array_length(current_choices) > 1 {
	if chosen_choice == 0 {
		obj_cursor.y = current_choices[0].y
	}
	else {
		obj_cursor.y = current_choices[1].y	
	}
}

// TEXT STUFF


switch(convo_state) {
	
	
	case CONVO_STATE.STARTING:
	
	
		new_label = convos.shopkeep.welcome
		
		convo_state = CONVO_STATE.SETTING
	
		break;
		
	
	case CONVO_STATE.CHOOSING:
	
	
	
		if keyboard_check_released(vk_up) or keyboard_check_released(vk_down) {
			if array_length(choices) > 1 {
				if chosen_choice == 0 {
					chosen_choice = 1	
				}
				else {
					chosen_choice = 0
				}
			}	
		}
		
		if keyboard_check_released(ord("1")) {
			var label = choices[chosen_choice]
			chosen_choice = 0
			new_label = convos.shopkeep[$ label]
			convo_state = CONVO_STATE.SETTING	
		}
		
		if keyboard_check_released(ord("2")) and array_length(choices)>1 {
			var label = choices[chosen_choice]
			chosen_choice = 1
			new_label = convos.shopkeep[$ choices[chosen_choice]]
			convo_state = CONVO_STATE.SETTING	
		}
		
		if keyboard_check_released(vk_enter) {
			var label = choices[chosen_choice]
			new_label = convos.shopkeep[$ choices[chosen_choice]]
			convo_state = CONVO_STATE.SETTING
		}

		
	
		break;
		
	case CONVO_STATE.SETTING:
	
	if new_label.next_action != noone {
		room_goto(new_label.next_action)
	}
	
		current_label = new_label
	
		dialogue_text = current_label.top_text
		
		choices = current_label.choices
		
		brow_state = current_label.brow_state
	
	
		if instance_exists(obj_choice) {
			instance_destroy(obj_choice)	
		}
		
		for (var i = 0;i<array_length(choices);i++) {
			current_choices[i] = instance_create_layer(100,1050+(i*50),"Instances",obj_choice)
			current_choices[i].text_string = choices[i]
		}
		
		convo_state = CONVO_STATE.CHOOSING
		
	
		break;


}


switch(brow_state) {
	
	case BROW_STATE.REGULAR:
	
		if brow1.y > brow_y {
			brow1.y--	
		}
		else if brow1.y < brow_y {
			brow1.y++	
		}
		
		if brow2.y > brow_y {
			brow2.y--	
		}
		else if brow2.y < brow_y {
			brow2.y++	
		}
		
		break;
		
		
	case BROW_STATE.SURPRISED:
	
	
		if brow1.y > 140 {
			brow1.y--	
		}
		
		if brow2.y > 140 {
			brow2.y--	
		}
		
		break;
		
	case BROW_STATE.MAD:
	
	
		if brow1.y < 155 {
			brow1.y++
		}
		
		if brow2.y < 155 {
			brow2.y++
		}
		
		break;
	
	  
}
```

#### **Step**

```gml
draw_set_color(c_white)


draw_set_font(fnt_title)
draw_text(100,810,title_text)

draw_set_font(fnt_text)
draw_text_ext(100,900,dialogue_text,35,600)
```

### **obj_shop_title**

#### **Step**

```gml
if keyboard_check_released(vk_enter) {

	room_goto(Rm_Main)	
	
}
```

#### **Draw**

```gml
draw_set_font(fnt_title)
draw_set_color(c_white)
draw_text(x,y,"The Shop")
```

