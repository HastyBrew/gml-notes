# Make Your Own 2D Platformer

## Navigation

1. [Gravity](#gravity)
2. [Wall Jumping](#wall-jumping)
3. [Hit Points](#hit-points)
4. [Enemies](#enemies)
5. [Dodging](#dodging)
6. [Building Levels](#building-levels)
7. [Collectibles](#collectibles)
8. [UI Menus](#ui-menus)
9. [Bosses](#bosses)

## Intro

For this project we'll be building a 2D Platformer using an asset kit from Craftpix. We'll be covering ALL the basics needed so you can start designing your own levels right away, from basic character movement and faking gravity to wall jumps, hit points, traps, enemies, dodging, building levels, collectibles, UI, and more! This is a pretty big project so I've tried to break it down into smaller pieces. If you have any questions you can reach out on Discord and we'll try to figure it out together, or if you want to suggest a better method for something, you can reach out through there too and I'll include it in the next video on this (which could take months). Speaking of...
<br /><br />
**Note:** I made this video a few months ago, I meant to reshoot it because there's a pretty big error. Since this originally came out of a much larger project, in the "Collectibles" and "UI" sections I accidentally used two sprites that aren't in the free kit, but in a paid kit only available though subscription. After much consideration I've decided to make lemonade out of it, leave those sections in (especially since a lot of people use their own assets anyways) and use my affiliate link for Craftpix, so I do receive a commission if you purchase their subscription service. I'm already a Craftpix subscriber, which is how this happened in the first place. Thank you!



# Gravity

To be clear upfront, we won't be using gravity in GameMaker at all. Like most Hasty Brews, we'll walk before we run and take our programming inspiration from early games and successful indies, which means forgetting realistic gravity for now. We're going to tweak individual jump and fall frames to get nice, smooth curves. And the best part is, you can change them however you want. Let's get into it.

## Project Setup
Open up GameMaker, select New Game, Blank Game, and click **Start**

### Rooms
#### Rm_Main
**Room Size:** 320x288

### GameMaker Settings

Before we start, let's change some internal GameMaker settings.
1. Under **Quick Access**, go to **Game Options > Windows** *(or whatever you're using)*
2. Open the **Graphics** Menu
   - Uncheck **Interpolate Colours Between Pixels** *(if checked)*
   - Check **Allow Window Resize**
  
### Groups

Let's create a few main groups, we'll make more as we go through.
#### Create Group
- Rooms
-  Sprites
   - Barriers
   - Blocks
   - Characters
     - Player
- Objects
   - Barriers
   - Blocks
   - Characters
     - Player

## Sprites
Fill out info about how these came from Craftpix, which ones they need to download, how to rename them, etc. <br /> <br />
How to set to center origin for all in Settings

### From Craftpix
Drag into **Sprites > Player**
1. spr_bg   *top left origin*
2. spr_boy_run_strip12
3. spr_boy_idle_strip11
4. spr_boy_jump
5. spr_boy_fall
6. spr_border   *turn on nine slice*

#### spr_block_master
We'll use this for a few things later, for now:

1. Duplicate spr_block_master to make **spr_ledge**
2. Click into **Edit Image**
3. At the top bar, select **Image > Convert to Frames**
	- **Number of Frames:** 1
  	- **Number per Row:** 1
    - **Frame Width:** 48
  	- **Frame Height:** 16
    - **Horizontal Pixel Offset:** 0
  	- **Vertical Pixel Offset:** 128
  
   
#### spr_border
Same basic thing, we'll take **spr_gui_nmaster**

1. Duplicate spr_gui_master to make **spr_border**
2. Click into **Edit Image**
3. At the top bar, select **Image > Convert to Frames**
	- **Number of Frames:** 1
  	- **Number per Row:** 1
    - **Frame Width:** 48
  	- **Frame Height:** 48
    - **Horizontal Pixel Offset:** 96
  	- **Vertical Pixel Offset:** 0
  

### Make in GameMaker
Make in **Sprites > Barriers**
1. **spr_barrier_floor**
   - **Size:** 32x4
   - **Color:** Magenta
   - **Origin:** Top Left
2. **spr_barrier_wall**
   - **Size:** 2x32
   - **Color:** Magenta
   - **Origin:** Top Left
3. **spr_player_floor**
   - **Size:** 14x2
   - **Color:** Aqua
   - **Origin:** Bottom Centre
4. **spr_player_mask**
   - **Size:** 18x30
   - **Color:** Aqua
   - **Origin:** 9x14

We create sprites so we can see the floors and fine-tune interactions, otherwise we could just draw them with collision_rectangle(). We'll use a mix of both, creating floor objects and adjusting them in the collision function.

Let's set our **Background** to **spr_bg** and turn on tiling, both horizontally and vertically. Let's change our grids to **16x16** at this point as well.

## Controller Objects

We'll make both an **initializer object** and a few **controller objects**, eventually moving the initializer to another Room.

### obj_initializer

First, let's stretch our window. Our original size is 320x288, original Gameboy-sized, but we'll need to beef that up for regular gameplay. We'll scale our window using a variable and set a few other top-level items.

#### Create

```gml
randomize()

window_scale = 4
window_set_size(room_width * window_scale, room_height * window_scale);
window_set_cursor(cr_none)
```

### obj_controller_input

Before we get to programming our player, we're going to make a little helper controller. This will be used to handle, like, actual physical controller input later, but we want to build in the functionality now so we're not troubleshooting later.

#### Create

```gml
global.left_pressed = 0
global.up_pressed = 0
global.right_pressed = 0
global.down_pressed = 0
global.A_pressed = 0
global.B_pressed = 0


global.left_held = 0
global.up_held = 0
global.right_held = 0
global.down_held =0

global.A_held = 0
global.B_held = 0
```

#### Step

```gml
global.left_pressed = keyboard_check_pressed(vk_left)
global.up_pressed = keyboard_check_pressed(vk_up)
global.right_pressed = keyboard_check_pressed(vk_right)
global.down_pressed = keyboard_check_pressed(vk_down)

global.A_pressed = keyboard_check_pressed(vk_space)
global.B_pressed = keyboard_check_pressed(vk_shift)


global.left_held = keyboard_check(vk_left)
global.up_held = keyboard_check(vk_up)
global.right_held = keyboard_check(vk_right)
global.down_held = keyboard_check(vk_down)

global.A_held = keyboard_check(vk_space)
global.B_held = keyboard_check(vk_shift)
```
### obj_controller_main

We won't actually use this yet, but it feels weird to leave it out. <br /><br />

Drop **obj_initializer, obj_controller_input,** and **obj_controller_main** into **Rm_Main**, in that order.

### par_solid

This will be the parent object for anything immovable. We don't use it until much, much later, but we'll use the same design theory when creating multiple objects for checking collisions, so we'll also set up a parent object to check for collisions between a group of objects. Just good dev thinking.

## Collision Objects

Again we're mostly just using these as visual cues for testing and fine-tuning. We'll set the image alpha to 0 for now but change it to 1 when we want to see the barriers.

### obj_barrier_floor
**Sprite:** spr_barrier_floor

#### Create

```gml
image_alpha = 0
depth = 0

is_moving = false
```

### obj_player_floor
**Sprite:** spr_player_floor

#### Create

```gml
image_alpha = 0
```

### obj_barrier_wall
**Sprite:** spr_barrier_wall
Leave for now


### obj_border
**Sprite:** spr_border

#### Create

```gml
depth = 8

barrier_floor = instance_create_layer(x,y,"Instances",obj_barrier_floor)
barrier_floor.image_xscale = image_xscale
```

Duplicate to make **obj_ledge_still**

### obj_ledge_still
**Sprite:** spr_ledge

#### Create

```gml
depth = 8

barrier_floor = instance_create_layer(x,y,"Instances",obj_barrier_floor)
barrier_floor.image_xscale = image_xscale
```

Drag **obj_block_metal** into the room, drag it out to make a floor, then place **obj_ledge** somewhere.

## Player Object

OK, here we go. We're going to set up a player and turn on collision with the floor with the **collision_rectangle()** function.

### obj_player

#### Create
We'll make a few enums and set some variables. The enum PR controls player direction and the enum PLAYER_STATE will control pretty much everything else. We'll also set can_move_left and can_move_right as hard variables, this could have been another enum or tied to PR but I feel this was the most readable.
<br />
<br />
We're not using real physics here. Classic games didn't, either. Instead we'll control the jump with frame counters and speed curves so we can get predictable movement that's easy to fine-tune.

```gml
depth = 5

enum PLAYER_STATE {

	IDLE,
	IS_ENTERING,
	RUNNING,
	JUMPING,
	FALLING
	
}

player_state = PLAYER_STATE.FALLING

enum PR {

	NONE,
	LEFT,
	RIGHT
		
}

pr = PR.NONE

mask_index = spr_player_mask

global.initial_x = x
global.initial_y = y

can_move_left = true
can_move_right = true

fall_speed = 2.3 // initial fall speed
fall_frames = 0 
fall_frames_max = 20 // frames before reaching max speed
fall_multiplier = 3

// horizontal movement
og_move_speed = 1.5 
move_speed = 1.5

jump_height = 1.3 // move up every frame
jump_counter = 0
jump_counter_total = 22 // frames before falling

is_jumping = false

still_falling = false

float_frames = 3 // horizontal frames to make arc
float_frames_total = 3

// create floor collision object
feet = instance_create_layer(x,y+15,"Instances",obj_player_floor) 
```

#### Step
So here's where we bring everything together. We use our globals to control input, we use our enums to control sprites and image speed, reset counters, etc, and finally we create our pseudo arc system.

```gml

	
// FEET CORRECTION
if instance_exists(feet) {
	feet.x = x
	feet.y = y+15
}
	

// COLLISION VARS

var feet_hitting_floor = collision_rectangle(feet.bbox_left,feet.bbox_top,feet.bbox_right,feet.bbox_bottom,obj_barrier_floor,false,false)
var hitting_wall = collision_rectangle(feet.bbox_left,feet.bbox_top,feet.bbox_right,feet.bbox_bottom,obj_barrier_wall,false,false)


// SWITCH
	
switch(pr) {

	case PR.LEFT:

		image_xscale = -1

		break;
		
	case PR.RIGHT:

		image_xscale = 1
	
		break;
	
}

switch(player_state) {

	case PLAYER_STATE.FALLING:

	can_move_left = true
	can_move_right = true

	still_falling = false

	
		is_jumping = true

		sprite_index = spr_boy_fall
		image_speed = 0
		jump_counter = 0
		
		if feet_hitting_floor {
			player_state = PLAYER_STATE.IDLE	
		}

		else {
			y += fall_speed	
			feet. y += fall_speed
		}
		

		if global.left_held and can_move_left {
			
			pr = PR.LEFT
			can_move_right = true
			x -= move_speed
			feet.x -= move_speed
			
		}
		else if global.right_held and can_move_right {
			pr = PR.RIGHT
			can_move_left = true
			x += move_speed
			feet.x += move_speed
		}

		break;
		
	case PLAYER_STATE.IDLE:
	
		float_frames = float_frames_total
	
		if x < 0 {
			player_state = PLAYER_STATE.IS_ENTERING	
		}
	
		still_falling = false
		is_jumping = false
	
		sprite_index = spr_boy_idle
		image_speed = 0.3
		jump_counter = 0
		

		if can_move_left{
			if global.left_held {
				player_state = PLAYER_STATE.RUNNING
			}
		}
		if can_move_right {
			if global.right_held {
				player_state = PLAYER_STATE.RUNNING	
			}
		}
		
		
		if global.A_pressed {
			player_state = PLAYER_STATE.JUMPING	
		}
			else {
				pr = PR.NONE
			}

			break;

		case PLAYER_STATE.RUNNING:

			still_falling = false
			is_jumping = false
	
			sprite_index = spr_boy_run
			image_speed = 0.8
			jump_counter = 0
		
			if x < 0 {
				player_state = PLAYER_STATE.IS_ENTERING	
			}
			else if x > room_width + 5 {
				room_restart()	
			}

		
			else if global.left_held and can_move_left {
			
				pr = PR.LEFT
				can_move_right = true
				x -= move_speed
				feet.x -= move_speed	
				
			}
			else if global.right_held and can_move_right {
				pr = PR.RIGHT
				can_move_left = true

				x += move_speed
				feet.x += move_speed
			}
			else {
				player_state = PLAYER_STATE.IDLE	
			}
		
			if global.A_pressed {
				player_state = PLAYER_STATE.JUMPING	
			}

		
		
			if !feet_hitting_floor {
				if !hitting_wall {
					if pr == PR.LEFT {
						x -= 4
						feet.x -= 4
						player_state = PLAYER_STATE.FALLING
					}
					if pr == PR.RIGHT {
						x += 4
						feet.x += 4
						player_state = PLAYER_STATE.FALLING
					}
				}
			}
		

		
		
			break;
			
		case PLAYER_STATE.IS_ENTERING:
	
			sprite_index = spr_boy_run
			image_speed = 0.8
			x += move_speed
			pr = PR.RIGHT
			can_move_left = true
			can_move_right = true
		
			if x > 16 {
				player_state = PLAYER_STATE.IDLE	
			}
			
			break;
		
		
		
		case PLAYER_STATE.JUMPING:
	

			sprite_index = spr_boy_jump
			image_speed = 0

	
		
			is_jumping = true
		
			if global.A_held {
				if jump_counter < jump_counter_total {
					jump_counter++
					y -= jump_height
					feet.y -= jump_height
				}
				else {
					if float_frames > 0 {
						float_frames--	
					}
					else {
						player_state = PLAYER_STATE.FALLING	
					}
				}
		
				if !still_falling{
					if global.left_held and can_move_left  {
			
						pr = PR.LEFT
						can_move_right = true
						image_xscale = -1
						x -= move_speed
						feet.x -= move_speed
				
					}
					else if global.right_held and can_move_right {
						pr = PR.RIGHT
						can_move_left = true
						image_xscale = 1
						x += move_speed
					}
				}
				else {
					if pr == PR.LEFT {
						image_xscale = -1
						x -= move_speed
						feet.x -= move_speed
					}
					if pr == PR.RIGHT {
						image_xscale = 1
						x += move_speed
						feet.x += move_speed
					}
				}
			}
		
			else {
				player_state = PLAYER_STATE.FALLING	
			}

			break;
		
}


if player_state == PLAYER_STATE.JUMPING or player_state == PLAYER_STATE.FALLING {

	can_hit = true	

	if player_state == PLAYER_STATE.JUMPING {
		fall_frames = 0
		jump_height = 1 / (jump_counter / jump_counter_total + 0.3)
	}
	else if player_state == PLAYER_STATE.FALLING {
		if fall_frames < fall_frames_max + 1 {
			fall_frames++
			fall_speed = (fall_frames/fall_frames_max) * fall_multiplier
		}
	}
}
else {
	jump_height = 2.3
	fall_speed = 2.3
	fall_frames = 0
	can_hit = false	
}
```
And there we go! Now we have a character that can jump around but move through walls. Next we'll take care of wall collision, including wall slides.

# Wall Jumping

## Sprites

### From Craftpix
Drag into **Sprites > Player**
- spr_boy_walljump_strip5

### Make in GameMaker
Make in **Sprites > Barriers**
- **spr_barrier_wall**
   - **Size:** 2x32
   - **Color:** Magenta
   - **Origin:** Top Left

## Objects

### obj_player
Really quickly, let's update our enum:

#### enum PLAYER_STATE (Create)

```gml
enum PLAYER_STATE {

	IDLE,
	IS_ENTERING,
	RUNNING,
	JUMPING,
	FALLING,
	SLIDING_WALL, // added in Pt. 2
	
}
```

### obj_barrier_floor

Duplicate to make: 
- **obj_barrier_floor_inner**
- **obj_barrier_ceil**

### obj_barrier_floor_inner
**Sprite:** spr_barrier_floor

#### Create
*same as obj_barrier_floor*
```gml
image_alpha = 0
depth = 0

is_moving = false
```

### obj_barrier_ceil
**Sprite:** spr_barrier_floor
#### Create
*same as obj_barrier_floor*
```gml
image_alpha = 0
depth = 0

is_moving = false
```

### obj_barrier_wall
**Sprite:** spr_barrier_wall

#### Create
```gml
image_alpha = 1
side = 0
```

#### Step
```gml
if instance_exists(obj_player) {
	
	if obj_player.player_state = PLAYER_STATE.RUNNING or obj_player.player_state == PLAYER_STATE.IDLE {
		
		if collision_rectangle(bbox_left,bbox_top+4,bbox_right,bbox_bottom,obj_player,false,false) {
		
			obj_player.sprite_index = spr_boy_idle
	
			if side = 0 {
				obj_player.x -= 2
				if obj_player.player_state != PLAYER_STATE.IDLE {
					obj_player.can_move_right = false
				}
		
			}
			else if side = 1 {
				obj_player.x += 2
				if obj_player.player_state != PLAYER_STATE.IDLE {
					obj_player.can_move_left = false
				}
			
			}
		}
	}
	
	if obj_player.player_state = PLAYER_STATE.JUMPING or obj_player.player_state = PLAYER_STATE.FALLING {
		
		if collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_player,false,false) {
	
			if side = 0 {
				if collision_rectangle(bbox_left+1,bbox_top,bbox_right-1,bbox_bottom,obj_player,false,false) {
					obj_player.x-=2
				}
				else {
					obj_player.can_move_right = false
					obj_player.pr = PR.LEFT
					obj_player.image_index = 1
					obj_player.fall_frames = 0
					obj_player.player_state = PLAYER_STATE.SLIDING_WALL
				}
		
			}
			else if side = 1 {
				if collision_rectangle(bbox_left+1,bbox_top,bbox_right-1,bbox_bottom,obj_player,false,false) {
					obj_player.x+=2
				}
				else {
					obj_player.can_move_right = true
					obj_player.pr = PR.RIGHT
					obj_player.image_index = 1
					obj_player.fall_frames = 0
					obj_player.player_state = PLAYER_STATE.SLIDING_WALL
				}
			
			}
	
		}
	}
}
```

### obj_border

#### Create
We're going back to our obj_border and making the barriers for all sides. We're also setting the image_xscale to whatever xscale we drag it out to when building our room.
```gml
depth = 8

barrier_floor = instance_create_layer(x+4,y,"Instances",obj_barrier_floor)
barrier_floor.image_xscale = image_xscale * ((sprite_width - 8 ) / (image_xscale * 32))

barrier_floor_inner = instance_create_layer(x+4,y+1,"Instances",obj_barrier_floor_inner)
barrier_floor_inner.image_xscale = image_xscale * ((sprite_width - 8 ) / (image_xscale * 32))


barrier_ceil = instance_create_layer(x+4,y+sprite_height-4,"Instances",obj_barrier_ceil)
barrier_ceil.image_xscale = image_xscale * ((sprite_width - 8 ) / (image_xscale * 32))

barrier_left_wall = instance_create_layer(x+2,y+5,"Instances",obj_barrier_wall)
barrier_left_wall.image_yscale = image_yscale * ((sprite_height - 8 ) / (image_yscale * 32))

barrier_left_wall.side = 0

barrier_right_wall = instance_create_layer(x+(sprite_width-4),y+5,"Instances",obj_barrier_wall)
barrier_right_wall.image_yscale = image_yscale * ((sprite_height - 8 ) / (image_yscale * 32))
barrier_right_wall.side = 1
```

### obj_player
Now we'll return to obj_player and build out the functionality for sliding down walls and clean up some edge-case collisions.

#### Step

##### Add to COLLISION VARS
```gml
// COLLISION VARS

var feet_hitting_floor = collision_rectangle(feet.bbox_left,feet.bbox_top,feet.bbox_right,feet.bbox_bottom,obj_barrier_floor,false,false)
var feet_through_floor = collision_rectangle(feet.bbox_left,feet.bbox_top,feet.bbox_right,feet.bbox_bottom,obj_barrier_floor_inner,false,false)
var hitting_wall = collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_barrier_wall,false,false)
var hitting_ceil = collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_barrier_ceil,false,false)
```

##### Add to switch(player_state)
```gml
		case PLAYER_STATE.SLIDING_WALL:
	
		image_speed = 0.2
		sprite_index = spr_boy_walljump
		jump_counter = 0
	
		float_frames = float_frames_total
	
		if feet_hitting_floor {
			jump_counter = 0
			player_state = PLAYER_STATE.IDLE	
		}
		else if !collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_barrier_wall,false,false) {
			pr = PR.NONE
			player_state = PLAYER_STATE.FALLING
		}
		else {
			y += fall_speed / 2
			feet.y += fall_speed / 2
		}
	

		if global.A_pressed  {
			if pr == PR.RIGHT {
			
				still_falling = true
			
				y -= 5
				feet.y -= 5
				x += 8
				feet.x += 8
				can_move_left = true
			
			
				image_xscale = 1
			
				pr = PR.RIGHT
			
				player_state = PLAYER_STATE.JUMPING
					
			}
			else if pr == PR.LEFT {
			
				still_falling = true

				y -= 5
				feet.y -= 5
				x -= 8
				feet.x -= 8
				can_move_right = true
		
		
				image_xscale = -1
			
				pr = PR.LEFT
			
				player_state = PLAYER_STATE.JUMPING
			
			
			}


		
		}
	
			break;
```

Then we just have to clean up some of the logic at the end of our Step:

##### Add to end statement:
```gml
if player_state == PLAYER_STATE.JUMPING or player_state == PLAYER_STATE.FALLING or player_state == PLAYER_STATE.SLIDING_WALL{

	can_hit = true	

	if player_state == PLAYER_STATE.JUMPING {
		fall_frames = 0
		jump_height = 1 / (jump_counter / jump_counter_total + 0.3)
	}
	else if player_state == PLAYER_STATE.FALLING or PLAYER_STATE.SLIDING_WALL {
		if fall_frames < fall_frames_max + 1 {
			fall_frames++
			fall_speed = (fall_frames/fall_frames_max) * fall_multiplier
		}
	}
}
if feet_hitting_floor {
	if feet_through_floor {
		y -=2
		feet.y -=2
	}
}
```
Now all we have to do is build out our room to make sure it all works! Next time we'll deal with hurting the player!

# Hit Points
Okay, so now that we've dealt with wall collision, we'll work on another type of collision, HURT collision. You know, stuff that hurts the player and also when the player hurts enemies. We won't be getting too much into enemy types now, but it'll be helpful to build this all out at one time. This will be a longer one as we have to re-write a good bit of script to accomodate our new functionality. Let's start with getting our sprites in order, then we can move on to the hard stuff.

## Sprites

1. Make a new group called **Characters**
2. Drag the group **Player** into **Characters**
3. Make a new group inside **Characters** called **Enemies**
4. Make a group inside **Enemies** called **TvHead**
5. Drag into **Sprites > Characters > Player**
	- spr_boy_doublejump_strip**
	- spr_boy_hit_strip**
6. Drag into **Sprites > Characters > Enemies > TvHead**
	- spr_tvhead_idle_strip**
 	- spr_tvhead_hit_strip
7. Make a new group called **Traps**
8. Drag into **Sprites > Traps**
	- spr_spikes
9. Resize **spr_spikes** in the **canvas** to 16x16, bottom center alignment
## Objects
As mentioned earlier, we have a fair bit of re-writing to do. Let's start by adding some globals. The global hit_grace will be used to determine if the character is in a grace period from being hit, essentially their "iframes" state.

### obj_controller

#### Create (add)

```gml
global.hit_grace = false
global.hp = 5
global.hp_max = 5
```
### obj_player

#### enum PLAYER_STATE (Create)
We need to add some functionality to our enum as well as add some variables
```gml
enum PLAYER_STATE {

	IDLE,
	IS_ENTERING,
	RUNNING,
	JUMPING,
	FALLING,
	SLIDING_WALL,
	ATTACKED,
	HIT,
	DYING
}

// Variables added in Pt 3.
// (anywhere is fine, closer to end is preferred)

ricochet = false

can_hit = false

// Frame Animation Control
hit_frames = 0
hit_frames_total = 14
hit_loops = 3

// Grace period before player can get hurt again
hit_grace_timer = 60
hit_grace_total = 60
```

#### Step

Now here's where we do major work cleaning up our Step. We have new booleans and timers, like **hit_loops** and **ricochet** that need to be reset in their correct places. We'll also add cases for **PLAYER STATES** **HIT, ATTACKED**, and **DYING**.

```gml

	
// STEP LOGIC
if instance_exists(feet) {
	feet.x = x
	feet.y = y+15
}
	
if ricochet {
	move_speed = og_move_speed
}
else {
	move_speed = og_move_speed
}


if global.hit_grace {	
	if hit_grace_timer > 0 {
		hit_grace_timer--
	}
	else {
		image_alpha = 1
		global.hit_grace = false
		hit_grace_timer = hit_grace_total	
		
	}
	
}
	// COLLISION VARS

var feet_hitting_floor = collision_rectangle(feet.bbox_left,feet.bbox_top,feet.bbox_right,feet.bbox_bottom,obj_buffer_floor,false,false)
var feet_through_floor = collision_rectangle(feet.bbox_left,feet.bbox_top,feet.bbox_right,feet.bbox_bottom,obj_buffer_floor_inner,false,false)
var hitting_wall = collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_buffer_wall,false,false)
var hitting_ceil = collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_buffer_ceil,false,false)

// SWITCH
	
switch(pr) {

	case PR.LEFT:

		image_xscale = -1



		break;
		
	case PR.RIGHT:
	
		image_xscale = 1
	
		break;
	
}

switch(player_state) {

	case PLAYER_STATE.FALLING:

	hit_loops = 2
	roll_time = roll_time_total
	can_move_left = true
	can_move_right = true

	still_falling = false

	
		is_jumping = true

		sprite_index = spr_boy_fall
		image_speed = 0
		jump_counter = 0
		
		if feet_hitting_floor {
			player_state = PLAYER_STATE.IDLE	
		}

		else {
			y += fall_speed	
			feet. y += fall_speed
		}
		
		if ricochet {
			if pr == PR.LEFT {
				x-=move_speed 
				feet.x -= move_speed 
			}
			else if pr == PR.RIGHT {
				x += move_speed
				feet.x += move_speed
			}
		}

		if global.left_held and can_move_left and !ricochet {
			
			pr = PR.LEFT
			can_move_right = true
			x -= move_speed
			feet.x -= move_speed
			
		}
		else if global.right_held and can_move_right and !ricochet {
			pr = PR.RIGHT
			can_move_left = true
			x += move_speed
			feet.x += move_speed
		}

		break;
		
	case PLAYER_STATE.IDLE:
		float_frames = float_frames_total
		hit_loops = 2
		roll_time = roll_time_total
		ricochet = false
	
		if x < 0 {
			player_state = PLAYER_STATE.IS_ENTERING	
		}
	
		still_falling = false
		is_jumping = false
	
		sprite_index = spr_boy_idle
		image_speed = 0.3
		jump_counter = 0
		

		if can_move_left{
			if global.left_held {
				player_state = PLAYER_STATE.RUNNING
			}
		}
		if can_move_right {
			if global.right_held {
				player_state = PLAYER_STATE.RUNNING	
			}
		}
		
		
		if global.A_pressed {
			player_state = PLAYER_STATE.JUMPING	
		}
			else {
				pr = PR.NONE
			}


			break;

		
		case PLAYER_STATE.RUNNING:
		
			still_falling = false
			is_jumping = false
			
			ricochet = false
	
			sprite_index = spr_boy_run
			image_speed = 0.8
			jump_counter = 0
		
			if x < 0 {
				player_state = PLAYER_STATE.IS_ENTERING	
			}
			else if x > room_width + 5 {
				global.level_index++
				room_goto(global.levels[global.level_index])	
			}

		
			else if global.left_held and can_move_left {
			
				pr = PR.LEFT
				can_move_right = true
				x -= move_speed
				feet.x -= move_speed	
				
			}
			else if global.right_held and can_move_right {
				pr = PR.RIGHT
				can_move_left = true

				x += move_speed
					feet.x += move_speed
			}
			else {
				player_state = PLAYER_STATE.IDLE	
			}
		
			if global.A_pressed {
				player_state = PLAYER_STATE.JUMPING	
			}
		
		
			if !feet_hitting_floor {
				if !collision_rectangle(feet.bbox_left,feet.bbox_top,feet.bbox_right,feet.bbox_bottom,obj_buffer_wall,false,false) {
					if pr == PR.LEFT {
						x -= 4
						feet.x -= 4
						player_state = PLAYER_STATE.FALLING
					}
					if pr == PR.RIGHT {
						x += 4
						feet.x += 4
						player_state = PLAYER_STATE.FALLING
					}
				}
			}
			break;
			
		case PLAYER_STATE.IS_ENTERING:
	
			sprite_index = spr_boy_run
			image_speed = 0.8
			x += move_speed
			pr = PR.RIGHT
			can_move_left = true
			can_move_right = true
		
			if x > 16 {
				player_state = PLAYER_STATE.IDLE	
			}
			
			break;
		
		case PLAYER_STATE.JUMPING:
	
			if ricochet {
				sprite_index = spr_boy_doublejump
				image_speed = 0.8
				
			}
			else {
				sprite_index = spr_boy_jump
				image_speed = 0
			}
	
		
		
			is_jumping = true
		
			if global.A_held or ricochet {
				if jump_counter < jump_counter_total {
					jump_counter++
					y -= jump_height
					feet.y -= jump_height
				}
				else {
					if float_frames > 0 {
						float_frames--	
					}
					else {
						player_state = PLAYER_STATE.FALLING	
					}
				}
		
				if !still_falling{
					if global.left_held and can_move_left and !ricochet {
			
						pr = PR.LEFT
						can_move_right = true
						image_xscale = -1
						x -= move_speed
						feet.x -= move_speed
				
					}
					else if global.right_held and can_move_right and !ricochet {
						pr = PR.RIGHT
						can_move_left = true
						image_xscale = 1
						x += move_speed
					}
				}
				else {
					if pr == PR.LEFT {
						image_xscale = -1
						x -= move_speed
						feet.x -= move_speed
					}
					if pr == PR.RIGHT {
						image_xscale = 1
						x += move_speed
						feet.x += move_speed
					}
				}
			}
		
			else {
				player_state = PLAYER_STATE.FALLING	
			}
		
			if hitting_wall and jump_counter > 3 {
				fall_frames = 0
				player_state = PLAYER_STATE.SLIDING_WALL	
			}
			else if hitting_ceil {
				player_state = PLAYER_STATE.FALLING	
			}

			break;
	
		case PLAYER_STATE.SLIDING_WALL:
	
		image_speed = 0.2
		sprite_index = spr_boy_walljump
		jump_counter = 0
	
		ricochet = false
	
		float_frames = float_frames_total
	

		if feet_hitting_floor {
			jump_counter = 0
			player_state = PLAYER_STATE.IDLE	
		}
		else if !collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_buffer_wall,false,false) {
			pr = PR.NONE
			player_state = PLAYER_STATE.FALLING
		}
		else {
			y += fall_speed / 2
			feet.y += fall_speed / 2
		}
	

		if global.A_pressed  {
			if pr == PR.RIGHT {
			
				still_falling = true
			
				y -= 5
				feet.y -= 5
				x += 8
				feet.x += 8
				can_move_left = true
			
			
				image_xscale = 1
			
				pr = PR.RIGHT
			
				player_state = PLAYER_STATE.JUMPING
					
			}
			else if pr == PR.LEFT {
			
				still_falling = true

				y -= 5
				feet.y -= 5
				x -= 8
				feet.x -= 8
				can_move_right = true
		
		
				image_xscale = -1
			
				pr = PR.LEFT
			
				player_state = PLAYER_STATE.JUMPING
			}
		}
			break;
			
	case PLAYER_STATE.ATTACKED:
	
		if pr == PR.LEFT {
			still_falling = true
			image_index = -1
			pr = PR.RIGHT
			ricochet = true
			x += 4
			player_state = PLAYER_STATE.JUMPING
		}
		else if pr == PR.RIGHT {
			still_falling = true
			image_index = 1
			pr = PR.LEFT
			ricochet = true
			x -= 4
			player_state = PLAYER_STATE.JUMPING
		}
	
		break;
		
	case PLAYER_STATE.HIT:
	
		sprite_index = spr_boy_hit
		y -= 0.05
		if hit_loops > 0 {
			if hit_frames < hit_frames_total {
				hit_frames++
				image_index = floor(hit_frames/2)
			}
			else {
				hit_loops--
				hit_frames = 0
			}
		}
		else {
			if global.hp > 0 {
				image_alpha = 0.75
				global.hit_grace = true
				global.hp--
				player_state = PLAYER_STATE.FALLING
			}
			else {
				global.hit_grace = true
				player_state = PLAYER_STATE.DYING
			}
		}
	
	
		break;
			
	case PLAYER_STATE.DYING:
	
		sprite_index = spr_boy_hit

		instance_destroy(feet)
		instance_create_layer(global.initial_x,global.initial_y,"Instances",obj_player)
		instance_destroy()




		break;
}


if player_state == PLAYER_STATE.JUMPING or player_state == PLAYER_STATE.FALLING or player_state == PLAYER_STATE.SLIDING_WALL{

	can_hit = true	

	if player_state == PLAYER_STATE.JUMPING {
		fall_frames = 0
		jump_height = 1 / (jump_counter / jump_counter_total + 0.3)
	}
	else if player_state == PLAYER_STATE.FALLING or PLAYER_STATE.SLIDING_WALL {
		if fall_frames < fall_frames_max + 1 {
			fall_frames++
			fall_speed = (fall_frames/fall_frames_max) * fall_multiplier
		}
	}
}
else {
	jump_height = 2.3
	fall_speed = 2.3
	fall_frames = 0
	can_hit = false	
}

if feet_hitting_floor {
	if feet_through_floor {
		y -=2
		feet.y -=2
	}
}
```

Now with all of that out of the way, we just need to make some objects for the player to interact with. <br />
We'll make two sets of spikes, ones that pop up and ones that stay up.

Create an Object Group called **Traps**, place these objects in there

### obj_spikes_still
**Sprite:** spr_spikes

#### Create

```gml
depth = 3

image_index = 3
image_speed = 0
```

#### Step

```gml
if instance_exists(obj_player) {
	if collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_player,0,0) and !global.hit_grace {
		obj_player.player_state = PLAYER_STATE.HIT
	}
}
```


### obj_spikes_moving
**Sprite:** spr_spikes
We'll use an enum for these spikes, even though we could just use timers this will help keep the code cleaner.

#### Create
```gml
image_index = 0
image_speed = 0

enum SPIKE_STATE {

	STILL,
	RISING,
	FALLING,
	RAISED
	
}

spike_state = SPIKE_STATE.STILL

rise_fall_timer = 12
rise_fall_timer_total = 12

up_timer = 15
up_timer_total = 15
```

#### Step

```gml
switch(spike_state) {

	case SPIKE_STATE.STILL:
	
		image_index = 0
		rise_fall_timer = 0
		
		if collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_player,0,0) {
			spike_state = SPIKE_STATE.RISING	
		}
	
		break;
		
	case SPIKE_STATE.RISING:
	
		up_timer = up_timer_total
		
		if rise_fall_timer < rise_fall_timer_total {
			rise_fall_timer++
			image_index = floor(rise_fall_timer/4)
		}
		else {
			spike_state = SPIKE_STATE.RAISED	
		}
	
		break;
		
	case SPIKE_STATE.RAISED:
	
		rise_fall_timer = rise_fall_timer_total
		
		if collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_player,0,0) and !global.hit_grace {
			obj_player.player_state = PLAYER_STATE.HIT
		}
		
		if up_timer > 0 {
			up_timer--	
		}
		else {
			spike_state = SPIKE_STATE.FALLING	
		}
	
	
		break;
		
		
	case SPIKE_STATE.FALLING:
	
		up_timer = up_timer_total
		
		if rise_fall_timer > 0 {
			rise_fall_timer--
			image_index = floor(rise_fall_timer/4)
		}
		else {
			spike_state = SPIKE_STATE.STILL	
		}
	
		break;
	
}
```
Create an Object Group called **Enemies**, place these objects in there

### obj_tvhead
**Sprite:** spr_tvhead_idle
We'll just use a simple enemy as a placeholder to test our functionality.

#### Create

```gml
depth = 6

image_index = random(10)
```

#### Step

```gml
if instance_exists(obj_player) {
	if collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_player,false,false) and tvhead_state != TVHEAD_STATE.DYING and !global.hit_grace {
			if obj_player.can_hit and obj_player.y < y{
				tvhead_state = TVHEAD_STATE.DYING
				obj_player.pr = (x<obj_player.x) ? PR.LEFT : PR.RIGHT
				obj_player.player_state = PLAYER_STATE.ATTACKED
			}
			else {
				obj_player.player_state = PLAYER_STATE.HIT	
			}
	}
	
}
```

And it's ready for testing! Fingers crossed!

# Enemies
We've already started building out our enemy logic, let's go ahead and finish that off.

## Sprites
1. Drag into **Sprites > Characters > Enemies > TvHead**:
	- spr_tvhead_run
2. Create a new Group called **Canon** in **Sprites > Characters > Enemies**
3. Drag into **Canon**:
	- spr_canon_hit
	- spr_canon_attack
	- spr_canon_idle
4. Drag into **Sprites > Traps**
	- spr_canon_ball


## Objects

### obj_tvhead
We've already worked out the collision, now we just need to add some movement logic to our TvHead enemy

#### Create

```gml
depth = 6

image_index = random(10)

enum TVHEAD_STATE {

	IDLE,
	RUNNING,
	DYING
	
}

tvhead_state = TVHEAD_STATE.IDLE

idle_counter = floor(random_range(100,300))
run_counter = floor(random_range(100,500))

is_moving_left = choose(true,false)

move_speed = 1

image_xscale = choose(1,-1)

dying_frames = 0
dying_frames_total = 14
dying_loops = 3
```

#### Step

```gml
var is_hitting_wall = collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_buffer_wall,0,0)

switch(tvhead_state) {

	case TVHEAD_STATE.IDLE:
	
		sprite_index = spr_tvhead_idle
		
		image_speed = 0.3
		
		if idle_counter > 0 {
			idle_counter--	
		}
		else {
			run_counter = floor(random_range(100,500))	
			tvhead_state = TVHEAD_STATE.RUNNING
		}
	
		break;
		
	case TVHEAD_STATE.RUNNING:
	
		sprite_index = spr_tvhead_run
		image_speed = 0.8
		
		if is_moving_left {
			image_xscale = 1 
			x -= move_speed
			
		}
		else {
			image_xscale = -1
			x += move_speed
		}
		
		if is_hitting_wall {
			if is_moving_left {
				x += move_speed * 2
				is_moving_left = false
			}
			else {
				x -= move_speed * 2
				is_moving_left = true
			}
		}
		
		if !collision_rectangle(x-2,y+3,x+2,y+26,obj_buffer_floor,false,false) and !obj_buffer_floor.is_moving {
			if is_moving_left {
				x += move_speed * 2
				is_moving_left = false
			}
			else {
				x -= move_speed * 2
				is_moving_left = true
			}
		}
		
		if run_counter > 0 {
			run_counter--	
		}
		else {
			
			if is_moving_left {
				is_moving_left = false	
			}
			else {
				is_moving_left = true	
			}
			
			idle_counter = floor(random_range(100,300))
			tvhead_state = TVHEAD_STATE.IDLE
		}
	
		break;
		
	case TVHEAD_STATE.DYING:
	
		sprite_index = spr_tvhead_hit
		
		if dying_loops > 0 {
			if dying_frames < dying_frames_total {
				dying_frames++
				image_index = floor(dying_frames/2)
			}
			else {
				dying_loops--
				dying_frames = 0
			}
		}
		else {
			instance_destroy()	
		}
	
	
		break;
	
}

if instance_exists(obj_player) {
	if collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_player,false,false) and tvhead_state != TVHEAD_STATE.DYING and !global.hit_grace {
			if obj_player.can_hit and obj_player.y < y{
				tvhead_state = TVHEAD_STATE.DYING
				obj_player.pr = (x<obj_player.x) ? PR.LEFT : PR.RIGHT
				obj_player.player_state = PLAYER_STATE.ATTACKED
			}
			else {
				obj_player.player_state = PLAYER_STATE.HIT	
			}
	}
}
```

### obj_enemy_canon_ball

#### Create

```gml
moving_left = false
moving_right = false

depth = 4

create_delay = 30
```
#### Step

```gml
if moving_right {
	x += 2.5	
}
if moving_left {
	x -= 2.5	
}

if instance_exists(obj_player) {
	
	if collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_player,0,0) and !global.hit_grace {
		obj_player.player_state = PLAYER_STATE.HIT
		instance_destroy()
	}
		
}

if (x<-50 or x>room_width+50) instance_destroy()
```

### obj_canon

#### Create
```gml
depth = 7

enum CANON_STATE {

	NULL,
	IDLE,
	FIRING,
	DYING
	
}

canon_state = CANON_STATE.IDLE

shooting_timer = 0
shooting_timer_total = 35

shoot_delay = 50
shoot_delay_total = 50

idle_timer = 300
idle_timer_total = 300

dying_frames = 0
dying_frames_total = 14
dying_loops = 3

is_facing_left = true

x_push = -1
ball = noone

image_speed = 0.3
```

#### Step

```gml
if instance_exists(obj_player) {
	if obj_player.x < x {
		image_xscale = 1
		is_facing_left = true
	}
	else {
		image_xscale = -1
		is_facing_left = false
	}
}

switch(canon_state) {

	case CANON_STATE.FIRING:
	
		image_speed = 0
		idle_timer = idle_timer_total
		sprite_index = spr_canon_attack
		
		image_index = 0
		
		if shooting_timer < shooting_timer_total {
			shooting_timer++
			image_index = floor(shooting_timer/5)
		}
		else {
			if is_facing_left {
				ball = instance_create_layer(x-16,y+7,"Instances",obj_enemy_canon_ball)
				ball.moving_left = true
				canon_state = CANON_STATE.IDLE
			}
			else {
				ball = instance_create_layer(x+16,y+7,"Instances",obj_enemy_canon_ball)
				ball.moving_right = true
				canon_state = CANON_STATE.IDLE
			}
				
		}

		break;
		
	case CANON_STATE.IDLE:
	
		image_speed = 0.3
		sprite_index = spr_canon_idle
		
		shooting_timer = 0
		
		if idle_timer > 0 {
			idle_timer--	
		}
		else {
			canon_state = CANON_STATE.FIRING	
		}
	
		break;
		
		
	case CANON_STATE.DYING:
	
		sprite_index = spr_canon_hit
		
		if dying_loops > 0 {
			if dying_frames < dying_frames_total {
				dying_frames++
				image_index = floor(dying_frames/2)
			}
			else {
				dying_loops--
				dying_frames = 0
			}
		}
		else {
			instance_destroy()	
		}
	
		break;
	
}

if instance_exists(obj_player) {
	if collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_player,false,false) and canon_state != CANON_STATE.DYING and !global.hit_grace {
			if obj_player.can_hit {
				canon_state = CANON_STATE.DYING
				obj_player.player_state = PLAYER_STATE.ATTACKED
			}
			else {
				obj_player.player_state = PLAYER_STATE.HIT
			}
	}
	
}
```

# Dodging
Okay, so this one's pretty simple. I saved this one for a little bit further in to show what it looks like when you add functionality down the road. It will be easy enough to add another enum case but then we'll have to look at how it interacts with our other code and just sort of clean up little edge cases. Since we build hit_grace as a global, we'll just continue to use that.

## Sprites

### From Craftpix
Drag into **Sprites > Collectibles**
1. spr_box1_strip...
2. spr_box2_strip...
3. Due to the way the boxes are chopped up in the asset kit, we need to upload a few more sprites and arrange them in our spr_boxes.

## Objects

### obj_box

#### Create

```gml
depth = 8

image_speed = 0

sprite_index = choose(spr_box1,spr_box2)

animation_done = false
animation_timer = 20

intact = true

decay = false
```

```gml
if !intact {
	image_speed = 0.25
	
	if !animation_done {
		if animation_timer > 0 {
			animation_timer--
		}
		else {
			animation_done = true	
		}
	}
	else {
		image_index = 4
		decay = true
	}
	
}

if decay {
	if image_alpha > 0 {
		image_alpha -= 0.05
	}
	else {
		instance_destroy()	
	}
}

if instance_exists(obj_player) and intact {
	if collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_player,0,0)
	if obj_player.player_state == PLAYER_STATE.ROLLING {
		intact = false	
	}
}
```

### obj_player
We'll only be adding one part to our enum and a couple of variables for tracking a timer, but this is the last time we'll have to deal with this sort of functionality for a while so let's use this time to review the whole script to make sure we're on the same page.

#### Create

```gml
depth = 5

enum PLAYER_STATE {

	IDLE,
	IS_ENTERING,
	RUNNING,
	JUMPING,
	FALLING,
	SLIDING_WALL,
	ROLLING,
	ATTACKED,
	HIT,
	DYING,
	
}

enum PR {

	NONE,
	LEFT,
	RIGHT
		
}

pr = PR.NONE

mask_index = spr_player_mask

global.initial_x = x
global.initial_y = y

player_state = PLAYER_STATE.FALLING

can_move_left = true
can_move_right = true

fall_speed = 2.3

fall_frames = 0

fall_multiplier = 3

// TIME BEFORE REACHING MAX VELOCITY
fall_frames_max = 20

og_move_speed = 1.5
move_speed = 1.5

jump_height = 1.3

jump_counter = 0
jump_counter_total = 22

is_jumping = false

still_falling = false

ricochet = false

float_frames = 3
float_frames_total = 3

can_hit = false

hit_frames = 0
hit_frames_total = 14
hit_loops = 3

hit_grace_timer = 90
hit_grace_total = 90

roll_time = 20
roll_time_total = 20

feet = instance_create_layer(x,y+15,"Instances",obj_player_floor)
```
#### Step

```gml

	
// STEP LOGIC
if instance_exists(feet) {
	feet.x = x
	feet.y = y+15
}
	
if ricochet {
	move_speed = og_move_speed
}
else {
	move_speed = og_move_speed
}


if global.hit_grace {	
	if hit_grace_timer > 0 {
		hit_grace_timer--
	}
	else {
		image_alpha = 1
		global.hit_grace = false
		hit_grace_timer = hit_grace_total	
		
	}
	
}


	// COLLISION VARS

var feet_hitting_floor = collision_rectangle(feet.bbox_left,feet.bbox_top,feet.bbox_right,feet.bbox_bottom,obj_buffer_floor,false,false)
var feet_through_floor = collision_rectangle(feet.bbox_left,feet.bbox_top,feet.bbox_right,feet.bbox_bottom,obj_buffer_floor_inner,false,false)
var hitting_wall = collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_buffer_wall,false,false)
var hitting_ceil = collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_buffer_ceil,false,false)

// SWITCH
	
switch(pr) {

	case PR.LEFT:

		image_xscale = -1



		break;
		
	case PR.RIGHT:
	
		image_xscale = 1
	
		break;
	
}

switch(player_state) {

	case PLAYER_STATE.FALLING:

	hit_loops = 2
	roll_time = roll_time_total
	can_move_left = true
	can_move_right = true

	still_falling = false

	
		is_jumping = true

		sprite_index = spr_boy_fall
		image_speed = 0
		jump_counter = 0
		
		if feet_hitting_floor {
			player_state = PLAYER_STATE.IDLE	
		}

		else {
			y += fall_speed	
			feet. y += fall_speed
		}
		
		if ricochet {
			if pr == PR.LEFT {
				x-=move_speed 
				feet.x -= move_speed 
			}
			else if pr == PR.RIGHT {
				x += move_speed
				feet.x += move_speed
			}
		}

		if global.left_held and can_move_left and !ricochet {
			
			pr = PR.LEFT
			can_move_right = true
			x -= move_speed
			feet.x -= move_speed
			
		}
		else if global.right_held and can_move_right and !ricochet {
			pr = PR.RIGHT
			can_move_left = true
			x += move_speed
			feet.x += move_speed
		}

		break;
		
	case PLAYER_STATE.IDLE:
		float_frames = float_frames_total
		hit_loops = 2
		roll_time = roll_time_total
		ricochet = false
	
		if x < 0 {
			player_state = PLAYER_STATE.IS_ENTERING	
		}
	
		still_falling = false
		is_jumping = false
	
		sprite_index = spr_boy_idle
		image_speed = 0.3
		jump_counter = 0
		

		if can_move_left{
			if global.left_held {
				player_state = PLAYER_STATE.RUNNING
			}
		}
		if can_move_right {
			if global.right_held {
				player_state = PLAYER_STATE.RUNNING	
			}
		}
		
		
		if global.A_pressed {
			player_state = PLAYER_STATE.JUMPING	
		}
			else {
				pr = PR.NONE
			}


			break;

		
		case PLAYER_STATE.RUNNING:
		
			
	
			still_falling = false
			is_jumping = false
			
			ricochet = false
	
			sprite_index = spr_boy_run
			image_speed = 0.8
			jump_counter = 0
		
			if x < 0 {
				player_state = PLAYER_STATE.IS_ENTERING	
			}
			else if x > room_width + 5 {
				global.level_index++
				room_goto(global.levels[global.level_index])	
			}

		
			else if global.left_held and can_move_left {
			
				pr = PR.LEFT
				can_move_right = true
				x -= move_speed
				feet.x -= move_speed	
				
			}
			else if global.right_held and can_move_right {
				pr = PR.RIGHT
				can_move_left = true

				x += move_speed
					feet.x += move_speed
			}
			else {
				player_state = PLAYER_STATE.IDLE	
			}
		
			if global.A_pressed {
				player_state = PLAYER_STATE.JUMPING	
			}
			else if global.B_pressed {
				player_state = PLAYER_STATE.ROLLING	
			}
		
		
			if !feet_hitting_floor {
				if !collision_rectangle(feet.bbox_left,feet.bbox_top,feet.bbox_right,feet.bbox_bottom,obj_buffer_wall,false,false) {
					if pr == PR.LEFT {
						x -= 4
						feet.x -= 4
						player_state = PLAYER_STATE.FALLING
					}
					if pr == PR.RIGHT {
						x += 4
						feet.x += 4
						player_state = PLAYER_STATE.FALLING
					}
				}
			}
		

		
		
			break;
			
		case PLAYER_STATE.IS_ENTERING:
	
			sprite_index = spr_boy_run
			image_speed = 0.8
			x += move_speed
			pr = PR.RIGHT
			can_move_left = true
			can_move_right = true
		
			if x > 16 {
				player_state = PLAYER_STATE.IDLE	
			}
			
			break;
		
		
		
		case PLAYER_STATE.JUMPING:
	
			if ricochet {
				sprite_index = spr_boy_doublejump
				image_speed = 0.8
				
			}
			else {
				sprite_index = spr_boy_jump
				image_speed = 0
			}
	
		
		
			is_jumping = true
		
			if global.A_held or ricochet {
				if jump_counter < jump_counter_total {
					jump_counter++
					y -= jump_height
					feet.y -= jump_height
				}
				else {
					if float_frames > 0 {
						float_frames--	
					}
					else {
						player_state = PLAYER_STATE.FALLING	
					}
				}
		
				if !still_falling{
					if global.left_held and can_move_left and !ricochet {
			
						pr = PR.LEFT
						can_move_right = true
						image_xscale = -1
						x -= move_speed
						feet.x -= move_speed
				
					}
					else if global.right_held and can_move_right and !ricochet {
						pr = PR.RIGHT
						can_move_left = true
						image_xscale = 1
						x += move_speed
					}
				}
				else {
					if pr == PR.LEFT {
						image_xscale = -1
						x -= move_speed
						feet.x -= move_speed
					}
					if pr == PR.RIGHT {
						image_xscale = 1
						x += move_speed
						feet.x += move_speed
					}
				}
			}
		
			else {
				player_state = PLAYER_STATE.FALLING	
			}
		
			if hitting_wall and jump_counter > 3 {
				fall_frames = 0
				player_state = PLAYER_STATE.SLIDING_WALL	
			}
			else if hitting_ceil {
				player_state = PLAYER_STATE.FALLING	
			}

			break;
		
		case PLAYER_STATE.ROLLING:
		
			sprite_index = spr_boy_doublejump
			
			if roll_time > 0 {
				global.hit_grace = true
				if pr == PR.LEFT {
					image_xscale = -1
					x -= move_speed + 1
					roll_time--
				}
				if pr == PR.RIGHT {
					image_xscale = 1
					x += move_speed + 1
					roll_time--
				}
			}
			else {
				global.hit_grace = false
				if (feet_hitting_floor) player_state = PLAYER_STATE.IDLE; 
				else player_state = PLAYER_STATE.FALLING;
			}
		
			break;
	
		case PLAYER_STATE.SLIDING_WALL:
	
		image_speed = 0.2
		sprite_index = spr_boy_walljump
		jump_counter = 0
	
		ricochet = false
	
		float_frames = float_frames_total
	

		if feet_hitting_floor {
			jump_counter = 0
			player_state = PLAYER_STATE.IDLE	
		}
		else if !collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_buffer_wall,false,false) {
			pr = PR.NONE
			player_state = PLAYER_STATE.FALLING
		}
		else {
			y += fall_speed / 2
			feet.y += fall_speed / 2
		}
	

		if global.A_pressed  {
			if pr == PR.RIGHT {
			
				still_falling = true
			
				y -= 5
				feet.y -= 5
				x += 8
				feet.x += 8
				can_move_left = true
			
			
				image_xscale = 1
			
				pr = PR.RIGHT
			
				player_state = PLAYER_STATE.JUMPING
					
			}
			else if pr == PR.LEFT {
			
				still_falling = true

				y -= 5
				feet.y -= 5
				x -= 8
				feet.x -= 8
				can_move_right = true
		
		
				image_xscale = -1
			
				pr = PR.LEFT
			
				player_state = PLAYER_STATE.JUMPING
			
			
			}


		
		}
	
			break;
			
	case PLAYER_STATE.ATTACKED:
	
		if pr == PR.LEFT {
			still_falling = true
			image_index = -1
			pr = PR.RIGHT
			ricochet = true
			x += 4
			player_state = PLAYER_STATE.JUMPING
		}
		else if pr == PR.RIGHT {
			still_falling = true
			image_index = 1
			pr = PR.LEFT
			ricochet = true
			x -= 4
			player_state = PLAYER_STATE.JUMPING
		}
	
		break;
		
	case PLAYER_STATE.HIT:
	
		global.hit_grace = true
		sprite_index = spr_boy_hit
		y -= 0.05
		if hit_loops > 0 {
			if hit_frames < hit_frames_total {
				hit_frames++
				image_index = floor(hit_frames/2)
			}
			else {
				hit_loops--
				hit_frames = 0
			}
		}
		else {
			if global.hp > 0 {
				image_alpha = 0.75
				global.hp--
				player_state = PLAYER_STATE.FALLING
			}
			else {
				player_state = PLAYER_STATE.DYING
			}
		}
	
	
		break;
			
	case PLAYER_STATE.DYING:
	
		sprite_index = spr_boy_hit

		instance_destroy(feet)
		instance_create_layer(global.initial_x,global.initial_y,"Instances",obj_player)
		instance_destroy()




		break;
}


if player_state == PLAYER_STATE.JUMPING or player_state == PLAYER_STATE.FALLING or player_state == PLAYER_STATE.SLIDING_WALL{

	can_hit = true	

	if player_state == PLAYER_STATE.JUMPING {
		fall_frames = 0
		jump_height = 1 / (jump_counter / jump_counter_total + 0.3)
	}
	else if player_state == PLAYER_STATE.FALLING or PLAYER_STATE.SLIDING_WALL {
		if fall_frames < fall_frames_max + 1 {
			fall_frames++
			fall_speed = (fall_frames/fall_frames_max) * fall_multiplier
		}
	}
}
else {
	jump_height = 2.3
	fall_speed = 2.3
	fall_frames = 0
	can_hit = false	
}

if feet_hitting_floor {
	if feet_through_floor {
		y -=2
		feet.y -=2
	}
}
```

# Building Levels

OK, now we're in the thick of it! This one will be extremely light on coding, we just need to move some stuff around and actually build out some levels that we can test with. Let's also make some more sprites and objects to serve as room decoration.

## Sprites

1. Make a new Sprite Group called **Room Deco**
2. Duplicate **spr_blocks_master** to make **spr_squares_gray**
   - Select **Edit Image**, then **Image > Convert to Frames**
     - **Frame Width:** 48
     - **Frame Height:** 48
     - **Horizontal Pixel Offset:** 0
     - **Vertical Pixel Offset:** 64
     - Turn on **Nine-Slice**
3. Duplicate **spr_blocks_master** to make **spr_squares_brown**
   - Select **Edit Image**, then **Image > Convert to Frames**
     - **Frame Width:** 48
     - **Frame Height:** 48
     - **Horizontal Pixel Offset:** 96
     - **Vertical Pixel Offset:** 128
     - Turn on **Nine-Slice**
4. Drag **spr_squares_gray** and **spr_squares_brown** into **Room Deco**
5. Duplicate **spr_blocks_master** to make **spr_greenbox**
   - Select **Edit Image**, then **Image > Convert to Frames**
     - **Frame Width:** 32
     - **Frame Height:** 32
     - **Horizontal Pixel Offset:** 208
     - **Vertical Pixel Offset:** 80

## Rooms
- Make a room called **Rm_Start**, make sure you current rooms are named things like Rm_Level1 etc.
- Make three or four more rooms, adjusting objects, enemies, and traps to make it interesting
- Place **obj_player** on the left side and leave an opening on the right in every room *except* **Rm_Start**

## Objects

### obj_squares_gray
**Sprite:** spr_squares_gray

#### Create

```gml
depth = 9
image_speed = 0
image_index = 0
```
Duplicate to make **obj_squares_brown**

### obj_squares_brown
**Sprite:** spr_squares_brown

#### Create

```gml
depth = 9
image_speed = 0
image_index = 0
```

### obj_greenbox
**Sprite:** spr_greenbox

- Duplicate **obj_border**
- set sprite to **spr_greenbox**

### obj_initializer
No Sprite, place in Rm_Start

```gml
randomize()

window_scale = 4
window_set_size(room_width * window_scale, room_height * window_scale);
window_set_cursor(cr_none)

global.hit_grace = false
global.hp = 5
```

### obj_controller_main
We now need to go back and delete that data out of obj_controller_main so it doesn't keep resetting every time we enter a room.

#### Create
```gml
// Should be blank for right now
```

### obj_initializer (again)
So we're going to build out a little array that stores our different rooms, then we can shuffle to change the order or just change them through testing or whatever.

#### Create (full script)
```gml
randomize()

window_scale = 3
window_set_size(room_width * window_scale, room_height * window_scale);
window_set_cursor(cr_none)

global.hit_grace = false
global.hp = 5

global.levels = [Rm_Level1,Rm_Level2]
global.levels = array_shuffle(global.levels)

global.level_index = 0
```

### obj_player

Let's just really quickly highlight the part of the script that lets us go to the next room:

#### case PLAYER_STATE.RUNNING (Step switch statement)
```gml
		if x < 0 {
				player_state = PLAYER_STATE.IS_ENTERING	
			}
			else if x > room_width + 5 {
				global.level_index++ // we increase the index by 1
				room_goto(global.levels[global.level_index]) // and then go to that index
			}
```
Now what happens when we reach the end of the array? Well, ideally you wouldn't be able to exit the room to the right on the final stage, instead it would trigger an end cutscene which goes to "Rm_End", something like that.

# Collectibles
We already started building this out with our boxes, now we'll add coins and other fun stuff.

## Sprites

### From Craftpix
Drag into **Sprites > Collectibles**
- spr_coins_strip7
- spr_key_strip7
- spr_redgem_strip7
- spr_chest_locked_strip7

## Objects

### obj_initializer

#### Create (add)
```gml
global.coins = 0
```

### obj_controller

#### Create (add)
```gml
global.has_key = false
```

### obj_coin
**Sprite:** spr_coins

#### Create

```gml
depth = 4
image_speed = 0.4

coin_counter = 20
```
#### Step

```gml
var is_hitting_player = collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_player,false,false)

if is_hitting_player and coin_counter <= 0{
	global.coins++
	instance_destroy();
	
}
if coin_counter > 0 {
	coin_counter--	
}
```

### obj_box (revisited)

#### Create (add)

```gml
coin_drop = irandom(10)
```

#### Step

```gml
if !intact {
	image_speed = 0.25
	
	if !animation_done {
		if animation_timer > 0 {
			animation_timer--
		}
		// Generate a coin with odds
		else {
			if coin_drop == 7 {
				instance_create_layer(x-6,y-8,"Instances",obj_coin)
			}
			animation_done = true	
		}
	}
	else {
		image_index = 4
		decay = true
	}
	
}

if decay {
	if image_alpha > 0 {
		image_alpha -= 0.05
	}
	else {
		instance_destroy()	
	}
}

if instance_exists(obj_player) and intact {
	if obj_player.player_state == PLAYER_STATE.ROLLING {
		intact = false	
	}
}
```

### obj_key
**Sprite:** spr_key

#### Create

```gml
depth = 4
image_speed = 0.4

key_counter = 20
```

#### Step

```gml
var is_hitting_player = collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_player,false,false)


if is_hitting_player and key_counter <= 0{

	global.has_key = true
	instance_destroy();
	
}

if key_counter > 0 {
	key_counter--	
}
```

### obj_chest
**Sprite:** spr_chest

#### Create

```gml
depth = 7

image_speed = 0

locked = true

animation_done = false
animation_timer = 20

if x > room_width / 2 {
	image_xscale = -1	
}
else {
	image_xscale = 1	
}
```

#### Step

```gml
var is_hitting_player = collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_player,false,false)

if !locked {
	image_speed = 0.25
	
	if !animation_done {
		if animation_timer > 0 {
			animation_timer--
		}
		else {
			instance_create_layer(x-6,y-8,"Instances",obj_coin)
			instance_create_layer(x-3,y-8,"Instances",obj_coin)
			instance_create_layer(x+3,y-8,"Instances",obj_coin)
			animation_done = true	
		}
	}
	else {
		image_index = 4	
	}
}

if is_hitting_player and global.has_key and locked {
	if global.up_pressed {
		global.has_key = false
		locked = false
	}
}
```

### spr_redgem
**Sprite:** spr_redgem

#### Create

```gml
depth = 4
image_speed = 0.4

gem_counter = 20
```


#### Step

```gml
var is_hitting_player = collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_player,false,false)

if is_hitting_player and gem_counter <= 0{

	if global.hp < global.hp_max {
		global.hp++	
	}
	instance_destroy();
	
}

if gem_counter > 0 {
	gem_counter--	
}
```

# UI Menus

## Fonts

### fnt_menu
- Font: Tilt Neon
- Size: 12

## Sprites

Make a new Sprite Group and Object Group called **UI**
1. **spr_menu_border**
   - Duplicate spr_border
   - Color the lower portion with a filled rectangle
   - Color the top portion with the base color
   - Remove the color on the lower portion
2. **spr_healthbar**
   - **Size:** 64x8
   - Make two frames
   - Fill the first with a dark red
   - Add a filled rectangle to make a shadow and highlight
   - Fill the second frame with dark gray
3. **spr_icon_boy**
   - Duplicate spr_boy_idle
   - Erase everything but the head
   - Finish the outline along the bottom of the head
  
## Objects

### obj_btn_start
**Sprite:** spr_border

#### Create

```gml
image_xscale = 3

x = (room_width-sprite_width)/2
y = 60

draw_string = "Press Enter"
```

#### Draw

```gml
draw_self()

draw_set_font(fnt_menu)
draw_set_color(c_white)

var sw = string_width(draw_string)
var sh = string_height(draw_string)

var swx = x+(sprite_width-sw)/2
var swh = y+(sprite_height-sh)/2

draw_text(swx,swh,draw_string)
```

Duplicate to make **obj_btn_paused**

### obj_btn_paused
**Sprite:** spr_border

#### Create

```gml
image_xscale = 3

x = (room_width-sprite_width)/2
y = 60

draw_string = "Paused"
```

#### Draw

```gml
draw_self()

draw_set_font(fnt_menu)
draw_set_color(c_white)

var sw = string_width(draw_string)
var sh = string_height(draw_string)

var swx = x+(sprite_width-sw)/2
var swh = y+(sprite_height-sh)/2

draw_text(swx,swh,draw_string)
```

We'll leave this for now

### obj_menu_border
**Sprite:** spr_menu_border

#### Create

```gml
depth = 2

image_xscale = (room_width/32)
```

### obj_healthbar_highlight
**Sprite:** spr_healthbar

#### Create

```gml
image_speed = 0

depth = 0

target = global.hp / global.hp_max
hscale = target

tl = 0.1
```

#### Step

```gml
image_xscale = hscale

target = global.hp / global.hp_max

if target < hscale + tl or target > hscale - tl {
	hscale = lerp(target,hscale,0.5)	
}
```

### obj_healthbar
**Sprite:** spr_healthbar

#### Create

```gml
image_speed = 0
image_index = 1

depth = 1

healthbar = instance_create_layer(x,y,"Instances",obj_healthbar_highlight)

```

### obj_icon_boy
**Sprite:** spr_icon_boy

#### Create

```gml
depth = 0
```

### obj_icon_coin
**Sprite:** spr_coins

#### Create

```gml
image_speed = 0
depth = 0
```

#### Draw

```gml
draw_self()

draw_set_font(fnt_menu)
draw_set_color(c_white)
draw_text(x+10,y-10,"x" + string(global.coins))
```

### obj_icon_key
**Sprite:** spr_key

#### Create

```gml
image_speed = 0
image_alpha = 0
```

#### Step

```gml
//image_alpha = global.has_key Consider we could do it like this...

if global.has_key {
	if image_alpha < 1 {
		image_alpha += 0.2
	}
}
else {
	if image_alpha > 0 {
		image_alpha -= 0.2
	}
}
```

### obj_controller_main

#### Create

```gml
global.has_key = false

global.paused = false

instance_create_layer(0,0,"Instances",obj_menu_border)

head_icon = instance_create_layer(30,16,"Instances",obj_icon_boy)
healthbar = instance_create_layer(47,14,"Instances",obj_healthbar)

key_icon = instance_create_layer(230,15,"Instances",obj_icon_key)
coin_icon = instance_create_layer(250,16,"Instances",obj_icon_coin)

pause_btn = noone
```

#### Step

```gml
if global.start_pressed {
	if !global.paused {
		pause_btn = instance_create_layer(-100,-100,"Instances",obj_btn_paused)
		global.paused = true
	}
	else {
		instance_destroy(pause_btn)
		global.paused = false
	}
}
```

# Bosses

## Sprites

#### From Craftpix

1. spr_frog_attack
2. spr_frog_hit
3. spr_frog_idle
4. spr_frog_run
Duplicate **spr_frog_attack** to make **spr_frog_highlight**
	- Recolor everything white except the tongue color and the outline color
	- Change all origins to 48x35

#### In GameMaker
1. spr_frog_cmask
   - **Size:** 20x24
   - **Color:** Light Blue, 50% Alpha
2. spr_frog_hmask
   - **Size:** 45x8
   - **Color:** Magenta, 50% Alpha
  
## Objects

### obj_frog_cmask
**Sprite:** spr_frog_cmask

#### Create

```gml
image_alpha = 0
```

### obj_frog
**Sprite:** spr_frog_idle

#### Create

```gml
depth = 7

image_index = random(10)

enum FROG_STATE {

	IDLE,
	RUNNING,
	HIT,
	ATTACKING,
	DYING
	
}

frog_state = FROG_STATE.IDLE

is_moving_left = true

move_speed = 1

image_xscale = 1

dying_frames = 0
dying_frames_total = 14
dying_loops = 3

hp = 5

attack_frames = 0
attack_frames_total = 47

attack_delay = 24
attack_delay_total = 24

cmask = instance_create_layer(x,y,"Instances",obj_frog_cmask)
hmask = noone
highlight = noone
```

### obj_frog_hmask
**Sprite:** spr_frog_hmask

#### Create

```gml
delay = 18
is_left = true
image_alpha = 0
```
#### Step

```gml
var hitting_player = collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_player,0,0)

if delay > 0 {
	delay--	
}
else {
	if is_left {
		if image_xscale > 0 {
			image_xscale -= 0.25
		}
	}
	else {
		if image_xscale < 0 {
			image_xscale += 0.25	
		}
	}
}

if hitting_player and !global.hit_grace {
	obj_player.player_state = PLAYER_STATE.HIT	
}
```

### obj_frog_highlight

#### Create

```gml
image_speed = 0

depth = 6

image_blend = c_red

image_alpha = 0
image_index = 1

attack_frames = 0
```
#### Step

```gml
if image_alpha < 0.4 {
	image_alpha += 0.05	
}
image_index = floor(attack_frames/4)

if instance_exists(obj_frog) {
	image_xscale = obj_frog.image_xscale
	
	if (obj_frog.frog_state != FROG_STATE.ATTACKING) instance_destroy()
}
```

### obj_frog (again)

#### Step

```gml
var is_hitting_wall = collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_buffer_wall,0,0)
var is_hitting_floor = collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_buffer_floor,0,0)


cmask.x = x
cmask.y = y

if !global.paused {
	switch(frog_state) {

		case FROG_STATE.IDLE:
	
			sprite_index = spr_frog_idle
		
			image_speed = 0.3
			
			if instance_exists(obj_player) {
				if obj_player.y >= y-10 {
					frog_state = FROG_STATE.RUNNING
				}
			}
	
			break;
		
		case FROG_STATE.RUNNING:
		
			dying_loops = 3
			attack_frames = 0
	
			sprite_index = spr_frog_run
			image_speed = 0.8
		
			if is_moving_left {
				image_xscale = 1 
				x -= move_speed
				cmask.x -= move_speed
			
			}
			else {
				image_xscale = -1
				x += move_speed
				cmask.x += move_speed
			}
		
			if is_hitting_wall or !is_hitting_floor {
				if is_moving_left {
					x += move_speed * 2
					cmask.x += move_speed * 2
					is_moving_left = false
				}
				else {
					x -= move_speed * 2
					cmask.x -= move_speed * 2
					is_moving_left = true
				}
			}
			else {
				if instance_exists(obj_player) {
					if x < obj_player.x {
						is_moving_left = false
						if obj_player.x - x < 50 and !attack_delay {
							image_index = 0
							frog_state = FROG_STATE.ATTACKING	
						}
					}
					else if x > obj_player.x {
						is_moving_left = true
						if x - obj_player.x < 50 and !attack_delay {
							image_index = 0
							frog_state = FROG_STATE.ATTACKING	
						}
					}
				}
			}
			
			if attack_delay > 0 {
				attack_delay--	
			}
			else {
				attack_delay = 0	
			}
	
			break;
			
		case FROG_STATE.ATTACKING:
		
			attack_delay = attack_delay_total
			sprite_index = spr_frog_attack
			image_speed = 0
			image_index = floor(attack_frames/4)

			
			
			if attack_frames < attack_frames_total {
				if instance_exists(highlight) {
					highlight.attack_frames = attack_frames
				}
				else {
					highlight = instance_create_layer(x,y,"Instances",obj_frog_highlight)
					if !is_moving_left {
						highlight.image_xscale = -1	
					}
				}
				attack_frames++
			if attack_frames >= 20 and !instance_exists(hmask) {
				hmask = instance_create_layer(x,y,"Instances",obj_frog_hmask)	
				if !is_moving_left {
					hmask.image_xscale = -1
					hmask.is_left = false
				}
			}
			}

			else {
				instance_destroy(hmask)
				instance_destroy(highlight)
				frog_state = FROG_STATE.RUNNING	
			}
			
			break;
		
		case FROG_STATE.HIT:
	
			sprite_index = spr_frog_hit
		
			if dying_loops > 0 {
				if dying_frames < dying_frames_total {
					dying_frames++
					image_index = floor(dying_frames/2)
				}
				else {
					dying_loops--
					dying_frames = 0
				}
			}
			else {
				if hp > 0 {
					hp--
					frog_state = FROG_STATE.RUNNING
				}
				else {
					instance_destroy(cmask)
					instance_destroy()	
				}
			}
	
	
			break;
	
	}

	if instance_exists(obj_player) and instance_exists(cmask) {
		if collision_rectangle(cmask.bbox_left,cmask.bbox_top,cmask.bbox_right,cmask.bbox_bottom,obj_player,false,false) and frog_state != FROG_STATE.DYING and !global.hit_grace {
				if obj_player.can_hit {
					if frog_state != FROG_STATE.ATTACKING {
						frog_state = FROG_STATE.HIT	
					}
					obj_player.pr = (x<obj_player.x) ? PR.LEFT : PR.RIGHT
					obj_player.player_state = PLAYER_STATE.DID_HIT
				}
				else {
					obj_player.player_state = PLAYER_STATE.HIT	
				}
		}
	
	}
}
else {
	image_speed = 0	
}

if instance_exists(obj_player) {
	if obj_player.player_state == PLAYER_STATE.DYING {
		frog_state = FROG_STATE.IDLE	
	}
}
```
