# Prank Test Game
This is a prank Hand-Eye Coordination Game made for April Fool's 2026. If you know the trick, the test is fair with a slight bias towards winning, but if you don't know the trick, you can never win. Fun!

## SPRITES

### spr_bg
- **Size:** 50x50
-  **Colors:** #7F94FF (Dark), #99AAFF (Light)
-  **Origin:** Top Left

1. Starting at Center (25x25), draw a closed rectangle with Dark color over Top Left and Bottom Right quadrants
2. Fill the other areas with Light Color
3. Draw a single Light pixel at Center

### spr_border
- **Size:** 50x50
-  **Colors:** #000066 (Dark Blue), #E3FFFF (Light Blue)
-  **Origin:** Top Left

1. Select a #2 Brush with Dark Blue
2. Use the open rectangle tool to draw a border
3. Fill the middle with Light Blue
4. Turn on Nine-Slice

### spr_bar
Duplicate spr_border

- **Colors:** Dark Gray and Light Gray

1. Replace Dark Blue with Dark Gray
2. Replace Light Blue with Light Gray

### spr_green
- **Size:** 40x50
-  **Colors:** Dark Gray, Light, Middle, and Dark Green
-  **Origin:** Top Center

1. Fill the square with Dark Green
2. Draw a Middle Green closed rectangle between 20-39 pixels
3. Draw a Light Green closed rectangle between 18-22 pixels
4. Draw a Dark Gray (same as spr_bar) horizontal line with #2 thickness on the top and bottom

### spr_trick
- **Size:** 25x25
-  **Colors:** Green, 50% Alpha
-  **Origin:** Top Center

### spr_cursor
- **Size:** 10x60
-  **Colors:** Red
-  **Origin:** Top Center

1. Create a red closed circle on the top and bottom
2. Connect them with a closed rectangle

### spr_btn
- **Size:** 150x100
-  **Colors:** Dark, Middle, and Light Red
-  **Origin:** Middle Center

1. Create a closed shallow ellipse of Dark Red
2. Create a closed rectangle at the edges of the ellipse
3. Create a rounder ellipse at the top of the rectangle with Medium Red
4. Create another ellipse on top of the last one
5. Copy and paste another frame
6. Using the rectangle selector, move the top of the button into the lower portion, shrinking it vertically


## FONTS

### fnt_h1
**Montserrat Black**, 45
### fnt_h2 
**Montserrat Black**, 30
### fnt_p 
**Arial Bold**, 15

## ROOMS

- **Size:** 1200x800
- **Background:** spr_bg *tile horizontally and vertically*

Duplicate to Make:

1. Rm_Start
2. Rm_Main
3. Rm_End

## OBJECTS

### obj_controller_main
Drop in Rm_Main, leave for now

### obj_controller_end
Drop in Rm_End, leave for now

### obj_initializer
Depending on your screen size you might want to change the window_scale. It makes it easy for me for testing so we'll leave that logic in.

#### Create

```gml
randomize()

window_scale = 2

window_set_size(room_width*window_scale,room_height*window_scale)

global.can_win = false
global.points = 0

global.round = 0

title = "HAND-EYE COORDINATION TEST"

header = "CLICK TO CONTINUE"

body = "This is not an official test. This test is not sanctioned by any governing body. Use this test is at your own risk. Caveat Emptor."
```

### Draw GUI

```gml
draw_set_color(c_black)

var _y = 100

draw_set_font(fnt_h2)
var sw = string_width(title)
var swx = (room_width-sw)/2

draw_text(swx,_y,title)

_y += 150

sw = string_width(header)
swx = (room_width-sw)/2

draw_text(swx,_y,header)

_y += 200

draw_set_font(fnt_p)
draw_text_ext(50,_y,body,28,700)
```
Drop into Rm_Start

### obj_border
**Sprite:** spr_border

#### Create

```gml
depth = 10
```

### POSITIONS

#### In Rm_Start
1. **Position:** 25x75, **Scale:** 15x2
2. **Position:** 150x225, **Scale:** 10x2
3. **Position:** 25x425, **Scale:** 15x2

#### In Rm_Main
1. **Position:** 75x75, **Scale:** 6x2
2. **Position:** 425x75, **Scale:** 6x2
3. **Position:** 50x425, **Scale:** 15x2.5

#### In Rm_End
1. **Position:** 75x50, **Scale:** 13x2
2. **Position:** 75x200, **Scale:** 6.5x3
3. **Position:** 450x200, **Scale:** 5.5x2
4. **Position:** 75x400, **Scale:** 13x3

### obj_trick
**Sprite:** spr_trick

#### Create

```gml
image_alpha = 0
```

#### Step
```gml
if mouse_check_button_pressed(mb_left) {
	if point_in_rectangle(mouse_x,mouse_y,bbox_left,bbox_top,bbox_right,bbox_bottom) {
		global.can_win = true
		room_goto(Rm_Main)
	}
	else {
		room_goto(Rm_Main)	
	}
}
```
Drop obj_trick somewhere sneaky in Rm_Start

### obj_controller_main (again)

#### Create

```gml
global.bar_active = true
global.released = false
global.grace_timer = 10

global.current_points = 0

room_timer = 120

round_info = ""

rules_info = "Release the Button when the Red Area is in the middle of the Green Area. When you are finished, click again to enter the next round. Your score will be averaged after 5 rounds."

score_title = "Score: "
score_info = ""

score_color = c_red
```

