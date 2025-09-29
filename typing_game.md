# Typing Game Notes

## **ROOMS**

**Size:** 1200x800

- **Rm_Main**
- **Rm_Gameover**

## **SPRITES**

**spr_input** 700x150

## **FONTS**

- **fnt_title:** Roboto Mono, 60
- **fnt_header:** Roboto Mono, 30

## **OBJECTS**

### **obj_input**

#### **Create**

```gml
randomize()

// DATA

data_arr = ["eagle","intrinsic","quirk"]

array_push(data_arr,"bird")
array_push(data_arr,"plane")
array_push(data_arr,"fish")
array_push(data_arr,"tiger")
array_push(data_arr,"insect")
array_push(data_arr,"repunzel")
array_push(data_arr,"edict")
array_push(data_arr,"cabinet")
array_push(data_arr,"easter")
array_push(data_arr,"inflection")
array_push(data_arr,"tired")
array_push(data_arr,"sandwich")
array_push(data_arr,"perspirate")

// GAME VARS

global.frame_counter = 0

words_completed = 0

input_string = ""
word_string = ""

// FUNCTION

function array_remove(arr,index) {
	var temp_arr = []
	for (var i = 0;i < array_length(arr);i++) {
		if index != i {
			array_push(temp_arr,arr[i])	
		}
	}
	return temp_arr
}

// TEXT VARS

finished_typing = true
word_color = c_white

add_letter = false

goal_word_x = 0
input_word_x = 0

draw_set_font(fnt_title)
char_width = string_width("i")
```

#### **Key Press - Any**

```gml

add_letter = true
```

#### **Step**

```gml

// ROLLING GOAL WORD 

if finished_typing {
	
	input_string = ""
	word_color = c_white
	
	var word_index = irandom(array_length(data_arr)-1)
	word_string = data_arr[word_index]
	data_arr = array_remove(data_arr,word_index)
	
	var letter_count = string_length(word_string)
	goal_word_x = (room_width - (letter_count * char_width)) / 2
	
	finished_typing = false
	
}

if add_letter {
	if string_length(input_string) < 14 {
		var new_key = string_lower(keyboard_lastchar)
		if new_key >= "a" and new_key <= "z" {
			input_string += new_key	
			add_letter = false
		}
		else {
			add_letter = false	
		}
		
		var word_snippet = string_copy(word_string,1,string_length(input_string))
		if input_string != word_snippet {
			word_color = c_red	
		}
		
	}
	else {
		add_letter = false	
	}
}

if keyboard_check_pressed(vk_backspace) {
	word_color = c_white
	input_string = ""
	add_letter = false
}

var input_letter_count = string_length(input_string)
input_word_x = (room_width - (input_letter_count * char_width)) / 2

// GAME LOGIC

if string_length(input_string) >= string_length(word_string) {
	if input_string == word_string {
		words_completed++
		finished_typing = true
		input_string = ""
	}
}

if words_completed >= 10 {
	room_goto(Rm_GameOver)	
}

global.frame_counter++
```



#### **Draw**

```gml

draw_self()
draw_set_color(c_white)

draw_set_font(fnt_title)
draw_text(100,80,"Typing Game")

draw_set_font(fnt_header)
draw_text(110,200,"Type the word you see!")

draw_text(900,80,"Score: ")
draw_text(900,140,string(words_completed) + " / 10")

draw_set_font(fnt_title)
draw_text(goal_word_x,300,word_string)

draw_set_color(word_color)
draw_text(input_word_x,y+15,input_string)
```
### **obj_gameover**

#### **Draw**

```gml

draw_set_color(c_white)

draw_set_font(fnt_title)
draw_text(100,80,"Game Over!")

draw_set_font(fnt_header)
var total_score = 10000 - global.frame_counter
draw_text(110,250,"Final Score: " + string(total_score))
```

#### **Key Press - Any**

```gml

room_goto(Rm_Main)
```


