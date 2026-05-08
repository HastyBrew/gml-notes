# Peg Solitaire Game Notes

## **ROOM**

**Size:** 800x800

 **Background color: Royal Purple**

## **SPRITES**

### spr_peg

**Size:** 50x50
- Starting at center, draw an inset circle
- Copy/Paste to make two frames
- Color the first frame light blue and the second light gray
- Add highlights to both

## **OBJECTS**

### obj_controller

```gml
if keyboard_check_pressed(vk_escape) {
	room_restart()	
}

if instance_number(obj_peg_full) <= 1 {
	room_restart()	
}
```

Drag into room

### obj_peg_full
**Sprite:** spr_peg <br /><br />
Add the pegs to the room like this:

```code

          o  o  o
          o  o  o
    o  o  o  o  o  o  o
    o  o  o     o  o  o
    o  o  o  o  o  o  o
          o  o  o
          o  o  o

```

### obj_peg_empty
**Sprite:** spr_peg

#### **Create**

```gml
depth = 2

image_speed = 0
image_index = 1
```

Add to the middle of the cross

### obj_peg_full (again)

#### Create

```gml
depth = 10

image_speed = 0
image_index = 0

enum PS {

	STILL,
	DRAGGING
	
}

ps = PS.STILL

move_speed = 5

dfx = 0
dfy = 0

to = noone

function check_jump(x1,y1,x2,y2) {
	var o = noone
	var d = sprite_width*4
	var t = 0
	
	if x1 == x2 and y1 == y2 {
		o = noone
	}
	else if abs(x1-x2) < d+1 and abs(y1-y2) < d+1 {
		if y1 == y2 {
			t = (x1 + x2) / 2
			o = instance_position(t,y1,obj_peg_full)
		}
		if x1 == x2 {
			t = (y1+y2) / 2
			o = instance_position(x1,t,obj_peg_full)
		}
	}
	
	return o

}

ep = instance_create_layer(x,y,"Instances",obj_peg_empty)
```

#### **Step**

```gml
te = collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_peg_empty,0,0)
tf = collision_rectangle(bbox_left,bbox_top,bbox_right,bbox_bottom,obj_peg_full,0,1)

switch(ps) {
	
	case PS.STILL:
	
		to = noone
		image_alpha = 1
	
		if point_in_rectangle(mouse_x,mouse_y,bbox_left,bbox_top,bbox_right,bbox_bottom) {
			if mouse_check_button_pressed(mb_left) {
					dfx = mouse_x-x
					dfy = mouse_y-y
					ps = PS.DRAGGING
				}
		}
	
		break;
	
	case PS.DRAGGING:
	
		image_alpha = 0.9
	
		if mouse_check_button(mb_left) {
			x = mouse_x - dfx
			y = mouse_y - dfy
		}
		else {
			if te {
				to = check_jump(ep.x,ep.y,te.x,te.y)
				if to != noone and !tf {
					x = te.x
					y = te.y
					ep = te
					instance_destroy(to)
					ps = PS.STILL
				}
				else {
					x = ep.x
					y = ep.y
					ps = PS.STILL
				}
			}
			else {
				x = ep.x
				y = ep.y
				ps = PS.STILL
			}
			
		}
	
		break;	
}
```

