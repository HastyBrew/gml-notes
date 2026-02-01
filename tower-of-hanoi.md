# Tower of Hanoi Game

## ROOM
Size: 1200x800
Color: Black

## SPRITES

*Bottom center origin for all*

### spr_ring
**Size:** 100x50 <br />
**Color:** White <br />
Make two circles at the very edges of the rectangle, then fill with a rectangle to make a pill shape<br />
Turn on Nine-Slice, inset by 21 on each side

### spr_base
**Size:** 300x50 <br />
**Color:** Gray <br />
Fill with gray, then use smaller rectangles to highlight and shade the top and bottom

### spr_rod
**Size:** 50x400 <br />
**Color:** Gray <br />
Fill with gray, then use smaller rectangles to shade the sides and highlight the middle

## OBJECTS

### obj_base
**Sprite:** spr_base

#### Create

```gml
depth = 10
```

### obj_rod
**Sprite:** spr_rod

#### Create

```gml
depth = 11
rod_num = -1
```

### obj_ring
**Sprite:** spr_ring

#### Create

```gml
depth = 2

ring_num = -1
rod_num = -1

last_ring = -1

enum RING_STATE {

	STILL,
	DRAGGING,
	HIGHLIGHT
	
}

ring_state = RING_STATE.STILL

can_move = false
hitting_rod = noone
```

#### Step

```gml
if rod_num > -1 and ring_num > -1 {
	hitting_rod = collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_rod,0,0)
	can_move = (global.orders[rod_num][(array_length(global.orders[rod_num])-1)]==ring_num)

}


switch(ring_state) {
	
	case RING_STATE.STILL:
		depth = 2
		image_alpha = 1
		
		if point_in_rectangle(mouse_x,mouse_y,bbox_left,bbox_top,bbox_right,bbox_bottom) {
			if mouse_check_button_pressed(mb_left) and can_move {
					
				dfx = mouse_x - x
				dfy = mouse_y - y
				
				highlight = instance_create_layer(x,y,"Instances",obj_ring)
				highlight.ring_state = RING_STATE.HIGHLIGHT
				highlight.image_xscale = image_xscale
				highlight.image_blend = image_blend
				ring_state = RING_STATE.DRAGGING
			}
		}
	
		break;
		
	case RING_STATE.DRAGGING:

		image_alpha = 1
		if mouse_check_button(mb_left) {
			x = mouse_x - dfx
			y = mouse_y - dfy
		}
		else {
			if !hitting_rod or hitting_rod.rod_num == rod_num {
				x = highlight.x
				y = highlight.y
				instance_destroy(highlight)
				ring_state = RING_STATE.STILL
			}
			else if hitting_rod {
				
					var cr = rod_num
					var nr = hitting_rod.rod_num
				
					var crl = array_length(global.orders[cr]) -1
					var nrl = array_length(global.orders[nr]) -1
				
					if global.orders[nr] != [] and nrl > -1 {
						last_ring = global.orders[nr][nrl]
					}
					else {
						last_ring = -1	
					}

					if last_ring < ring_num {
				
						array_delete(global.orders[cr],crl,1)
						array_push(global.orders[nr],ring_num)
						
						nrl = array_length(global.orders[nr]) -1
						
						x = global.pos_x_arr[nr]
						y = global.pos_y_arr[nrl]
				
						rod_num = nr
				
						instance_destroy(highlight)
						ring_state = RING_STATE.STILL
					}
					else {
						x = highlight.x
						y = highlight.y
						instance_destroy(highlight)
						ring_state = RING_STATE.STILL
					}		
			}
		}
	
		break;
		
	case RING_STATE.HIGHLIGHT:
		depth = 3
		image_alpha = 0.5
	
		break;
	
}
```

### obj_controller
**Sprite:** none

#### Create

```gml
// GAME DATA

difficulty = 7

color_arr = [c_red,c_orange,c_yellow,c_green,c_blue,c_maroon,c_purple]
length_arr = [2.5,2.25,2,1.75,1.5,1.25,1]

global.pos_y_arr = [750,700,650,600,550,500,450]
global.pos_x_arr = [250,600,950]


// BUILD BASES

base_buffer = 350
base_x = 250
base_y = 800

global.orders = [[],[],[]]

bases = []
rods = []

for (var i =0;i<3;i++) {
	
	bases[i] = instance_create_layer(global.pos_x_arr[i],base_y,"Instances",obj_base)
	rods[i] = instance_create_layer(global.pos_x_arr[i],base_y,"Instances",obj_rod)
	rods[i].rod_num = i
	
}

// BUILD RINGS

rings = []

for (var i=0;i<difficulty;i++) {
	rings[i] = instance_create_layer(global.pos_x_arr[0],global.pos_y_arr[i],"Instances",obj_ring)
	rings[i].image_blend = color_arr[i]
	rings[i].image_xscale = length_arr[i]
	rings[i].ring_num = i
	rings[i].rod_num = 0
	array_push(global.orders[0],i)
	
}
```

#### Step

```gml
if global.orders[2] != [] {
	if array_length(global.orders[2]) >= difficulty {
		room_restart()	
	}
}
```
