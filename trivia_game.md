# Trivia Game Notes

**Room Size:** 1200x800

## **FONTS**

- **fnt_title:** Montserrat, 60
- **fnt_header:** Montserrat, 48
- **fnt_text:** Roboto

## **OBJ_CONTROLLER**

### **CREATE**

```gml
// ARRAY

// 2D Array: [
// [Question Text, Choice 1, Choice 2, Choice 3, Choice 4, Correct Answer]
// [Question Text, Choice 1, Choice 2, Choice 3, Choice 4, Correct Answer]
// [Question Text, Choice 1, Choice 2, Choice 3, Choice 4, Correct Answer]
// ]

text_arr = []
array_push(text_arr,["How many licks does it take to get to the center of a tootsie pop?","One","Two","Three","Unknowable",4])
array_push(text_arr,["What is the meaning of life, the universe, and everything?","Religion","Philosophy","42","No one knows",3])
array_push(text_arr,["What number am I thinking of?","Seven","Eleven","Four","Pizza",2])


// ENUM

enum GAME_STATE {
	
	PREGAME,
	QUESTION_ASKED,
	QUESTION_ANSWERED,
	END_SCREEN
		
}

game_state = GAME_STATE.PREGAME

// VARS

points = 0
level = 0

title_text = ""
question_text = ""

choice1_text = ""
choice2_text = ""
choice3_text = ""
choice4_text = ""

chosen_answer = -1
correct_answer = -1
```

### **DRAW**

```gml
draw_set_color(c_white)

// TITLE

draw_set_font(fnt_title)
draw_text(100,100,title_text)

// QUESTION TEXT
draw_set_font(fnt_text)
draw_text_ext(100,250,question_text,50,1000)

// QUESTION 1
draw_set_font(fnt_header)
draw_text(100,400,choice1_text)

// QUESTION 2
draw_set_font(fnt_header)
draw_text(600,400,choice2_text)

// QUESTION 3
draw_set_font(fnt_header)
draw_text(100,550,choice3_text)

// QUESTION 4
draw_set_font(fnt_header)
draw_text(600,550,choice4_text)
```


### **STEP**

```gml
switch game_state {
	
	case GAME_STATE.PREGAME:

		title_text = "Trivia Game!"
		question_text = "Click anywhere to start!"

		
		if mouse_check_button_released(mb_left) {
			
			game_state = GAME_STATE.QUESTION_ASKED
			
		}
		
		break;
		
	case GAME_STATE.QUESTION_ASKED:
	
		title_text = "Question " + string(level + 1)
		question_text = text_arr[level][0]
		choice1_text = text_arr[level][1]
		choice2_text = text_arr[level][2]
		choice3_text = text_arr[level][3]
		choice4_text = text_arr[level][4]
		correct_answer = text_arr[level][5]
		
		if mouse_check_button_released(mb_left){
			if point_in_rectangle(mouse_x,mouse_y,100,400,500,550) {
				chosen_answer = 1	
			}
			else if point_in_rectangle(mouse_x,mouse_y,600,400,1000,550) {
				chosen_answer = 2
			}
			else if point_in_rectangle(mouse_x,mouse_y,100,550,500,700) {
				chosen_answer = 3
			}
			else if point_in_rectangle(mouse_x,mouse_y,600,550,1000,700) {
				chosen_answer = 4
			}
		}
		
		if chosen_answer > 0 {
			game_state = GAME_STATE.QUESTION_ANSWERED	
		}
	
		break;
		
	case GAME_STATE.QUESTION_ANSWERED:
		
		if chosen_answer == correct_answer {
			title_text = "Correct!"	
		}
		else {
			title_text = "Nope!"	
		}
		
		var temp_text = "Correct Answer: " + text_arr[level][correct_answer]
		question_text = temp_text
		
		choice1_text = ""
		choice2_text = ""
		choice3_text = ""
		choice4_text = ""
		
		if mouse_check_button_released(mb_left){
			
			chosen_answer = -1
			correct_answer = -1
			level += 1
			points += 1
			
			if level >= array_length(text_arr) {
				game_state = GAME_STATE.END_SCREEN	
			}
			else {
				game_state = GAME_STATE.QUESTION_ASKED
			}
			
				
		}
		
		break;
		
	case GAME_STATE.END_SCREEN:
		
		choice1_text = ""
		choice2_text = ""
		choice3_text = ""
		choice4_text = ""
		
		title_text = "Game Over!"
		question_text =  "Your Score: " + string(points) + ". Click Anywhere to Restart."
		
		if mouse_check_button_released(mb_left) {
			room_restart()	
		}
		
}
```