#### Step

```gml
if global.grace_timer > 0 {
	global.grace_timer--	
}
else {
	global.grace_timer = 0	
}


round_info = "ROUND: " + string(global.round) + "/5"

if global.bar_active = false {
	score_info = string(global.current_points)	
}

if global.released {
	if mouse_check_button_released(mb_left) and !global.grace_timer {
		if global.round < 5 {
			global.round++
			global.points += global.current_points
			room_restart()
		}
		else {
			room_goto(Rm_End)	
		}
	}
}

if global.current_points > 60 {
	score_color = c_green	
}
else if global.current_points > 0 {
	score_color = c_red
}
```

#### Draw GUI

```gml
draw_set_color(c_black)


draw_set_font(fnt_h2)

draw_text(100,100,score_title)

draw_set_color(score_color)
var stx = 105+string_width(score_title)

draw_set_font(fnt_h1)
draw_text(stx,86,score_info)

draw_set_font(fnt_h2)
draw_set_color(c_black)

draw_set_font(fnt_h2)
draw_text(440,100,round_info)


draw_set_font(fnt_p)
draw_text_ext(50,450,rules_info,28,700)
```

### obj_bar_green
**Sprite:** spr_green

#### Create

```gml
depth = 7
```

### obj_bar_cursor
**Sprite:** spr_cursor

#### Create

```gml
depth = 3
```

### obj_bar

#### Create

```gml
image_speed = 0

depth = 8

gx = floor(random_range(200,650))

green = instance_create_layer(gx,y,"Instances",obj_bar_green)

cursor = instance_create_layer(x,y-5,"Instances",obj_bar_cursor)

cursor_start = x+10
cursor_end = x+sprite_width-10

cursor_going_right = true

cursor_speed = floor(random_range(7,10))
losing_score = floor(random_range(30,60))

already_lost = false
```

#### Step

```gml
var winning_touch = collision_rectangle(cursor.bbox_left-10,cursor.bbox_top,cursor.bbox_right+10,cursor.bbox_bottom,obj_bar_green,0,0)
var losing_touch = !collision_rectangle(cursor.bbox_left-20,cursor.bbox_top,cursor.bbox_right+20,cursor.bbox_bottom,obj_bar_green,0,0)

if global.bar_active {
	if cursor_going_right {
		if cursor.x < cursor_end {
			cursor.x += cursor_speed
		}
		else {
			cursor_going_right = false	
		}
	}
	else {
		if cursor.x > cursor_start {
			cursor.x -= cursor_speed	
		}
		else {
			cursor_going_right = true	
		}
	}
}

if global.released and !already_lost{
	if global.can_win {
		global.current_points = 100-abs(abs(cursor.x)-green.x)
		global.bar_active = false	
	}
	else {
		already_lost = true
	}
}

if already_lost {
	if losing_touch {
		global.bar_active = false	
		var temp_score =  100-abs(abs(cursor.x)-green.x)
		if temp_score > 50 {
			global.current_points = losing_score	
		}
		else if temp_score > 0 {
			global.current_points = temp_score	
		}
		else {
			global.current_points = 0
		}
		
	}
}
```

### obj_btn
**Sprite:** spr_btn

#### Create

```gml
image_speed = 0
```

#### Step

```gml
if point_in_rectangle(mouse_x,mouse_y,bbox_left,bbox_top,bbox_right,bbox_bottom) and !global.grace_timer {
	if mouse_check_button_released(mb_left) {
		if !global.released {
			global.grace_timer = 10
			global.released = true
		}
	}
	if mouse_check_button(mb_left) {
		image_index = 1	
	}
	else {
		image_index = 0	
	}
}
else {
	image_index = 0	
}
```

### obj_controller_end (again)

#### Create

```gml
draw_set_color(c_black)

var _x = 40
var _y = 75

draw_set_font(fnt_h2)
var sw = string_width(title)
var swx = (room_width-sw)/2

draw_text(swx,_y,title)

_y += 150

draw_text(_x,_y,score_title)

var stx = string_width(score_title)

draw_set_font(fnt_h1)
draw_set_color(score_color)
draw_text(_x+stx,_y-14,string(final_score))

draw_set_color(c_black)
draw_text(_x + 450,_y-14,restart)

_y += 45

draw_set_color(score_color)
draw_text(_x,_y,eval_text)

_y += 150

draw_set_color(c_black)
draw_set_font(fnt_p)
draw_text_ext(_x,_y,eval_info,36,700)
```

#### Draw GUI

```gml
draw_set_color(c_black)

var _x = 150
var _y = 100

draw_set_font(fnt_h2)

draw_text(_x+20,_y,title)

_y += 200


draw_text(_x,_y,score_title)

var stx = string_width(score_title)

draw_set_font(fnt_h1)
draw_set_color(score_color)
draw_text(_x+stx+15,_y-8,string(final_score))

draw_set_color(c_black)
draw_text(_x + 570,_y-8,try_again)

_y += 60

draw_set_color(score_color)
draw_text(_x,_y,eval_text)

_y += 200

draw_set_color(c_black)
draw_set_font(fnt_p)
draw_text_ext(_x,_y,eval_info,36,900)
```

### obj_border_restart
**Sprite:** spr_border

#### Create

```gml
depth = 10
```

#### Left Released

```gml
room_goto(Rm_Start)
```






