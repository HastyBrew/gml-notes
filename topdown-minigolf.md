# Topdown Mini Golf

## ROOMS

### Room1
**Size:** 1200x800 <br />
**Background Color:** #00AD00 <br />
**Grid Space:** 25x25 <br />


## SPRITES

### spr_ball
**Size:** 25x25 <br />
**Origin:** Middle Center <br />
- Use the open circle tool with a medium gray along the boundaries
- Fill with light gray
- Use white to add highlights
- Set Collision Mask to Ellipse

### spr_hole
**Size:** 30x30 <br />
**Origin:** Middle Center <br />
- Use the open circle tool with a dark green along the boundaries
- Fill with black
- Set Collision Mask to Ellipse
- Manually bring in the Collision Mask

### spr_arrow
**Size:** 25x40 <br />
**Origin:** Bottom Center <br />
- Start with a 25x25 sprite
- Use a medium gray line tool to draw lines from the top center to the left middle
- Repeat for the right side
- Draw a line down
- Resize Canvas to 25x40 adding toward Bottom Center (move it up)

### spr_wall
**Size:** 25x25 <br />
**Origin:** Top Left <br />
**Color:** Dark Green <br />

### spr_bar
**Size:** 25x25 <br />
**Origin:** Bottom Center <br />
- With a Size 2 Brush, use the open rectangle tool to draw a magenta border
- Add two blank frames
- Color the 2nd frame light blue
- Color the 3rd frame medium gray
- Activate Nine-Slicing and draw the edges to the inside of the magenta border
- Change the magenta color to dark gray


## OBJECTS

### obj_initializer
**Sprite:** none

#### Create

```gml
global.room_index = 0
global.rooms = [Room1,Room2,Room3]
```
Drop in Room 1

### obj_wall
**Sprite:** spr_wall

#### Create

```gml
depth = 10
```
- Drop in Room, drag out width to form bottom barrier.
- Copy to make top barrier
- Rotate to make left and right barriers

### obj_hole
**Sprite:** spr_hole

#### Create

```gml
depth = 7
```

### obj_arrow
**Sprite:** spr_arrow

#### Create

```gml
depth = 6
```


### obj_ball
**Sprite:** spr_ball

#### Create

```gml
depth = 5

enum GAME_STATE {

	CHOOSING_DIR,
	CHOOSING_POWER,
	MOVING,
	NEXT_ROUND
	
}

game_state = GAME_STATE.CHOOSING_DIR

scale = 1

arrow = instance_create_layer(x,y,"Instances",obj_arrow)

speed_adjust = 0
speed_max = 10
speed_rise_rate = 0.2
speed_rising = true

fric = 0.02

round_won = false
win_delay = 20

bar_x = 0
```

#### Step

```gml
image_xscale = scale 
image_yscale = scale

if x > room_width - 100 {
	bar_x = x - 40	
}
else {
	bar_x = x + 40	
}

if game_state != GAME_STATE.CHOOSING_DIR {
	arrow.image_alpha = 0	
}
else {
	arrow.image_angle = point_direction(x, y, mouse_x, mouse_y) + 90
	arrow.x = x
	arrow.y = y
	arrow.image_alpha = 1
}

switch(game_state) {
	
	case GAME_STATE.CHOOSING_DIR:
		speed_adjust = 0
		if mouse_check_button_released(mb_left) {
			direction = arrow.image_angle + 90	
			game_state = GAME_STATE.CHOOSING_POWER
		}
	
		break;
	
	case GAME_STATE.CHOOSING_POWER:
	
		if speed_rising {
			if speed_adjust < speed_max {
				speed_adjust += speed_rise_rate
			}
			else {
				speed_rising = false	
			}
		}
		else {
			if speed_adjust > 0 {
				speed_adjust -=  speed_rise_rate
			}
			else {
				speed_rising = true	
			}
		}
		
		if mouse_check_button_released(mb_left) {
			speed = speed_adjust
			game_state = GAME_STATE.MOVING
		}
	
		break;
		
	case GAME_STATE.MOVING:	
	
		if speed > 0 {
			speed -= fric	
		}
		else {
			game_state = GAME_STATE.CHOOSING_DIR
		}

	
		break;
		
	case GAME_STATE.NEXT_ROUND:
	
		speed = 0

		if instance_exists(obj_hole) {
			x = obj_hole.x
			y = obj_hole.y
		}
		
		if scale > 0.7 {
			scale -= 0.1	
		}
		else {
			image_alpha = 0
		}
		
		if win_delay > 0 {
		win_delay--	
		}
		else {
			global.room_index++
			room_goto(global.rooms[global.room_index])
		}
	
		break;

	
}
```

#### Draw

```gml
draw_self()


if game_state == GAME_STATE.CHOOSING_POWER {
	
	var yscale = 4
	var bar_y = y + 40

	// Draw Bar Background
	draw_sprite_ext(spr_bar,2,bar_x,bar_y,1,yscale,0,-1,1)
	
	var y_adjust = yscale*(speed_adjust/speed_max)

	// Draw Bar
	draw_sprite_ext(spr_bar,1,bar_x,bar_y,1,y_adjust,0,-1,1)

	// Draw Bar Overlay
	draw_sprite_ext(spr_bar,0,bar_x,bar_y,1,yscale,0,-1,1)
	
}
```

#### Collision > obj_wall

```gml
direction = (2 * other.image_angle) - direction
```

#### Collision > obj_hole

```gml
if speed < 5 {
	game_state = GAME_STATE.NEXT_ROUND
}
```

## ROOMS (again)

- Drop the ball and hole in the room, make a little course
- Duplicate it and name it Room2, remove obj_initializer and change the course
- Duplicate and name it Room3, remove everything from this room



