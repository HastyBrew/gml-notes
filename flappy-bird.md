# Flappy Bird

## ROOM
Size: 800x1200
Color: Light Blue

## FONT

**fnt_score:** Jersey 20, size 120

## SPRITES

### spr_grass
**Size:** 50x50 <br />
**Color:** Dark Green <br />

### spr_cloud
**Size:** 200x100 <br />
Make some white circles of different sizes to make clouds. <br />
*Cartoon clouds have long, big circles on bottom and smaller circles on top.*

### spr_bird
**Size:** 150x150
1. Draw a yellow circle for the head
2. Draw a long oval for the body
3. Give it a little tail
4. Make an eye with a black circle and two white dots
5. Make a beak with a slightly larger orange line
6. Copy and paste to make five frames
7. Draw triangles around the center of the body, with a sharp angle pointing downward moving to upward
8. Duplicate the frames and rearrange so the bird appears to be flapping its wings

### spr_pipe
**Size:** 150x800
1. Make a full-length rectangle, width inset
2. Shade with darker rectangles near the edges and lighter toward the middle
3. Erase the top
4. Draw a dark oval near the top
5. Draw a dark square that lines up with the edges of the oval
6. Make a lighter oval for the top of the pipe
7. Make a darker oval in the middle for the hole in the pipe

## OBJECTS

### obj_controller
**Sprite:** none

#### Create

```gml
randomize();

global.moving = true

cloud_timer = 20

pipe_timer = 200
pipe_timer_total = 200

pipe_buffer = 200

global.points = 0
```
Drop in Room

### obj_grass
**Sprite:** spr_grass

#### Create

```gml
depth = 5
```
Drop in Room, drag to fit the bottom

### obj_cloud
**Sprite:** spr_cloud

#### Create

```gml
depth = 10

move_speed = floor(random_range(5,10))

image_xscale = random_range(0.8,1.2)
image_yscale = random_range(0.8,1.2)

image_alpha = random_range(0.1,0.7)
```

#### Step

```gml
depth = 7

move_speed = 10
```

### obj_pipe
**Sprite:** spr_pipe

#### Create

```gml
depth = 10

move_speed = floor(random_range(5,10))

image_xscale = random_range(0.8,1.2)
image_yscale = random_range(0.8,1.2)

image_alpha = random_range(0.1,0.7)
```

#### Step

```gml
if global.moving {
	if x > -200 {
		x -= move_speed	
	}
	else {
		instance_destroy()	
	}
}
```

### obj_bird
**Sprite:** spr_bird

#### Create

```gml
depth = 3

image_speed = 0


flap_delay = 5
flap_delay_total = 5

flap_timer = 10
flap_timer_total = 10

drop_speed = 10

frame = 0

enum BIRD_STATE {
	
	DROPPING,
	FLAPPING,
	DYING
	
}

bird_state = BIRD_STATE.DROPPING
```

#### Step

```gml
switch(bird_state) {
	
	case BIRD_STATE.DROPPING:
	
		image_speed = 0
		image_index = 0
		image_angle = -10
		
		y += drop_speed
	
		break;
		
	case BIRD_STATE.FLAPPING:
	
		image_speed = 1
		image_angle = 10
		
		y -= drop_speed
		
		if flap_timer > 0 {
			flap_timer--	
		}
		else {
			flap_timer = flap_timer_total
			bird_state = BIRD_STATE.DROPPING
		}
	
		break;
		
	case BIRD_STATE.DYING:
	
		y += drop_speed * 2
		x -= drop_speed / 2
		if y > _height + 50 {
			room_restart()	
		}
		image_angle += 30
	
	
		break;
}

if flap_delay > 0 {
	flap_delay--	
}
else {
	flap_delay = 0	
}
```

#### Key Press - Space

```gml
if !flap_delay and bird_state != BIRD_STATE.DYING {
	bird_state = BIRD_STATE.FLAPPING
	flap_timer = flap_timer_total
	flap_delay = flap_delay_total
}
```

#### Collision > obj_grass

```gml
bird_state = BIRD_STATE.DYING
global.moving = false
```

#### Collision > obj_pipe

```gml
bird_state = BIRD_STATE.DYING
global.moving = false
```

### obj_controller (again)

#### Step

```gml
// CLOUD SPAWNER

if cloud_timer > 0 {
	cloud_timer--	
}
else {
	instance_create_layer(room_width+200,random_range(100,400),"Instances",obj_cloud)	
	cloud_timer = random_range(50,100)
}

// PIPE SPAWNER

if pipe_timer > 0 {
	pipe_timer--	
}
else {
	var tube_top = instance_create_layer(room_width+200,random_range(400,800),"Instances",obj_pipe)
	var tube_bottom = instance_create_layer(room_width+200,tube_top.y,"Instances",obj_pipe)
	tube_bottom.image_yscale = -1
	tube_bottom.y -= pipe_buffer
	pipe_timer += pipe_timer_total
}
```

#### Draw

```gml
draw_set_font(fnt_score)
draw_set_color(c_red)
draw_text(room_width-200,50,string(floor(global.points)))
```

