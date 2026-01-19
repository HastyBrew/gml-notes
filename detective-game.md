
# **Navigation**
1. [Project Setup](#project-setup)
2. [Loading Game Data with Structs and Arrays](#loading-game-data-with-structs-and-arrays)
3. [Animating with Enums](#animating-with-enums)
4. [Dynamic, Draggable UI](#dynamic-ui)
5. [Super Easy Dialogue with Structs](#super-easy-dialogue-with-structs)
6. [Next Steps](#next-steps)
 
## **Intro**

OK here we go! This is a detective game but it's more similar in core mechanics to games like "Papers, Please" or "No, I'm Not a Human" or "That's Not My Neighbor", a lot of "Nots" in there, huh? <br /><br />
But good gamedev means we can be inspired by these core mechanics without ripping them off, just like they all did to each other. So this game is a Detective "spot the doppelganger" game, where you have to individually interrogate the suspects.<br /><br />
I tried to leave a lot of room for customization. You can add new crime stories, control "guilty" or "innocent" through narrative, or get creative and combine this game and assets with the interrogation game.
Assets are made by me and provided under CC0, they're free to use with no restrictions whatsoever. Except one, you gotta promise me you'll have fun doing gamedev.<br /><br />
Finally, I hope this can help clear up some common pitfalls in GM, namely draggable, selectable UI. I've created a method that works for me by assigning instances depth based on a position in an array.
Arrays are just really snappy in GameMaker. We'll also use global flags to control more UI behavior, enums for simple animation, and a combo of arrays, structs, and globals for simple, simple dialogue.<br /><br />
This was inspired through ideas from subscribed members on Ko-Fi and Discord. Thank you so incredibly much for your continued support, you make it easier for me to keep making cool things so you can make cool things.
And thank you in general to the GameMaker community for supporting this weird stuff.<br /><br />
If you have questions or need help expanding this into your own game, feel free to reach out on Discord and we can figure it out together.<br /><br />

# **Project Setup**

Open up GameMaker, give it a project name, and click **Start**

## **Rooms**
- **Rm_Main:** 1200x800
- **Rm_End:** 1200x800

## **Fonts**
Fonts come from Google Fonts
- **fnt_h1:** Roboto Mono, size 24
- **fnt_h1:** Roboto Mono, size 18
- **fnt_num:** Montserrat Black, size 40
- **fnt_text:** Roboto Mono, size 14

## **Sprites**

### **Ko-Fi**
You can download these sprites from Ko-Fi, make sure to leave the name as they are in Ko-Fi, the _strip convention puts them into frames.<br ><br />
Download:

- spr_bg_strip2
- spr_bodies_strip5
- spr_deskobjects_strip9
- spr_eyes_strip10
- spr_heads_strip5
- spr_mouths_strip20
- spr_photos_strip10
- spr_uibg_strip3

### GameMaker

Drag the assets into the sprites folder in GameMaker. They should automatically go into frames. <br /><br />
Normally, to be nice to our memory we want to group our assets into as many sprite sheets as possible, like we've done here. Ideally we would set these as objects to the correct frame and spawn them from a controller object, leaving our Rooms nice and tidy, but since this is an exercise, we'll want to break them up a little bit so they'll be easier to work with.<br /><br />
In GameMaker, only the first frame of the sprite will show on the object in our Room. So we simply want to make a few more sprites out of these sheets. We'll duplicate sprites and delete frames so that we're left with only the frames we need. For example:
- **spr_bg**<br /><br />
Duplicate it, and split it into:<br /><br />
- spr_bg (frame 0)
- spr_bg_overlay (frame 1)

To keep things clean, we'll rename the following sprites:

- **spr_deskobjects** to **spr_deskobjects_master**
- **spr_uibg** to **spr_uibg_master**

#### spr_deskobjects_master
 - spr_phone (frames 0-2)
 - spr_speaker (frames 3,4)
 - spr_clipboard (frame 5)
 - spr_file (frame 6)
 - spr_coffee (frame 7)
 - spr_badge (frame 8)

#### spr_uibg_master
- spr_ui_file
- spr_ui_clipboard
- spr_ui_badge<br />

Let's go over our sprites and **set everything but spr_bg and spr_bg_overlay to Center Origin** <br /><br />

**We should have set to Center Origin:**

- spr_badge
- spr_bodies
- spr_clipboard
- spr_coffee
- spr_eyes
- spr_file
- spr_heads
- spr_mouths
- spr_phone
- spr_photos
- spr_speaker
- spr_ui_badge
- spr_ui_clipboard
- spr_ui_file

**We should have set to Top Left Origin**

- spr_bg
- spr_bg_overlay

In addition, we'll need to make a few sprites in GameMaker:

- **spr_overlay_black:** 1200x800, Black
- **spr_dialogue_box:** 1100x200, Black
- **spr_placeholder:** 100x50, Magenta 


# **Loading Game Data with Structs and Arrays**

### Explanation

To set up our game, we'll create a couple structs. One holds meta game mechanics, like the tutorial, while the other contains all of the data for the NPCs. <br /><br />
Since this is a "doppelganger" game where you have to find incorrect information, we'll create a struct with the following structure:
```text
> 1st NPC Name

    > Clipboard Info Matches Dialogue (Innocent)

    > Incorrect Clipboard Information (Guilty)

    > Dialogue Info Matches Clipboard (Innocent)

    > Incorect Dialogue Information (Guilty)

> 2nd NPC Name

    > Etc.
```
And then create **parallel arrays** of our game data, randomized per run, identified by an NPC index. This gives us the greatest flexibilty over the data, NPCs can be chosen for dialogue or the clipboard can be flipped through based on their own indices, but the data stays the same. <br /><br />
But before that, let's set up our Room and initialize a few objects so we don't error.

### obj_overlay_black
**Sprite:** spr_overlay_black

#### Create

```gml
image_alpha = 0

fading_out = false

Step

if fading_out {
	if image_alpha < 1 {
		image_alpha += 0.05	
	}
	else {
		room_goto(Rm_End)	
	}
}
else {
	if image_alpha > 0 {
		image_alpha -= 0.05	
	}
	else {
		instance_destroy()	
	}
}
```

### obj_bg
**Sprite:** spr_bg

#### Create

```gml
depth = 16

Step

if global.selecting_perp or global.final_decision {
	image_blend = c_gray	
}
else {
	image_blend = -1	
}
```

### obj_bg_overlay
**Sprite:** spr_bg_overlay

#### Create

```gml
depth = 12
```
### obj_controller
*No Sprite*<br />

Drop obj_controller in Room, then obj_bg and obj_bg_overlay

### obj_body
**Sprite:** spr_bodies

Make this but leave it for now, we'll come back to it.

### obj_controller (again)

#### Create
```gml
randomize()

overlay = instance_create_layer(0,0,"Instances",obj_overlay_black)
overlay.image_alpha = 1

global.game_info = {
	 
	phone_intro: ["Commandant: Welcome, detective",
				"A pawn shop was robbed this morning. There were five suspects located in the store at the time.",
				"We're not sure if there are one or more perpetrators, you'll have to check them individually.",
				"Also, we think the criminals are working with accomplices, body doubles that look just like them.",
				"You'll have to check all information and see if their story matches. If not, call it in.",
				"Good luck, Detective."
				],
	phone_callback: [],	
	file: {
		
		title: "Gunderson's Safe",
		description: "Mr. Gunderson was working at his pawn shop when around 9:30a.m. he noticed his safe was empty. There were five people present in the store at the time."
	}
	
}

global.npcs = {
	
	barry: {
		data: {
			npc_id: 0,
			head_adjust: [0,-169],
			eyes_adjust: [3,-179],
			mouth_adjust : [0,-120],
			draw_adjust:  [-10,-48],
		},
		inno_info: {
			photo_index: 0,
			npc_name: "Barry Oldman",
			profession: "Business",
			description: "Claims to have been looking for a present for his wife."
			
		},
		guilty_info: {
			photo_index: 0,
			npc_name: "Barry Oldman",
			profession: "Business",
			description: "Claims to have been looking for a present for his wife."
			
		},
		inno_dialogue: {
			npc_name: "Uh, my name? It's uh... Barry Oldman",
			what_doing: "I was looking for a gift for my wife."
			
			
		},
		guilty_dialogue: {
			npc_name: "Uh, my name? It's uh... Larry Oldman",
			what_doing: "I was looking for a gift for my wife."
			
		}
	},
	
	dennis: {
		data: {
			npc_id: 1,
			head_adjust: [0,-68],
			eyes_adjust: [2,-80],
			mouth_adjust : [0,-25],
			draw_adjust:  [-18,30]
		},
		inno_info: {
			photo_index: 1,
			npc_name: "Dennis Belvitas",
			profession: "Funemployed",
			description: "Says he was in the pawn shop to sell his WWII Memorabilia."
			
		},
		guilty_info: {
			photo_index: 1,
			npc_name: "Dennis Belvitas",
			profession: "Funemployed",
			description: "Says he was in the pawn shop to sell his WWII Memorabilia."
			
		},
		inno_dialogue: {
			npc_name: "My name is Dennis!",
			what_doing: "I was selling my war trophies"
			
			
		},
		guilty_dialogue: {
			npc_name: "My name is Dennis!",
			what_doing: "I was selling my collection of Warhammer 30k collectibles"
			
		}
	},
	
	rita: {
		data: {
			npc_id: 2,
			head_adjust: [0,-112],
			eyes_adjust: [0,-110],
			mouth_adjust : [-1,-53],
			draw_adjust:  [-16,0]
		},
		inno_info: {
			photo_index: 2,
			npc_name: "Rita Hudson",
			profession: "Entrepeneur",
			description: "Says she was in the shop to reclaim her pearl earrings."
			
		},
		guilty_info: {
			photo_index: 2,
			npc_name: "Rita Hudson",
			profession: "Entrepeneur",
			description: "Says she was in the shop to reclaim her pearl earrings."
			
		},
		inno_dialogue: {
			npc_name: "It's Rita. Rita Hudson.",
			what_doing: "None of your business! But I was selling some things."
			
			
		},
		guilty_dialogue: {
			npc_name: "It's Rita. Rita Hudson.",
			what_doing: "I was um... looking for a gift."
		}
	},
	
	chris: {
		data: {
			npc_id: 3,
			head_adjust: [0,-200],
			eyes_adjust: [0,-211],
			mouth_adjust : [-1,-138],
			draw_adjust:  [-20,-23]
		},
		inno_info: {
			photo_index: 3,
			npc_name: "Chris Grover",
			profession: "Kiosk Worker",
			description: "Works in the mall next door, frequents the pawn shop on his lunch break."
			
		},
		guilty_info: {
			photo_index: 9,
			npc_name: "Chris Grover",
			profession: "Kiosk Worker",
			description: "Works in the mall next door, frequents the pawn shop on his lunch break."
			
		},
		inno_dialogue: {
			npc_name: "I'm Chris Grover. Nice to meet you!",
			what_doing: "I was just browsing."
			
			
		},
		guilty_dialogue: {
			npc_name: "I'm Chris Grover. Nice to meet you!",
			what_doing: "I was just browsing."
			
		}
	},
	
	buddy: {
		data: {
			npc_id: 4,
			head_adjust: [6,-112],
			eyes_adjust: [4,-120], 
			mouth_adjust : [8,-68],
			draw_adjust:  [-12,10]
		},
		inno_info: {
			photo_index: 4,
			npc_name: "Buddy Hudson",
			profession: "Driver",
			description: "Says he frequents the shop because he likes the smell"
			
		},
		guilty_info: {
			photo_index: 4,
			npc_name: "Buddy Hudson",
			profession: "Driver",
			description: "Says he frequents the shop because he likes the smell"
			
		},
		inno_dialogue: {
			npc_name: "My name is Buddy, buddy",
			what_doing: "Just looking around, you know?"
			
			
		},
		guilty_dialogue: {
			npc_name: "I'm Chris Grover. Nice to meet you!",
			what_doing: "Just looking around, you know?"
			
		}
	}
	
	
}

npc_create_arr = [global.npcs.barry,global.npcs.dennis,global.npcs.rita,global.npcs.chris,global.npcs.buddy]
npc_create_arr = array_shuffle(npc_create_arr)

global.clipboard_arr = []
global.dialogue_arr = []
global.selected_dialogue = {}
global.selected_id = noone
global.correct_arr = []

npc_id_arr = []
npc_pos_margin = 240
npc_buffer = 175

for (var i = 0;i<array_length(npc_create_arr);i++) {
	
	var final_position = npc_pos_margin + (npc_buffer*i)
	var start_position = 0
	if i > 2{
		start_position = final_position + 600
	}
	else {
		start_position = final_position - 600
	}

	
	npc_id_arr[i] = instance_create_layer(start_position,466,"Instances",obj_body)
	npc_id_arr[i].npc = npc_create_arr[i]
	npc_id_arr[i].npc_index = i
	npc_id_arr[i].final_x = final_position
	
	var guilty = choose(true,false)
	if guilty {
		global.correct_arr[i] = true
		global.clipboard_arr[i] = npc_create_arr[i].guilty_info
		global.dialogue_arr[i] = npc_create_arr[i].guilty_dialogue
	}
	else {
		global.correct_arr[i] = false
		global.clipboard_arr[i] = npc_create_arr[i].inno_info
		global.dialogue_arr[i] = npc_create_arr[i].inno_dialogue
	}
	
}
```
We initialize the object instances for the NPCs (obj_body) and store the ID in the same **for** loop we use to create the arrays that store the guilty/innocent data. Everything runs through **i** so we know it's all loosely tied together.
<br /><br />
# Animating with Enums

### Explanation

In this section we'll use both enums and just plain old boolean flags to control some simple animation. Since our sprite sheets contain frames for multiple characters, we'll want to control the specific frames manually.<br /><br />
We'll use the "Farmer in the Dell" approach to building out our NPCs, or in other words we'll build out the objects like eyes and mouth first, then build the container object that holds them, obj_body, last.

### obj_eyes
**Sprite:** spr_eyes

#### Create

```gml
depth = 12

image_speed = 0
image_index = 0

blink_timer = random_range(100,300)

blink_frames = 10
blink_frames_total = 10

initial_frame = 0
blink_frame = initial_frame + 1
```

#### Step

```gml
blink_frame = initial_frame + 1

if blink_timer > 0 {
	image_index = initial_frame
	blink_timer--	
}
else {
	image_index = blink_frame
	
	if blink_frames > 0 {
		blink_frames--	
	}
	else { 
		blink_frames = blink_frames_total
		blink_timer = random_range(100,300)
	}
}
```

### obj_mouth
**Sprite:** spr_mouth

#### Create

```gml
depth = 12

image_speed = 0

initial_frame = 0
frame = 0
final_frame = initial_frame + 3
frame_speed = 0.5

is_closing = false

enum MOUTH_STATE {
	
	STILL,
	FLAPPING
	
}

mouth_state = MOUTH_STATE.STILL
```
#### Step

```gml
final_frame = initial_frame + 3
image_index = floor(frame)

switch(mouth_state) {
	
	case MOUTH_STATE.STILL:
	
		if frame > initial_frame {
			frame -= frame_speed	
		}
		else {
			frame = initial_frame	
		}
		break;
		
		
	case MOUTH_STATE.FLAPPING:
	
		if !is_closing {
			if frame < final_frame {
				frame += frame_speed
			}
			else {
				is_closing = true
			}
		}
		else {
			if frame > initial_frame {
				frame -= frame_speed	
			}
			else {
				is_closing = false	
			}
		}
		break;
}
```

### obj_head
**Sprite:** spr_heads

#### Create

```gml
depth = 12

image_speed = 0
```

Before we move on to obj_body, our container object, let's go back to **obj_controller** and add a few globals so we won't have to return to it for a while

### obj_controller (again)

#### Create

```gml
global.can_press = false
global.dialogue_open = false

global.selecting_perp = false
global.final_decision = false
global.decision_num = -1

global.ringing = true
global.tutorial_finished = false
```
*Note: if you're trying to test after creating obj_body, you'll want to flip the last two booleans*

### obj_body
**Sprite:** spr_bodies

#### Create

```gml
depth = 15

image_speed = 0

npc = {}

npc_id = -1
npc_index = -1
selected = false

start_x = -1000
final_x = 0

final_y = 466
move_speed = 5

head = noone
mouth = noone
eyes = noone

bouncing_up = false
bounce_y = final_y + 15

bounce_delay = irandom(10)

head_adjust = []
eyes_adjust = []
mouth_adjust = []
draw_adjust = [0,0]

talk_timer = 60
talk_timer_total = 60

enum NPC_STATE {
	
	NULL,
	INITIALIZE,
	CREATE_FACE,
	WAITING,
	WALKING,
	STANDING,
	SELECTING,
	TALKING,
	
}

npc_state = NPC_STATE.INITIALIZE
```

#### Step

```gml
switch(npc_state) {
	
	case NPC_STATE.INITIALIZE:
	
		if npc != {} {
			head_adjust = npc.data.head_adjust
			eyes_adjust = npc.data.eyes_adjust
			mouth_adjust = npc.data.mouth_adjust
			npc_id = npc.data.npc_id 
			draw_adjust = npc.data.draw_adjust
			image_index = npc.data.npc_id
			
			if head_adjust != [] and eyes_adjust != [] and mouth_adjust != [] and npc_id > -1 {
				npc_state = NPC_STATE.CREATE_FACE	
			}
			
		}
	
		break;
		
	case NPC_STATE.CREATE_FACE:
	
		if !instance_exists(head) {
			head = instance_create_layer(x+head_adjust[0],y+head_adjust[1],"Instances",obj_head)
			head.image_index = npc_id
		}
		if !instance_exists(eyes) {
			eyes = instance_create_layer(x+eyes_adjust[0],y+eyes_adjust[1],"Instances",obj_eyes)
			eyes.initial_frame = npc_id * 2
		}
		if !instance_exists(mouth) {
			mouth = instance_create_layer(x+mouth_adjust[0],y+mouth_adjust[1],"Instances",obj_mouth)
			mouth.initial_frame = npc_id * 4
		}
		
		if instance_exists(head) and instance_exists(eyes) and instance_exists(mouth) {
			npc_state = NPC_STATE.WAITING
		}
	
		break;
		
	case NPC_STATE.WAITING:
	
		if global.tutorial_finished {
			npc_state = NPC_STATE.WALKING	
		}
	
		break;
		
	case NPC_STATE.WALKING:
	
		if bounce_delay > 0 {
			bounce_delay--	
		}
		else {
			if bouncing_up {
				if y > final_y {
					y -= move_speed	/ 3
				}
				else {
					bouncing_up = false	
				}
			}
			else {
				if y < bounce_y {
					y += move_speed	/ 3
				}
				else {
					bouncing_up = true	
				}
			}
		}
		
		if x > final_x {
			x -= move_speed
		}
		else if x < final_x {
			x += move_speed	
		}
		else {
			npc_state = NPC_STATE.STANDING	
		}
		
		head.x = x + head_adjust[0]
		head.y = y + head_adjust[1]
		
		mouth.x = x + mouth_adjust[0]
		mouth.y = y + mouth_adjust[1]
		
		eyes.x = x + eyes_adjust[0]
		eyes.y = y + eyes_adjust[1]
	
		break;
		
		
	case NPC_STATE.STANDING:
	
		if global.final_decision and global.decision_num != npc_index {
			image_blend = c_gray
			head.image_blend = c_gray
			mouth.image_blend = c_gray
			eyes.image_blend = c_gray
		}
		else {
			image_blend	= -1
			head.image_blend = -1
			mouth.image_blend = -1
			eyes.image_blend = -1
		}
		
		talk_timer = talk_timer_total
		
		if y > final_y {
			y -= move_speed / 3
		}
		
		head.x = x + head_adjust[0]
		head.y = y + head_adjust[1]
		
		mouth.x = x + mouth_adjust[0]
		mouth.y = y + mouth_adjust[1]
		
		eyes.x = x + eyes_adjust[0]
		eyes.y = y + eyes_adjust[1]
	
		break;


	case NPC_STATE.TALKING:
	
		if talk_timer > 0 {
			mouth.mouth_state = MOUTH_STATE.FLAPPING
			talk_timer--
		}
		else {
			mouth.mouth_state = MOUTH_STATE.STILL
			npc_state = NPC_STATE.STANDING
		}
		image_blend	= -1
		head.image_blend = -1
		mouth.image_blend = -1
		eyes.image_blend = -1
		break;


	case NPC_STATE.SELECTING:
	
		if point_in_rectangle(mouse_x,mouse_y,bbox_left+50,bbox_top-100,bbox_right-50,bbox_bottom-50) {
			image_blend	= -1
			head.image_blend = -1
			mouth.image_blend = -1
			eyes.image_blend = -1
			
			if mouse_check_button_pressed(mb_left) {
				selected = true
				global.selected_dialogue = global.dialogue_arr[npc_index]
				var dgbox = instance_create_layer(x,y,"Instances",obj_dialogue_box)
				dgbox.npc_num = npc_index + 1
				dgbox.dialogue_state = DIALOGUE_STATE.PERP_ASK
				global.selected_id = id
				global.selecting_perp = false
				
			}
			
		}
		else {
			image_blend = c_gray
			head.image_blend = c_gray
			mouth.image_blend = c_gray
			eyes.image_blend = c_gray
		}
	
		break;
	
}

if global.selecting_perp and npc_state == NPC_STATE.STANDING {
	npc_state = NPC_STATE.SELECTING
}
else if !global.selecting_perp and npc_state == NPC_STATE.SELECTING {
	npc_state = NPC_STATE.STANDING	
}
```

#### Draw

```gml
draw_self()

draw_set_font(fnt_num)
draw_set_color(c_gray)

var order_num = npc_index + 1
draw_text(x+draw_adjust[0],y+draw_adjust[1],string(order_num))
```
<br /><br />
# Dynamic UI

The general idea behind this is pretty simple, since this is a detective game we'll have a bunch of objects on the desk that serve various purposes, like a clipboard you can flip through to
see character info or a speaker to interrogate, and a badge for some flavor text. ("Detectvie is spelled incorrectly on the badge on purpose, it's an old reference. Bonus points if you know what it's from!")<br /><br />
We'll start with adding our "button" objects to the room, then go through them one by one.

## Part 1: Making the Room Objects

Make the following objects:
- **obj_coffee:** spr_coffee
- **obj_file:** spr_file
- **obj_phone_base:** spr_phone
- **obj_speaker:** spr_speaker
- **obj_badge:** spr_badge
- **obj_clipboard:** spr_clipboard

Drop the objects in the room on the table. Anywhere is fine but I went in order: Phone, Clipboard, Speaker, Badge / Coffee, File<br /><br />
Next we'll make our UI objects, the pop-ups that contain our data. First we want to make an object called:<br /><br />
**par_ui**<br /><br />
It'll be our parent object (set par_ui to Parent in the Object Settings) for the following objects:
- **obj_ui_badge:** spr_ui_badge
- **obj_ui_btn:** spr_ui_btn
- **obj_ui_clipboard:** spr_ui_clipboard
- **obj_ui_file:** spr_ui_file
Parent objects are great for the collision_rectangle function, which we'll use to check for overlaps.<br /><br />
Let's control the depth of our inert objects and the functionality on the desk objects that control our UI:

#### obj_coffee

#### Create

```gml
depth = 10
```

#### obj_file

#### Create

```gml
depth = 7
is_open = false
```

#### Step

```gml
var hitting_ui = collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,par_ui,0,0)

if instance_exists(obj_ui_file) {
	is_open = true
	image_alpha = 0
}
else {
	is_open = false	
	image_alpha = 1
}
if !is_open and global.can_press and !hitting_ui {
	if point_in_rectangle(mouse_x,mouse_y,bbox_left,bbox_top,bbox_right,bbox_bottom) {
		if mouse_check_button_pressed(mb_left) {
			image_alpha = 0
			var ui_obj = instance_create_layer(room_width/2,room_height/2,"Instances",obj_ui_file)
			array_push(global.z_bag,ui_obj.id)
			is_open = true
			
		}
	}
}
```
obj_clipboard and obj_badge share a lot of the same code but we change out the relevant UI objects

#### obj_badge

#### Create

```gml
depth = 7
is_open = false
```

#### Step

```gml
var hitting_ui = collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,par_ui,0,0)

if instance_exists(obj_ui_badge) {
	is_open = true
	image_alpha = 0
}
else {
	is_open = false	
	image_alpha = 1
}
if !is_open and global.can_press and !hitting_ui {
	if point_in_rectangle(mouse_x,mouse_y,bbox_left,bbox_top,bbox_right,bbox_bottom) {
		if mouse_check_button_pressed(mb_left) {
			image_alpha = 0
			var ui_obj = instance_create_layer(room_width/2+200,room_height/2+100,"Instances",obj_ui_badge)
			array_push(global.z_bag,ui_obj.id)
			is_open = true
			
		}
	}	
}
```

#### obj_clipboard

#### Create

```gml
depth = 7
is_open = false
```

#### Step

```gml
if instance_exists(obj_ui_clipboard) {
	is_open = true
	image_alpha = 0
}
else {
	is_open = false	
	image_alpha = 1
}
if !is_open and global.can_press {
	if point_in_rectangle(mouse_x,mouse_y,bbox_left,bbox_top,bbox_right,bbox_bottom) {
		global.touching_file = true
		if mouse_check_button_pressed(mb_left) {
			image_alpha = 0
			var ui_obj = instance_create_layer(room_width/2-250,room_height/2,"Instances",obj_ui_clipboard)
			array_push(global.z_bag,ui_obj.id)
			is_open = true
			
		}
	}
	else {
		global.touching_file = false	
	}
}
```

## Part 2: Making the UI Objects

We covered the method in the intro but basically we're using the UI objects as container objects, they spawn various other UI objects and also control their movement, that keeps things nice
and snappy when dragging around. The depth trick is a little more involved, since arrays are also snappy in GameMaker we can load a few depth states into a multidimensional array and simply remove
and / or re-add objects to the array, assign them a fresh depth, and all that's left is disabling the UI objects "behind" the top object in the array. It sounds sort of complicated, because it is!<br /><br />
So let's add that to **obj_controller**, with a bit more code we'll use later on.

### obj_controller (again)

#### Create 

```gml
global.z_arr = [[5,6],[3,4],[1,2]]
global.z_bag = []

global.ui_delay = 0
global.ui_delay_total = 3

global.decision_bag = []

global.can_press = false
global.dialogue_open = false
```
*Note: if testing at the end of this stage, set global.can_press to true*<br /><br />

Next we want to follow another "Farmer in the Dell" and create the inner objects for our container objects.

### obj_photo

#### Create 

```gml
depth = 7
image_speed = 0
```

### obj_ui_btn

#### Create 

```gml
right = true
draw_string = ""

Step

if right {
	draw_string = ">>"	
}
else {
	draw_string = "<<"	
}
```

#### Draw

```gml
draw_set_font(fnt_num)
draw_set_color(c_gray)
draw_text(x+50,y+10,draw_string)
```

Next we make our three UI objects, following the same rules around container objects and using a psuedo-function in the Step Event to check for object depth.

### obj_ui_clipboard

#### Create 

```gml
depth = 8

photo = instance_create_layer(x-100,y-100,"Instances",obj_photo)
photo.image_index = 0

dfx = 0
dfy = 0

z_index = -1

is_dragging = false
is_active = true

btn_right = instance_create_layer(x+62,y+200,"Instances",obj_ui_btn)
btn_left = instance_create_layer(x-230,y+200,"Instances",obj_ui_btn)
btn_left.right = false

data_arr = global.clipboard_arr
data_index = 0

npc_name = ""
profession = ""
description = ""


function mouse_in_bounds() {

	if mouse_x > 50 and mouse_x < room_width-50 and mouse_y > 50 and mouse_y < room_height-50 {
		return true	
	}
}

function array_remove(arr,value) {

	var temp_arr = []
	
	for (var i = 0;i<array_length(arr);i++) {
		if arr[i] != value {
			array_push(temp_arr,arr[i])	
		}
	}
	return temp_arr
}
```

#### Step

```gml
npc_name = data_arr[data_index].npc_name
profession = data_arr[data_index].profession
description = data_arr[data_index].description
photo.image_index = data_arr[data_index].photo_index

if instance_exists(obj_ui_file) or instance_exists(obj_ui_badge) {
	
	var obj1_pass = true
	var obj2_pass = true
	
	if instance_exists(obj_ui_file) {
		var ui_obj = obj_ui_file
		if ui_obj.z_index > z_index and point_in_rectangle(mouse_x,mouse_y,ui_obj.bbox_left,ui_obj.bbox_top,ui_obj.bbox_right,ui_obj.bbox_bottom) {
			obj1_pass = false
		}
		else {
			obj1_pass = true	
		}
	}
	if instance_exists(obj_ui_badge) {
			var ui_obj = obj_ui_badge
				if ui_obj.z_index > z_index and point_in_rectangle(mouse_x,mouse_y,ui_obj.bbox_left,ui_obj.bbox_top,ui_obj.bbox_right,ui_obj.bbox_bottom) {
			obj2_pass = false
		}
		else {
			obj2_pass = true	
		}
	}
	
	if !obj1_pass or !obj2_pass {
		is_active = false	
	}
	else {
		is_active = true	
	}
	
}
else {
	is_active = true	
}


if array_contains(global.z_bag,id) {
	
	z_index = array_get_index(global.z_bag,id)
	
	depth = global.z_arr[z_index][1]
	photo.depth = global.z_arr[z_index][0]
	btn_left.depth = global.z_arr[z_index][0]
	btn_right.depth = global.z_arr[z_index][0]
	
}

if is_active and !is_dragging {

	if point_in_rectangle(mouse_x,mouse_y,btn_left.bbox_left,btn_left.bbox_top,btn_left.bbox_right,btn_left.bbox_bottom) {
		if mouse_check_button_pressed(mb_left) {
			
			if data_index > 0 {
				data_index--	
			}
			else if data_index <= 0{
				data_index = array_length(data_arr)-1	
			}

		}
	}
	else if point_in_rectangle(mouse_x,mouse_y,btn_right.bbox_left,btn_right.bbox_top,btn_right.bbox_right,btn_right.bbox_bottom) {
		if mouse_check_button_pressed(mb_left) {
			
			if data_index < array_length(data_arr)-1 {
				data_index++
			}
			else if data_index >= array_length(data_arr)-1{
				data_index = 0
			}

		}
	}
	else if point_in_rectangle(mouse_x,mouse_y,bbox_left,bbox_top,bbox_right,bbox_bottom) {
		
		if mouse_check_button_pressed(mb_left) {
			if array_contains(global.z_bag,id) {
				global.z_bag = array_remove(global.z_bag,id)
			}
			array_push(global.z_bag,id)
			z_index = array_get_index(global.z_bag,id)
			dfx = x - mouse_x
			dfy = y - mouse_y
			is_dragging = true	
		}
		if mouse_check_button_pressed(mb_right) and !global.ui_delay {
			
			if array_contains(global.z_bag,id) {
				global.z_bag = array_remove(global.z_bag,id)
			}
			if instance_exists(photo) {
				instance_destroy(photo)	
			}
			if instance_exists(btn_left) {
				instance_destroy(btn_left)	
			}
			if instance_exists(btn_right) {
				instance_destroy(btn_right)	
			}
			if !instance_exists(photo) and !instance_exists(btn_right) and !instance_exists(btn_left){
				global.ui_delay = global.ui_delay_total
				instance_destroy()
			}
		}
		 
	}
	
}

if is_active and is_dragging {
	
	if mouse_check_button(mb_left) and mouse_in_bounds() {
		x = mouse_x + dfx
		y = mouse_y + dfy
		
		photo.x = mouse_x + dfx - 100
		photo.y = mouse_y + dfy - 100
		
		btn_left.x = mouse_x + dfx - 230
		btn_left.y = mouse_y + dfy + 200
		
		btn_right.x = mouse_x + dfx + 62
		btn_right.y = mouse_y + dfy + 200
		
	}
	else {
		is_dragging = false	
	}
}
```

#### Draw

```gml
draw_self()
draw_set_font(fnt_h2)
draw_set_color(c_black)
draw_text(x-23,y-150,npc_name)
draw_text(x-23,y-100,profession)

draw_set_font(fnt_h2)
draw_text_ext(x-173,y,description,30,350)
```

### obj_ui_file

#### Create

```gml
depth = 8

file_data = global.game_info.file

safe_photo = instance_create_layer(x+140,y-140,"Instances",obj_photo)
safe_photo.image_index = 5

weapon_photo = instance_create_layer(x+300,y,"Instances",obj_photo)
weapon_photo.image_index = 6

dfx = 0
dfy = 0

z_index = -1

is_dragging = false
is_active = true

function mouse_in_bounds() {

	if mouse_x > 50 and mouse_x < room_width-50 and mouse_y > 50 and mouse_y < room_height-50 {
		return true	
	}
	
}

function array_remove(arr,value) {

	var temp_arr = []
	
	for (var i = 0;i<array_length(arr);i++) {
		if arr[i] != value {
			array_push(temp_arr,arr[i])	
		}
	}
	return temp_arr
}
```

#### Step

```gml
if instance_exists(obj_ui_clipboard) or instance_exists(obj_ui_badge) {
	
	var obj1_pass = true
	var obj2_pass = true
	
	if instance_exists(obj_ui_clipboard) {
		var ui_obj = obj_ui_clipboard
		if ui_obj.z_index > z_index and point_in_rectangle(mouse_x,mouse_y,ui_obj.bbox_left,ui_obj.bbox_top,ui_obj.bbox_right,ui_obj.bbox_bottom) {
			obj1_pass = false
		}
		else {
			obj1_pass = true	
		}
	}
	if instance_exists(obj_ui_badge) {
			var ui_obj = obj_ui_badge
				if ui_obj.z_index > z_index and point_in_rectangle(mouse_x,mouse_y,ui_obj.bbox_left,ui_obj.bbox_top,ui_obj.bbox_right,ui_obj.bbox_bottom) {
			obj2_pass = false
		}
		else {
			obj2_pass = true	
		}
	}
	
	if !obj1_pass or !obj2_pass {
		is_active = false	
	}
	else {
		is_active = true	
	}
	
}
else {
	is_active = true	
}

if array_contains(global.z_bag,id) {
	
	z_index = array_get_index(global.z_bag,id)
	
	depth = global.z_arr[z_index][1]
	safe_photo.depth = global.z_arr[z_index][0]
	weapon_photo.depth = global.z_arr[z_index][0]
	
}

if is_active and !is_dragging {

	if point_in_rectangle(mouse_x,mouse_y,bbox_left,bbox_top,bbox_right,bbox_bottom) {
		
		if mouse_check_button_pressed(mb_left) {
			if array_contains(global.z_bag,id) {
				global.z_bag = array_remove(global.z_bag,id)
			}
			array_push(global.z_bag,id)
			dfx = x - mouse_x
			dfy = y - mouse_y
			is_dragging = true	
		}
		if mouse_check_button_pressed(mb_right) and !global.ui_delay {
			if array_contains(global.z_bag,id) {
				global.z_bag = array_remove(global.z_bag,id)
			}
			if instance_exists(safe_photo) {
				instance_destroy(safe_photo)	
			}
			if instance_exists(weapon_photo) {
				instance_destroy(weapon_photo)	
			}
			if !instance_exists(safe_photo) and !instance_exists(weapon_photo) {
				global.ui_delay = global.ui_delay_total
				instance_destroy()
			}
		}
		
	}
	
}

if is_dragging {

	if mouse_check_button(mb_left) and mouse_in_bounds() {
		x = mouse_x + dfx
		y = mouse_y + dfy
		
		safe_photo.x = mouse_x + dfx + 140
		safe_photo.y = mouse_y + dfy - 140
		
		weapon_photo.x = mouse_x + dfx + 300
		weapon_photo.y = mouse_y + dfy
		
	}
	else {
		is_dragging = false	
	}
}
```

#### Draw

```gml
draw_self()
draw_set_font(fnt_h1)
draw_set_color(c_black)
draw_text(x-400,y-200,file_data.title)

draw_set_font(fnt_text)
draw_text_ext(x-400,y-150,file_data.description,18,350)
```

### obj_ui_badge

#### Create

```gml
depth = 7

dfx = 0
dfy = 0

is_dragging = false
is_active = true



function mouse_in_bounds() {

	if mouse_x > 50 and mouse_x < room_width-50 and mouse_y > 50 and mouse_y < room_height-50 {
		return true	
	}
	
	
}

z_index = -1

function array_remove(arr,value) {

	var temp_arr = []
	
	for (var i = 0;i<array_length(arr);i++) {
		if arr[i] != value {
			array_push(temp_arr,arr[i])	
		}
	}
	return temp_arr
}
```

#### Step

```gml
if instance_exists(obj_ui_file) or instance_exists(obj_ui_clipboard) {
	
	var obj1_pass = true
	var obj2_pass = true
	
	if instance_exists(obj_ui_file) {
		var ui_obj = obj_ui_file
		if ui_obj.z_index > z_index and point_in_rectangle(mouse_x,mouse_y,ui_obj.bbox_left,ui_obj.bbox_top,ui_obj.bbox_right,ui_obj.bbox_bottom) {
			obj1_pass = false
		}
		else {
			obj1_pass = true	
		}
	}
	if instance_exists(obj_ui_clipboard) {
			var ui_obj = obj_ui_clipboard
				if ui_obj.z_index > z_index and point_in_rectangle(mouse_x,mouse_y,ui_obj.bbox_left,ui_obj.bbox_top,ui_obj.bbox_right,ui_obj.bbox_bottom) {
			obj2_pass = false
		}
		else {
			obj2_pass = true	
		}
	}
	
	if !obj1_pass or !obj2_pass {
		is_active = false	
	}
	else {
		is_active = true	
	}
	
}
else {
	is_active = true	
}


if array_contains(global.z_bag,id) {
	
	z_index = array_get_index(global.z_bag,id)
	
	depth = global.z_arr[z_index][1]
	
}

if is_active and !is_dragging {

	
	if point_in_rectangle(mouse_x,mouse_y,bbox_left,bbox_top,bbox_right,bbox_bottom) and is_active {
		
		if mouse_check_button_pressed(mb_left) {
			if array_contains(global.z_bag,id) {
				global.z_bag = array_remove(global.z_bag,id)
			}
			array_push(global.z_bag,id)
			z_index = array_get_index(global.z_bag,id)
			dfx = x - mouse_x
			dfy = y - mouse_y
			is_dragging = true	
		}
		if mouse_check_button_pressed(mb_right) and !global.ui_delay {
			
			if array_contains(global.z_bag,id) {
				global.z_bag = array_remove(global.z_bag,id)
			}
			instance_destroy()
		}
		
	}
	
}

if is_active and is_dragging {
	
	if mouse_check_button(mb_left) and mouse_in_bounds() {
		x = mouse_x + dfx
		y = mouse_y + dfy

	}
	else {
		is_dragging = false	
	}
}
```
<br /><br />
# Super Easy Dialogue With Structs

The dialogue itself just pulls from the global structs we set up, based on the global dialogue array and index of the selected character. Simple!
But before we start, let's check the Create Event for our **obj_controller** and add our Step Event.

### obj_controller (full)

#### Create

```gml
randomize()

overlay = instance_create_layer(0,0,"Instances",obj_overlay_black)
overlay.image_alpha = 1

global.can_press = false
global.dialogue_open = false

global.selecting_perp = false
global.final_decision = false
global.decision_num = -1

global.game_info = {
	 
	phone_intro: ["Commandant: Welcome, detective",
				"A pawn shop was robbed this morning. There were five suspects located in the store at the time.",
				"We're not sure if there are one or more perpetrators, you'll have to check them individually.",
				"Also, we think the criminals are working with accomplices, body doubles that look just like them.",
				"You'll have to check all information and see if their story matches. If not, call it in.",
				"Good luck, Detective."
				],
	phone_callback: [],	
	file: {
		
		title: "Gunderson's Safe",
		description: "Mr. Gunderson was working at his pawn shop when around 9:30a.m. he noticed his safe was empty. There were five people present in the store at the time."
	}
	
}

global.npcs = {
	
	barry: {
		data: {
			npc_id: 0,
			head_adjust: [0,-169],
			eyes_adjust: [3,-179],
			mouth_adjust : [0,-120],
			draw_adjust:  [-10,-48],
		},
		inno_info: {
			photo_index: 0,
			npc_name: "Barry Oldman",
			profession: "Business",
			description: "Claims to have been looking for a present for his wife."
			
		},
		guilty_info: {
			photo_index: 0,
			npc_name: "Barry Oldman",
			profession: "Business",
			description: "Claims to have been looking for a present for his wife."
			
		},
		inno_dialogue: {
			npc_name: "Uh, my name? It's uh... Barry Oldman",
			what_doing: "I was looking for a gift for my wife."
			
			
		},
		guilty_dialogue: {
			npc_name: "Uh, my name? It's uh... Larry Oldman",
			what_doing: "I was looking for a gift for my wife."
			
		}
	},
	
	dennis: {
		data: {
			npc_id: 1,
			head_adjust: [0,-68],
			eyes_adjust: [2,-80],
			mouth_adjust : [0,-25],
			draw_adjust:  [-18,30]
		},
		inno_info: {
			photo_index: 1,
			npc_name: "Dennis Belvitas",
			profession: "Funemployed",
			description: "Says he was in the pawn shop to sell his WWII Memorabilia."
			
		},
		guilty_info: {
			photo_index: 1,
			npc_name: "Dennis Belvitas",
			profession: "Funemployed",
			description: "Says he was in the pawn shop to sell his WWII Memorabilia."
			
		},
		inno_dialogue: {
			npc_name: "My name is Dennis!",
			what_doing: "I was selling my war trophies"
			
			
		},
		guilty_dialogue: {
			npc_name: "My name is Dennis!",
			what_doing: "I was selling my collection of Warhammer 30k collectibles"
			
		}
	},
	
	rita: {
		data: {
			npc_id: 2,
			head_adjust: [0,-112],
			eyes_adjust: [0,-110],
			mouth_adjust : [-1,-53],
			draw_adjust:  [-16,0]
		},
		inno_info: {
			photo_index: 2,
			npc_name: "Rita Hudson",
			profession: "Entrepeneur",
			description: "Says she was in the shop to reclaim her pearl earrings."
			
		},
		guilty_info: {
			photo_index: 2,
			npc_name: "Rita Hudson",
			profession: "Entrepeneur",
			description: "Says she was in the shop to reclaim her pearl earrings."
			
		},
		inno_dialogue: {
			npc_name: "It's Rita. Rita Hudson.",
			what_doing: "None of your business! But I was selling some things."
			
			
		},
		guilty_dialogue: {
			npc_name: "It's Rita. Rita Hudson.",
			what_doing: "I was um... looking for a gift."
		}
	},
	
	chris: {
		data: {
			npc_id: 3,
			head_adjust: [0,-200],
			eyes_adjust: [0,-211],
			mouth_adjust : [-1,-138],
			draw_adjust:  [-20,-23]
		},
		inno_info: {
			photo_index: 3,
			npc_name: "Chris Grover",
			profession: "Kiosk Worker",
			description: "Works in the mall next door, frequents the pawn shop on his lunch break."
			
		},
		guilty_info: {
			photo_index: 9,
			npc_name: "Chris Grover",
			profession: "Kiosk Worker",
			description: "Works in the mall next door, frequents the pawn shop on his lunch break."
			
		},
		inno_dialogue: {
			npc_name: "I'm Chris Grover. Nice to meet you!",
			what_doing: "I was just browsing."
			
			
		},
		guilty_dialogue: {
			npc_name: "I'm Chris Grover. Nice to meet you!",
			what_doing: "I was just browsing."
			
		}
	},
	
	buddy: {
		data: {
			npc_id: 4,
			head_adjust: [6,-112],
			eyes_adjust: [4,-120], 
			mouth_adjust : [8,-68],
			draw_adjust:  [-12,10]
		},
		inno_info: {
			photo_index: 4,
			npc_name: "Buddy Hudson",
			profession: "Driver",
			description: "Says he frequents the shop because he likes the smell"
			
		},
		guilty_info: {
			photo_index: 4,
			npc_name: "Buddy Hudson",
			profession: "Driver",
			description: "Says he frequents the shop because he likes the smell"
			
		},
		inno_dialogue: {
			npc_name: "My name is Buddy, buddy",
			what_doing: "Just looking around, you know?"
			
			
		},
		guilty_dialogue: {
			npc_name: "I'm Chris Grover. Nice to meet you!",
			what_doing: "Just looking around, you know?"
			
		}
	}
	
	
}

npc_create_arr = [global.npcs.barry,global.npcs.dennis,global.npcs.rita,global.npcs.chris,global.npcs.buddy]
npc_create_arr = array_shuffle(npc_create_arr)

global.clipboard_arr = []
global.dialogue_arr = []
global.selected_dialogue = {}
global.selected_id = noone
global.correct_arr = []

npc_id_arr = []
npc_pos_margin = 240
npc_buffer = 175

for (var i = 0;i<array_length(npc_create_arr);i++) {
	
	var final_position = npc_pos_margin + (npc_buffer*i)
	var start_position = 0
	if i > 2{
		start_position = final_position + 600
	}
	else {
		start_position = final_position - 600
	}

	
	npc_id_arr[i] = instance_create_layer(start_position,466,"Instances",obj_body)
	npc_id_arr[i].npc = npc_create_arr[i]
	npc_id_arr[i].npc_index = i
	npc_id_arr[i].final_x = final_position
	
	var guilty = choose(true,false)
	if guilty {
		global.correct_arr[i] = true
		global.clipboard_arr[i] = npc_create_arr[i].guilty_info
		global.dialogue_arr[i] = npc_create_arr[i].guilty_dialogue
	}
	else {
		global.correct_arr[i] = false
		global.clipboard_arr[i] = npc_create_arr[i].inno_info
		global.dialogue_arr[i] = npc_create_arr[i].inno_dialogue
	}
	
}

global.z_arr = [[5,6],[3,4],[1,2]]
global.z_bag = []

global.ui_delay = 0
global.ui_delay_total = 3

global.ringing = true
global.tutorial_finished = false

global.decision_bag = []
```

#### Step

```gml
if global.ui_delay > 0 {
	global.ui_delay--	
}
else {
	global.ui_delay = 0	
}

if instance_exists(obj_dialogue_box) {
	global.dialogue_open = true	
}
else {
	global.dialogue_open = false	
}

if global.ringing or global.dialogue_open or global.selecting_perp {
	global.can_press = false
}
```

OK, now that we've got our obj_controller finally squared away, we can work on our dialogue box. Since this game sort of lives on switchng out different game states, it was inevitable that we would shove a lot of functionality into an enum.
And here it is! We'll make an enum for any functionality we want, drop the code in there, and we know that it will only run when triggered.

### obj_dialogue_box

#### Create

```gml
depth = 1

image_alpha = 0
text_alpha = 0

intro_index = 0
selected_index = -1

npc_num = -1

enum DIALOGUE_STATE {

	NULL,
	INTRO,
	PERP_ASK,
	PERP_ANSWER_NAME,
	PERP_ANSWER_DOING,
	PHONE_ASK,
	PHONE_ANSWER,
	FINAL_DECISION,
	FINAL_RESET,
	END_GAME,
	CLOSING

}

enum TYPING_STATE {


	EMPTY,
	TYPING,
	FINISHED
	
}

typing_state = TYPING_STATE.EMPTY

fading_in = true
dialogue_state = DIALOGUE_STATE.NULL
draw_string = ""
current_string = ""
finished_typing = false
data = {}

btn1 = noone
btn2 = noone

x = room_width / 2
y = 650

function type_text(cs,fs) {
	var fs_length = string_length(fs)
	for (var i = 0;i<fs_length;i++) {
		cs += string_char_at(fs,i)	
	}
}
```

#### Step

```gml
var touching_box = point_in_rectangle(mouse_x,mouse_y,bbox_left,bbox_top,bbox_right,bbox_bottom)

if fading_in {
	if image_alpha < 1 {
		image_alpha += 0.05
		text_alpha += 0.05
	}
	else {
		fading_in = false	
	}
}
else {
	switch(dialogue_state) {
		
		case DIALOGUE_STATE.INTRO:
		
			if touching_box {
				if mouse_check_button_pressed(mb_left) {
					draw_string = ""
					current_string = ""
					intro_index++	
				}
			}
	
			if intro_index < array_length(global.game_info.phone_intro)-1 {
				typing_state = TYPING_STATE.TYPING
				draw_string = global.game_info.phone_intro[intro_index]
				
			}
			else {
					global.tutorial_finished = true
					dialogue_state = DIALOGUE_STATE.CLOSING
			}
			
		
			break;
			
		case DIALOGUE_STATE.PERP_ASK:
			typing_state = TYPING_STATE.TYPING
			draw_string = "Number " + string(npc_num) + ". I have questions for you."
			
			if finished_typing {
				if !instance_exists(btn1) {
					btn1 = instance_create_layer(x-520,y-30,"Instances",obj_dialogue_btn)	
					btn1.name_q = true
				}
			
				if !instance_exists(btn2) {
					btn2 = instance_create_layer(x-520,y+20,"Instances",obj_dialogue_btn)	
					btn2.what_doing_q = true
				}
			}
			
		
			break;
			
		case DIALOGUE_STATE.PERP_ANSWER_NAME:
		
			if instance_exists(btn1) {
				instance_destroy(btn1)	
			}
			if instance_exists(btn2) {
				instance_destroy(btn2)	
			}
			
			if touching_box {
				if mouse_check_button_pressed(mb_left) {
					dialogue_state = DIALOGUE_STATE.CLOSING	
				}
			}
			
		
			draw_string = global.selected_dialogue.npc_name
			typing_state = TYPING_STATE.TYPING
		
		
			break;
			
		case DIALOGUE_STATE.PERP_ANSWER_DOING:
		
			if instance_exists(btn1) {
				instance_destroy(btn1)	
			}
			if instance_exists(btn2) {
				instance_destroy(btn2)	
			}
			
			
			typing_state = TYPING_STATE.TYPING
			draw_string = global.selected_dialogue.what_doing
			
			
			if touching_box {
				if mouse_check_button_pressed(mb_left) {
					dialogue_state = DIALOGUE_STATE.CLOSING	
				}
			}
		
		
			break;
			
			
		case DIALOGUE_STATE.CLOSING:
		
				draw_string = ""
				current_string = ""
		
			if image_alpha > 0 {
				image_alpha -= 0.05
				text_alpha -= 0.05
			}
			else {
				
				global.can_press = true
				instance_destroy()	
			}
		
		
			break;
			
		case DIALOGUE_STATE.PHONE_ASK:
		
			typing_state = TYPING_STATE.TYPING
			draw_string = "Commandant: Well, detective? Are you ready?"
			
			if finished_typing {
				if !instance_exists(btn1) {
					btn1 = instance_create_layer(x-520,y-30,"Instances",obj_dialogue_btn)	
					btn1.ready = true
				}
			
				if !instance_exists(btn2) {
					btn2 = instance_create_layer(x-520,y+20,"Instances",obj_dialogue_btn)	
					btn2.more_time = true
				}
			}
			
			break;
			
		case DIALOGUE_STATE.FINAL_DECISION:
			
			global.final_decision = true
		
			global.decision_num = array_length(global.decision_bag)
			
			if global.decision_num < 5 {
				draw_string = "What do you think about #" + string(global.decision_num+1) 	
			}
			else {
				dialogue_state = DIALOGUE_STATE.END_GAME	
			}
			
			if !instance_exists(btn1) {
				btn1 = instance_create_layer(x-520,y-30,"Instances",obj_dialogue_btn)	
				btn1.guilty = true
			}
			
			if !instance_exists(btn2) {
				btn2 = instance_create_layer(x-520,y+20,"Instances",obj_dialogue_btn)	
				btn2.inno = true
			}
			
			break;
			
		case DIALOGUE_STATE.FINAL_RESET:
		
			if instance_exists(btn1) {
				instance_destroy(btn1)	
			}
			if instance_exists(btn2) {
				instance_destroy(btn2)	
			}
			else if !instance_exists(btn1) and !instance_exists(btn2) {
				dialogue_state = DIALOGUE_STATE.FINAL_DECISION	
			}
		
			break;
			
		case DIALOGUE_STATE.END_GAME:
		
			if instance_exists(btn1) {
				instance_destroy(btn1)	
			}
			if instance_exists(btn2) {
				instance_destroy(btn2)	
			}
			
			draw_string = "Alright, detective. We'll look into your findings."
			
			if touching_box {
			
				if mouse_check_button_pressed(mb_left) {
					var overlay = instance_create_layer(0,0,"Instances",obj_overlay_black)
					overlay.fading_out = true
				}
				
			}
		
			break;
	
		
	}
}

switch (typing_state) {
	
	case TYPING_STATE.EMPTY:
	
		draw_string = ""
		current_string = ""
		
		break;
		
	case TYPING_STATE.TYPING:
	
		var dsl = string_length(draw_string)
		var csl = string_length(current_string)
		if dsl > 0 {
			if csl < dsl {
				current_string += string_char_at(draw_string,csl+1)
				finished_typing = false
			}
			else {
				finished_typing = true
			}
		}
	
		break;
		
	case TYPING_STATE.FINISHED:
	
	
		break;
	
}
```

#### Draw

```gml
draw_self()

draw_set_font(fnt_h1)
draw_set_color(c_white)
draw_set_alpha(text_alpha)
draw_text_ext(x-520,y-80,current_string,50,1000)
draw_set_alpha(1)
```

At the same time we also need to make the buttons for it, same basic idea:

### obj_dialogue_btn

#### Create

```gml
right = true
draw_string = ""
text_color = c_gray
text_alpha = 0

image_xscale = 9

name_q = false
what_doing_q = false

ready = false
more_time = false

guilty = false
inno = false
```

#### Step

```gml
if name_q {
	draw_string = "What's your name again?"	
}
if what_doing_q {
	draw_string = "What were you doing at the time of the robbery?"	
}
if ready {
	draw_string = "I'm ready to answer."	
}
if more_time {
	draw_string = "I need more time."	
}
if guilty {
	draw_string = "Guilty"	
}
if inno {
	draw_string = "Innocent"	
}

if point_in_rectangle(mouse_x,mouse_y,bbox_left,bbox_top,bbox_right,bbox_bottom) {

	text_color = c_white	
	
	if mouse_check_button_pressed(mb_left) {
		if instance_exists(obj_dialogue_box) {
			obj_dialogue_box.current_string = ""
			obj_dialogue_box.draw_string = ""
			if name_q {
				global.selected_id.npc_state = NPC_STATE.TALKING
				obj_dialogue_box.dialogue_state = DIALOGUE_STATE.PERP_ANSWER_NAME	
			}
			if what_doing_q {
				global.selected_id.npc_state = NPC_STATE.TALKING
				obj_dialogue_box.dialogue_state = DIALOGUE_STATE.PERP_ANSWER_DOING	
			}
			if ready {
				global.final_selection = true
				obj_dialogue_box.dialogue_state = DIALOGUE_STATE.FINAL_RESET
			}
			if more_time {
				obj_dialogue_box.dialogue_state = DIALOGUE_STATE.CLOSING	
			}
			if guilty {
				array_push(global.decision_bag,true)
				obj_dialogue_box.DIALOGUE_STATE = DIALOGUE_STATE.FINAL_RESET
			}
			if inno {
				array_push(global.decision_bag,false)
				obj_dialogue_box.DIALOGUE_STATE = DIALOGUE_STATE.FINAL_RESET
			}
		}
	}
	
}
else {
	text_color = c_gray	
}


if instance_exists(obj_dialogue_box) {
	text_alpha = obj_dialogue_box.text_alpha	
}
else {
	instance_destroy()	
}
```

#### Draw

```gml
draw_set_alpha(text_alpha)
draw_set_font(fnt_h1)
draw_set_color(text_color)
draw_text(x,y,draw_string)
draw_set_alpha(1)
```
Now we just need to make the desk objects that hook up to the dialogue!

### obj_phone_cord
**Sprite:** spr_phone

#### Create

```gml
depth = 11
image_speed = 0
image_index = 2
```

### obj_phone_receiver
**Sprite:** spr_phone

#### Create

```gml
depth = 9
image_speed = 0
image_index = 1
```

### obj_phone_base
**Sprite:** spr_phone

#### Create

```gml
depth = 10

image_speed = 0
image_index = 0

cord = instance_create_layer(x,y,"Instances",obj_phone_cord)
receiver = instance_create_layer(x,y,"Instances",obj_phone_receiver)

ringing_left = false

ring_counter = 5
ring_counter_total = 5

ring_delay = 15
ring_delay_total = 15
```

#### Step

```gml
var touching_receiver = point_in_rectangle(mouse_x,mouse_y,receiver.bbox_left,receiver.bbox_top,receiver.bbox_right,receiver.bbox_bottom)
var touching = point_in_rectangle(mouse_x,mouse_y,bbox_left,bbox_top,bbox_right,bbox_bottom)
var hitting_ui = collision_rectangle(bbox_left,bbox_top-20,bbox_right,bbox_bottom,par_ui,0,0)

if global.ringing {
	if ring_counter > 0 {
		if !ringing_left {
			if receiver.image_angle > -3 {
				receiver.image_angle--
				cord.image_yscale -= 0.01
			}
			else {
				ringing_left = true	
			}
		}
		else {
			if receiver.image_angle < 3 {
				receiver.image_angle++
				cord.image_yscale += 0.01
			}
			else {
				ring_counter--
				ringing_left = false	
			}
		}
	}
	else {
		receiver.image_angle = 0
		cord.image_yscale = 1
		if ring_delay > 0 {
			ring_delay--	
		}
		else {
			ring_delay = ring_delay_total
			ring_counter = ring_counter_total	
		}
	}
}
else {
	receiver.image_angle = 0
	cord.image_yscale = 1
}

if global.ringing {
	if touching or touching_receiver {
		if mouse_check_button_pressed(mb_left) {
			var dialogue = instance_create_layer(x,y,"Instances",obj_dialogue_box)
			dialogue.dialogue_state = DIALOGUE_STATE.INTRO
			global.ringing = false
		}
	}
}
else if global.can_press and !hitting_ui {
	if touching or touching_receiver {
		if mouse_check_button_pressed(mb_left) {
			var dialogue = instance_create_layer(x,y,"Instances",obj_dialogue_box)
			dialogue.dialogue_state = DIALOGUE_STATE.PHONE_ASK
			global.can_press = false
		}
	}
}
```

### obj_speaker

#### Create

```gml
depth = 10

image_speed = 0
image_index = 0
```

#### Step

```gml
var hitting_ui = collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,par_ui,0,0)

if global.can_press and !hitting_ui {
	if point_in_rectangle(mouse_x,mouse_y,bbox_left,bbox_top,bbox_right,bbox_bottom) {
		if mouse_check_button(mb_left) {
			image_index = 1	
		}
		else {
			image_index = 0
		}
		if mouse_check_button_released(mb_left) {
			global.selecting_perp = true
		}
	}
	else {
		image_index = 0
	}
}
```
Finally, we just need to make a controller object for our end game state, which just sort of consolidates everything that happened.

### obj_controller_gameover

#### Create

```gml
delay = 50
text_alpha = 0
title_string = "Interrogation Over"

function get_score(arr,arr2) {

	var corrects = 0
	
	for (var i = 0;i<array_length(arr);i++) {
		if arr[i] == arr2[i] {
			corrects++	
		}		
	}
	
	return corrects
	
}

final_score = get_score(global.correct_arr,global.decision_bag)

score_string = "You got " + string(final_score) + " out of 5 correct"

if final_score == 5 {
	draw_string = "Great work, detective, you solved the case."
}
else if final_score > 3 {
	draw_string = "You did OK, detective, but there's still more work to be done."	
}
else {
	draw_string = "Oh boy, that was pretty terrible."	
}

play_string = "Click to play again."
```

#### Step

```gml
if text_alpha < 1 {
		text_alpha += 0.02
}

if delay > 0 {
	delay--	
}
else {
	if mouse_check_button_pressed(mb_left) {
		room_goto(Rm_Main)	
	}
}
```

#### Draw

```gml
draw_set_alpha(text_alpha)
draw_set_color(c_white)

draw_set_font(fnt_num)
draw_text(100,100,title_string)
draw_set_font(fnt_h1)
draw_text(100,200,score_string)
draw_text(100,300,draw_string)
draw_text(100,400,play_string)


draw_set_alpha(1)
```

Drop it in Rm_End, and we're ready for testing!
<br /><br />
# Next Steps

That's it! That should be the game! If you have any questions or ran into a bug you can reach out on Discord and I'll try to fix it as soon as I can. And if you need help either getting through this Speedrun Tutorial
or with expanding the project, feel free to reach out on Discord for that too.<br /><br />
So what now? You've got a nifty little game that was maybe more fun to make than it is to play, but that's the beauty of it! Now you can add your own story and customize it however you want. <br /><br />
If you wanted to turn it into the Next Big Indie GameTM I would look into how previous games repeated their core mechanic instead of introducing variety, and maybe combine it with a game from
an adjacent genre with a similar problem, like "Don't Feed the Monkeys" or "Curse of the Golden Idol", the latter shares a lot of mechanics with the Point and Click Speedrun Tutorial you can find on
my YouTube and here.<br /><br />
Thank you again to my Ko-Fi members for keeping me going, if you'd like to support my projects and the gamedev community please consider becoming a member! Thanks again!
