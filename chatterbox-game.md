
# **Navigation**
- [Intro and Explanation](#intro-and-explanation)
- [Building a Game in ChatterScript](#building-a-game-in-chatterscript)
- [Loading your Chatterbox in GameMaker](#loading-your-chatterbox-in-gamemaker)
- [Building the GameMaker Game Wrapper](#building-the-gamemaker-game-wrapper)
  - [Rooms](#rooms)
  - [Fonts](#fonts)
  - [Sprites](#sprites)
  - [Objects](#objects)
 
# **Intro and Explanation**

These notes need a lot of cleaning up, I'll get to posting a better explanation and fixing the formatting of the GML later in these notes. Thank you for your patience.

# **Building a Game in Chatterscript**

### **Start**

```text

<<set $PressurePoints to 10>>

<<set $Fidgeting to false>>

// FaceMoods 0=Regular,1=Surprised,2=Angry

<<set $FaceMood to 0>>

<<set $Day to 1>>

<<set $ResetRoom to false>>

<<set $GameOver to false>>

<<set $GameWon to false>>
```

### **Clean**

```text
<<set $ResetRoom to false>>

<<set $FaceMood to 0>>

<<set $Fidgeting to false>>
```

### **Day1 Begin**

```text
Narrator: You walk into the interrogation room and sit down. Across from you is a middle-aged woman, distraught, still wearing her pajamas.

Narrator: She patiently waits for you to ask your first question.

  -> Let's start with some formalities. What's your name?

  <<jump Gretchen Name>>
```

### **Day2 Begin**

```text
<<set $Fidgeting to true>>

Narrator: You walk back into the interrogation room for the second day. Gretchen sits before you again, looking more upset than usual.

  -> Hi Gretchen, let's begin

  <<jump Accusation>>
```

### **Gretchen Name**

```text
Gretchen: My name is Gretchen Balmer

   -> Hi Gretchen, nice to meet you!

       << set $PressurePoints -= 1>>

       Gretchen: Nice to meet you, too.

       -> Continue

       <<jump Pressure Checkpoint 1>>

   -> Balmer? I feel like I've heard that before

       Gretchen: Yes, my family owns Balmer Industries, we make those little caps when you, when you go to put air in your tire and you take off that little cap.

       -> Wow, you must be worth a fortune!

           <<set $FaceMood to 2>>

           Gretchen: Yes, somewhat.

               -> Wow...

               << set $PressurePoints -= 1>>

               <<jump Pressure Checkpoint 1>>

       -> In my experience, more money tends to create more problems.

           <<set $Fidgeting to true>>

           << set $PressurePoints += 2>>

           <<jump Pressure Checkpoint 1>>
```

### **Pressure Checkpoint 1**

```text
<<if $PressurePoints < 10 >>

   Narrator: That seems to have made her more comfortable. If you want to get a confession, you'll need to apply more pressure.

   -> Tell me what happened earlier this evening.

   <<jump Testimony>>

<<else>>

   Narrator: That put her on the defensive.

   -> Tell me what happened earlier this evening.

   <<jump Testimony>>
```

### **Testimony**

```text
<<set $Fidgeting to false>>

<<set $FaceMood to 0>>

<<if once("Testimony")>>

   <<set $FaceMood to 1>>

   Gretchen: Well, I was getting ready to get into bed when I heard a loud bang downstairs. When I went down there, a man in a ski mask put a bag over my head...

   Gretchen: They tied me up and made me tell them where the safe was. When they were finished, they took the bag off and told me to count down from 100...

   Gretchen: When I finished counting, I called the cops.

   <<set $FaceMood to 0>>

<<endif>>

Gretchen: What else do you want to know?

   -> That must have been very hard for you. <<if !optionChosen() >>

       <<set $PressurePoints += 2>>

       <<set localCounter("questions")>>

       <<jump Small Talk>>

   

   -> Who were "they"? Where was the other man? <<if !optionChosen() >>

       <<set $Fidgeting to true>>

       <<set $FaceMood to 2>>

       Gretchen: I don't know, but I heard two voices.

       -> What were there voices like? Loud? Soft?

       Gretchen: They were two younger men, that's all I know.

       -> I'll keep it in mind.

       <<set $PressurePoints -= 1>>

       <<set localCounter("questions")>>

       <<jump Jumpback Testimony>>



   -> Why did you wait to call the cops? <<if !optionChosen() >>

       <<set $Fidgeting to true>>

       <<set $PressurePoints -= 1 >>

       <<set localCounter("questions")>>

       Gretchen: I... I don't know, I was scared.

           -> I understand.

           <<jump Jumpback Testimony>>



   -> What was stolen out of the safe? <<if !optionChosen() >>

       <<set localCounter("questions")>>

       Gretchen: Jewelry, mostly

           -> Can you be more specific?

           <<jump Stolen Goods>>

<<if localCounter("questions") > 3>>

   -> Do you think you know the men who broke into your house?

   <<jump The Men>>
```

### **Jumbpback Testimony**

```text
<<set $Fidgeting to false>>

<<jump Testimony>>
```

### **Small Talk**

```text
Gretchen: Yeah, it was...

   -> I can only imagine. One time a bee stung me in both of my eyes

       Gretchen:...

       -> I couldn't see for a week!

       Gretchen: Oh wow...

       -> Yeah and I had this ointment

       Gretchen: uh huh...

       -> but I couldn't put it directly in my eyes

       Gretchen:...

       -> because they were so swollen.

       Gretchen:...

       -> Maybe we should get back to the case

       <<set $PressurePoints -= 3>>

       <<jump Testimony>>



   -> Maybe we should get back to the case

       <<jump Testimony>>
```
### **Stolen Goods**

```text
Gretchen: Let's see, there was a silver locket, some pearl earrings...

Gretchen: ...and about fifty thousand dollars in cash.

   -> What was in the locket?

       Gretchen: Oh, a photo of my late husband. I miss him dearly, I try to keep his memory close.

           -> So why was his locket in the safe?

               <<set $Fidgeting to true>>

               <<set $PressurePoints += 2>>

               Gretchen: Oh why... I'm not sure. Now that I remember, I had it on my nightstand.

               Gretchen: Yes, they must have stolen it from there.

                   -> Tell me more about what was in the safe

                   <<jump Testimony>>

   -> Pearl earrings? Sounds kind of cliche

       Gretchen: Oh, I don't know. I like to think of them as classic.

           -> Fair enough

           <<jump Testimony>>

   -> Why do you need to keep so much cash?

       <<set $FaceMood to 1>>

       <<set $Fidgeting to true>>

       <<set $PressurePoints += 2>>

       Gretchen: Oh, well, I have a lot to pay for, and I pay people in cash. I pay people for all kinds of things.

           -> Are you bribing me? Because I'm not opposed

               <<set $FaceMood to 2>>

               <<set $PressurePoints -= 1>>

               Gretchen: Not in the least.

                   -> Let's get back to the questions

                   <<jump Testimony>>

           -> Interesting.

                   <<jump Testimony>>
```

### **The Men**

```text
<<set $Fidgeting to true>>

Gretchen: No, I mean, of course not. I couldn't have. Not in the least.

   -> You sound unsure of yourself.

       <<set $FaceMood to 1>>

       Gretchen: No, they didn't look familiar at all. I've never seen them before.

           -> I thought you said they were wearing masks

           <<set $PressurePoints += 2>>

           <<jump Day1 End>>

   -> OK, let's talk about something else.

       <<jump Day1 End>>
```

### **Day1 End**

```text
<<if $PressurePoints < 12>>

   <<set $FaceMood to 0>>

   <<jump GameOver>>

<<else>>

   <<set $FaceMood to 2>>

   Gretchen: I'm tired, I've had a long night. Can we continue this tomorrow?

   -> Yes, of course

   <<set $Day += 1>>

   <<set $ResetRoom to true>>

   <<stop>>

<<endif>>
```

### **GameOver**

```text
Narrator: You have failed to gather pertinent information. The case has been lost.

<<set $GameOver to true>>
```

### **Accusation**

```text
Gretchen: H-hello, it's good to see you, too.

   

   -> I think you did it.

       << set $PressurePoints += 5>>

       <<set $FaceMood to 2>>

       Gretchen: I... what? What do you mean?

           -> I think you staged the robbery

           Gretchen:...

           -> Or know who broke in.

           Gretchen: Where's your proof?

               -> Right here (bluff)

                   <<jump Pressure Checkpoint 2>>

               -> I don't have any

                   <<set $PressurePoints -= 3>>

                   <<jump Pressure Checkpoint 2>>

   -> I don't think you did it

       <<set $FaceMood to 0>>

       <<set $PressurePoints -= 3>>

       <<jump Pressure Checkpoint 2>>
```

### **Pressure Checkpoint 2**

```text
<<if $PressurePoints < 20>>

   <<jump GameOver>>

<<else>>

   <<jump Confession>>
```

### **Confession**

```text
<<set $FaceMood to 2>>

Gretchen: OK, you're right! I did it! I'm sorry! I staged the whole thing. I just want this to be over already.

   -> Me too

       <<jump GameWon>>
```

### **GameWon**

```text
Narrator: With that confession, Gretchen will spend some time behind bars. Congratulations, Detective.

   <<set $GameWon to true>>

   <<stop>>
```

# **Loading Your Chatterbox in GameMaker**

- Download .yymps from Chatterbox
- Save .yarn file in editor
- Open up GameMaker
- Go to Tools > Import Local Package, Import Chatterbox .yymps
- Navigate to Help > Open Project in Explorer, drop .yarn file in datafiles


# **Building the GameMaker Game Wrapper**

## **ROOMS**
Size: 1200x800
- Rm_Start
- Rm_Main
- Rm_Night
- Rm_GameWon
## **FONTS**
- **fnt_title** Roboto Mono, size 60
- **fnt_text** Roboto Mono, size 18
## **SPRITES**
### **From Ko-Fi:**
- **spr_bg**
- **spr_body** Origin: 197x6
- **spr_brows** Origin: Middle centre
- **spr_eyes** Origin: Middle centre
- **spr_head** Origin: 141x209
- **spr_mouth** Origin: 26x9
### **Make in GameMaker**
- **spr_overlay_black** Size: 1200x800
- **spr_text_black** Size: 100x250

## **OBJECTS**
Make obj_initializer, obj_controller_main, obj_controller_night, drop in rooms.<br />
Make obj_text, leave all for now.<br />

### **obj_bg**

#### **Create**

```gml
depth = 10

image_speed = 0

image_index = 0
```

### **obj_overlay**

#### **Create**

```gml
depth = 2

image_speed = 0

image_index = 1
```

### **obj_overlay_black**

#### **Create**

```gml
depth = 0



if room == Rm_Start {

image_alpha = 0.9

}

else {

image_alpha = 0

}
```

### **obj_brows**

#### **Create**

```gml
image_speed = 0

image_index = 0

depth = 4
```

### **obj_eyes**

#### **Create**

```gml
image_speed = 0

depth = 5

enum BLINK_STATE {

OPEN,
BLINKING,
CLOSED

}

blink_state = BLINK_STATE.OPEN

blink_timer = random_range(100,300)

blink_frame = 0

has_blinked = false
```

#### **Step**

```gml
switch(blink_state) {

case BLINK_STATE.OPEN:

image_index = 0

has_blinked = false

if blink_frame > 0 {

blink_frame -= 0.2

}

else {

blink_frame = 0

}

if blink_timer >= 0 {

blink_timer--

}

else {

blink_state = BLINK_STATE.BLINKING

}

break;

case BLINK_STATE.BLINKING:

image_index = ceil(blink_frame)

if !has_blinked and blink_frame < 2{

blink_frame += 0.2

}

else if !has_blinked and blink_frame >= 2 {

has_blinked = true

}

else if has_blinked and blink_frame > 0 {

blink_frame -= 0.2

}

else if has_blinked and blink_frame <= 0 {

blink_timer = random_range(100,300)

blink_state = BLINK_STATE.OPEN

}

break;

case BLINK_STATE.CLOSED:

image_index = ceil(blink_frame)

if blink_frame < 2 {

blink_frame += 0.2

}

break;
}

if global.pressure_points >= 20 {

blink_state = BLINK_STATE.CLOSED

}

```

### **obj_mouth**

#### **Create**

```gml
depth = 2

image_speed = 0

enum MOUTH_STATE {

CLOSED,
FLAPPING

}

mouth_state = MOUTH_STATE.CLOSED

mouth_frame = 0

mouth_speed = 0.5

is_closing = false

```

#### **Step**

```gml
switch(mouth_state) {

case MOUTH_STATE.CLOSED:

image_index = 0

is_closing = false

if mouth_frame > 0 {

mouth_frame -= mouth_speed

}

break;


case MOUTH_STATE.FLAPPING:

image_index = floor(mouth_frame)

if !is_closing and mouth_frame < 3 {

mouth_frame += mouth_speed

}

else if !is_closing and mouth_frame >= 3 {

is_closing = true

}

else if is_closing and mouth_frame > 0 {

mouth_frame -= mouth_speed

}

else if is_closing and mouth_frame <= 0 {

mouth_frame = 0

is_closing = false

}

break;

}
mouth_state = global.is_talking
```

### **obj_head**

#### **Create**

```gml
Create

depth = 7



brow_x = 36

brow_y = 87

bia = 0



if room == Rm_Start {

image_blend = c_black

}

else {

left_brow = instance_create_layer(x-brow_x,y-brow_y,"Instances",obj_brows)

right_brow = instance_create_layer(x+brow_x,y-brow_y,"Instances",obj_brows)

right_brow.image_index = 1

eyes = instance_create_layer(x,y-68,"Instances",obj_eyes)

mouth = instance_create_layer(x,y-1,"Instances",obj_mouth)

}



enum FACE_STATE {



REGULAR,

SURPRISED,

ANGRY



}



face_state = FACE_STATE.REGULAR
```

#### **Step**

```gml
if room != Rm_Start {



left_brow.image_angle = bia

right_brow.image_angle = -(bia)



left_brow.x = x-36

left_brow.y = y-87



right_brow.x = x+36

right_brow.y = y-87



eyes.x = x

eyes.y = y-68



mouth.x = x

mouth.y = y-1





switch(face_state) {



case FACE_STATE.REGULAR:

if bia < 0 {

bia++

}

else if bia > 0 {

bia--

}

break;



case FACE_STATE.SURPRISED:



if bia < 10 {

bia++

}
break;

case FACE_STATE.ANGRY:

if bia > -10 {

bia--

}

break;


}

}
face_state = global.face_mood
```

### **obj_body**

#### **Create**

```gml
depth = 8



if room == Rm_Start {

image_blend = c_black

}



enum BODY_STATE {



REGULAR,

TILTED_LEFT,

TILTED_RIGHT,

TILTED_DOWN



}



body_state = BODY_STATE.REGULAR



enum FIDGET_STATE {

REGULAR,

FIDGETING

}



fidget_state = FIDGET_STATE.REGULAR



fidget_timer = 5



head = instance_create_layer(x,y+2,"Instances",obj_head)



og_x = x

og_y = y

og_ia = image_angle



move_speed = 2



tilted_ia = 4

tilted_x = 577

tilted_y = 420

```

#### **Step**

```gml
if instance_exists(obj_head) {

if body_state == BODY_STATE.TILTED_DOWN {

if obj_head.y < y + 6 {

obj_head.y += move_speed

}

}

else {

obj_head.x = x

obj_head.y = y + 2

}

}





switch(body_state) {



case BODY_STATE.REGULAR:



if image_angle < og_ia {

image_angle += move_speed / 2

}

else if image_angle > og_ia {

image_angle -= move_speed / 2

}



if x < og_x {

x += move_speed

}

else if x > og_x {

x -= move_speed

}



if y < og_y {

y += move_speed

}

else if y > og_y {

y -= move_speed

}



break;



case BODY_STATE.TILTED_LEFT:



if x > tilted_x {

x -= move_speed

}

if y < tilted_y {

y += move_speed

}

if image_angle < tilted_ia {

image_angle += move_speed / 2

}



break;



case BODY_STATE.TILTED_RIGHT:



if x < tilted_x+43 {

x += move_speed

}

if y < tilted_y {

y += move_speed

}

if image_angle > -(tilted_ia) {

image_angle -= move_speed / 2

}



break;



case BODY_STATE.TILTED_DOWN:



if y < tilted_y {

y += move_speed

}



break;



}



switch(fidget_state) {



case FIDGET_STATE.REGULAR:



body_state = BODY_STATE.REGULAR





break;







case FIDGET_STATE.FIDGETING:



if fidget_timer > 0 {

fidget_timer--

}

else {

if body_state == BODY_STATE.REGULAR {

body_state = choose(1,2,3)

}

else {

body_state = BODY_STATE.REGULAR

}

fidget_timer = random_range(20,100)

}



break;

}



fidget_state = global.fidgeting
```

### **obj_initializer**

#### **Create**

```gml
global.pressure_points = 10



global.file = "InterrogationData1a.yarn"



ChatterboxLoadFromFile(global.file)

chatterbox = ChatterboxCreate();

ChatterboxJump(chatterbox, "Start");

ChatterboxStop(chatterbox)



global.fidgeting = false

global.reset_room = false

global.face_mood = 0



instance_create_layer(0,0,"Instances",obj_bg)

instance_create_layer(0,0,"Instances",obj_overlay)

instance_create_layer(0,0,"Instances",obj_overlay_black)

instance_create_layer(600,400,"Instances",obj_body)

```

#### **Step**

```gml
if keyboard_check_released(vk_anykey) {

room_goto(Rm_Main)

}
```

#### **Draw**

```gml
draw_set_font(fnt_title)

draw_set_color(c_white)



var sw = string_width("Interrogation")

draw_text((room_width-sw)/2,200,"Interrogation")



var sw = string_width("Start Game")

draw_text((room_width-sw)/2,400,"Start Game")
```

### **obj_controller_main**

#### **Create**

```gml
enum GAME_STATE {



NULL,

FADE_IN,

FADE_OUT,

PLAY





}



game_state = GAME_STATE.FADE_IN



reset_room = false

game_over = false



game_won = false



global.fidgeting = false

global.reset_room = false



global.is_typing = false

global.is_talking = false

global.face_mood = 0



instance_create_layer(0,0,"Instances",obj_bg)

instance_create_layer(0,0,"Instances",obj_overlay)

instance_create_layer(600,400,"Instances",obj_body)

```

#### **Step**

```gml
if instance_exists(obj_text) {

if obj_text.chatterbox {

global.pressure_points = ChatterboxVariableGet("PressurePoints")

global.fidgeting = ChatterboxVariableGet("Fidgeting")

reset_room = ChatterboxVariableGet("ResetRoom")

game_over = ChatterboxVariableGet("GameOver")

game_won = ChatterboxVariableGet("GameWon")

global.face_mood = ChatterboxVariableGet("FaceMood")

}

}



if game_over {

if instance_exists(obj_text) {

ChatterboxStop(obj_text.chatterbox)

instance_destroy(obj_text)

}

room_goto(Rm_Start)

}



if global.reset_room and !instance_exists(obj_text) {

room_goto(Rm_Night)

}



switch(game_state) {





case GAME_STATE.NULL:



break;





case GAME_STATE.FADE_IN:



if !instance_exists(obj_overlay_black) {

instance_create_layer(0,0,"Instances",obj_overlay_black)

obj_overlay_black.image_alpha = 1

}

else {

if obj_overlay_black.image_alpha > 0 {

obj_overlay_black.image_alpha -= 0.05

}

else {

game_state = GAME_STATE.PLAY

}

}



break;



case GAME_STATE.FADE_OUT:

if !instance_exists(obj_overlay_black) {

instance_create_layer(0,0,"Instances",obj_overlay_black)

obj_overlay_black.image_alpha = 1

}

else {

if obj_overlay_black.image_alpha > 0 {

obj_overlay_black.image_alpha -= 0.05

}

else {

if game_won {

room_goto(Rm_GameWon)

}

else if game_over {

room_goto(Rm_Start)

}

else {

room_goto(Rm_Night)

}

}

}

break;



case GAME_STATE.PLAY:

if !instance_exists(obj_text) {

instance_create_layer(100,530,"Instances",obj_text)

}

break;



}
```

### **obj_controller_night**

#### **Create**

```gml
ChatterboxLoadFromFile(global.file)

chatterbox = ChatterboxCreate();

ChatterboxJump(chatterbox, "Clean");

ChatterboxStop(chatterbox)

room_goto(Rm_Main)
```

### **obj_text**

#### **Create**

```gml
depth = 1

image_alpha = 0.8



ChatterboxLoadFromFile(global.file)

chatterbox = ChatterboxCreate();



day_nodes = ["Day1 Begin","Day2 Begin"]

day = ChatterboxVariableGet("Day") - 1

ChatterboxJump(chatterbox, day_nodes[day]);





dialogue_string = ""

current_string = ""

label_string = ""



string_arr = []



options_alpha = 0

options_delay = 10



dsl = 0

csl = 0

lsl = 0
```

#### **Step**

```gml
if (ChatterboxIsStopped(chatterbox)) {

if instance_exists(obj_controller_main) {

obj_controller_main.game_state = GAME_STATE.FADE_OUT

}

}

else if (ChatterboxIsWaiting(chatterbox)) and !global.is_typing {

   if options_alpha < 1 and options_delay <= 0 {

options_alpha += 0.05

}

   if (keyboard_check_released(vk_space))

   {

current_string = ""

csl = 0

options_alpha = 0

       ChatterboxContinue(chatterbox);



   }

}

else if !global.is_typing {

if options_alpha < 1 {

options_alpha += 0.05

}



   var _index = undefined;

   if (keyboard_check_released(ord("1"))) _index = 0;

   if (keyboard_check_released(ord("2"))) _index = 1;

   if (keyboard_check_released(ord("3"))) _index = 2;

   if (keyboard_check_released(ord("4"))) _index = 3;

   

   if _index != undefined {

options_alpha = 0

current_string = ""

csl = 0

ChatterboxSelect(chatterbox, _index);

}

}



if !(ChatterboxIsStopped(chatterbox)) {



dialogue_string = ChatterboxGetContent(chatterbox, 0);

dsl = string_length(dialogue_string)

string_arr = string_split(ChatterboxGetContent(chatterbox, 0),":")

label_string = string(string_arr[0])

if label_string == "Gretchen" {

global.is_talking = true

}

lsl = string_length(label_string)



}





if csl < lsl {

global.is_typing = true

current_string += label_string

csl = lsl+1

}



if csl < dsl+1 {



if keyboard_check_released(vk_space) {

current_string = dialogue_string

csl = dsl+1

}

else {

global.is_typing = true

current_string += string_char_at(dialogue_string,csl)

csl++

}

}

else {

global.is_typing = false

global.is_talking = false

}



if options_delay > 0 {

options_delay--

}

```

#### **Draw**

```gml
draw_self()



draw_set_alpha(1)

draw_set_font(fnt_text)

draw_set_color(c_white)



var _x = x+20;

var _y = y+10;





if !(ChatterboxIsStopped(chatterbox)) {



draw_text_ext(_x, _y,current_string,30,980);

   

   _y += 110



draw_set_alpha(options_alpha)

   if (ChatterboxIsWaiting(chatterbox))

   {

       draw_text(_x, _y, "(Continue)");

   }

   else

   {

       var _i = 0;

       repeat(ChatterboxGetOptionCount(chatterbox))

       {

           var _string = ChatterboxGetOption(chatterbox, _i);

           draw_text(_x, _y, string(_i+1) + ") " + _string);

           _y += 30;

           ++_i;

       }

   }

}

draw_set_alpha(1)
```
