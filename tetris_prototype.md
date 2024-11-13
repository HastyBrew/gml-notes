## TETRIS NOTES

### BASIC INFO

- Room: 400x800
- Font: Handjet (Google Fonts) Size 30

### SPRITES

- spr_animation (250x25)
- spr_block (25x25)
- spr_border (50x50)
- spr_scorebar (300x50)

### OBJECTS

- obj_animation
- obj_bar
- obj_block_moving
- obj_block_still
- obj_border_bottom
- obj_border_sides
- obj_controller
- obj_score
- obj_shape


#### OBJ_ANIMATION

##### Create

```gml
image_speed = 0.4;
```

##### Step

```gml
if image_index >= image_number - 1 {
    instance_destroy();
}
```

#### OBJ_BLOCK_STILL

##### Step

```gml
if y <= 150 {
	room_restart()	
}
```

#### OBJ_CONTROLLER

##### Create

```gml
randomize()

// globals

// drop
global.drop_timer = 20;
global.drop_delay = 20;
global.drop_distance = 25;

global.points = 0;


// switch blocks
global.blocks_spawned = false;
global.switch_blocks = false;

// find grid
global.grid_width = 10;
global.block_size = 25; 
global.grid_start_x = 75; 
global.grid_start_y = 150;

// animate change
global.animation_timer = 0;
global.cleared_row_y = -1; 
global.rows_cleared = [];
global.shift_rows_pending = false; 
global.rows_to_shift = []; 
global.grid_height = 20 
global.total_rows_cleared = 0
```


##### Step

```gml
if global.animation_timer > 0 {
    global.animation_timer--; 
    return; 
}

if global.shift_rows_pending {
	
    for (var i = 0; i < array_length(global.rows_to_shift); i++) {
        global.row_y = global.rows_to_shift[i]; 

        with (obj_block_still) {
            if (y < global.row_y) {
                y += global.total_rows_cleared * global.block_size; 
            }
        }
    }

    global.shift_rows_pending = false;
    global.rows_to_shift = [];
    global.total_rows_cleared = 0; 
}


if global.drop_timer > 0 {
    global.drop_timer--;
} else {
    if global.blocks_spawned == true {
        global.drop_timer = global.drop_delay;
    } else {

        instance_create_layer(150, 150, "Instances", obj_shape);
        global.blocks_spawned = true;
        global.drop_timer = global.drop_delay;
    }
}
```

#### OBJ_SHAPE

##### Create

```gml
var shapes = [
    // Square shape
    [
        [[0, 0], [25, 0], [0, 25], [25, 25]],  
        [[0, 0], [25, 0], [0, 25], [25, 25]],  
        [[0, 0], [25, 0], [0, 25], [25, 25]], 
        [[0, 0], [25, 0], [0, 25], [25, 25]] 
    ],
    
    // Line shape
    [
        [[0, 0], [-25, 0], [-50, 0], [25, 0]],
        [[0, 0], [0, -25], [0, -50], [0, 25]], 
        [[0, 0], [-25, 0], [-50, 0], [25, 0]],  
        [[0, 0], [0, -25], [0, -50], [0, 25]]  
    ],
    
    // T-shape
    [
        [[0, -25], [-25, 0], [0, 0], [25, 0]],  
        [[0, 0], [0, -25], [0, 25], [25, 0]],     
        [[0, 25], [-25, 0], [0, 0], [25, 0]],     
        [[0, 0], [-25, 0], [0, -25], [0, 25]]   
    ],
    
    // L-shape
    [
        [[-25, 0], [0, 0], [25, 0], [25, -25]], 
        [[0, -25], [0, 0], [0, 25], [25, 25]],  
        [[-25, 0], [0, 0], [25, 0], [-25, 25]], 
        [[0, -25], [0, 0], [0, 25], [-25, -25]]   
    ],
    
    // Reverse L-shape
    [
        [[25, 0], [0, 0], [-25, 0], [-25, -25]], 
        [[0, -25], [0, 0], [0, 25], [-25, 25]],  
        [[25, 0], [0, 0], [-25, 0], [25, 25]],  
        [[0, -25], [0, 0], [0, 25], [25, -25]]   
    ],
    
    // Z-shape
    [
        [[-25, 0], [0, 0], [0, 25], [25, 25]],   
        [[0, 25], [0, 0], [25, 0], [25, -25]],    
        [[-25, 0], [0, 0], [0, 25], [25, 25]],   
        [[0, 25], [0, 0], [25, 0], [25, -25]]   
 
    ],
    
    // Reverse Z-shape
    [
        [[25, 0], [0, 0], [0, 25], [-25, 25]],   
        [[0, 25], [0, 0], [-25, 0], [-25, -25]],  
        [[25, 0], [0, 0], [0, 25], [-25, 25]],  
        [[0, 25], [0, 0], [-25, 0], [-25, -25]] 
    ]
];


selected_shape = irandom(array_length(shapes) - 1);
rotation_state = 0;

rotation_positions = shapes[selected_shape];

block1 = instance_create_layer(x + rotation_positions[rotation_state][0][0], y + rotation_positions[rotation_state][0][1], "Instances", obj_block_moving);
block2 = instance_create_layer(x + rotation_positions[rotation_state][1][0], y + rotation_positions[rotation_state][1][1], "Instances", obj_block_moving);
block3 = instance_create_layer(x + rotation_positions[rotation_state][2][0], y + rotation_positions[rotation_state][2][1], "Instances", obj_block_moving);
block4 = instance_create_layer(x + rotation_positions[rotation_state][3][0], y + rotation_positions[rotation_state][3][1], "Instances", obj_block_moving);

blocks_arr = [block1, block2, block3, block4];

drop_it = false;
allow_movement = true

game_over = false
collision_check = false
allow_horizontal = true;

grace_timer = 1; 
has_collided = false;

function check_and_clear_rows() {
    global.rows_cleared = [];
    global.total_rows_cleared = 0;

    for (var row = global.grid_height - 1; row >= 0; row--) {
        global.row_y = global.grid_start_y + (row * global.block_size);
        var blocks_in_row = 0;

        with (obj_block_still) {
            if (y == global.row_y) {
                blocks_in_row++;
            }
        }

        if (blocks_in_row == global.grid_width) {
            global.rows_cleared[array_length(global.rows_cleared)] = global.row_y;
            global.total_rows_cleared++;
        }
    }

    if (array_length(global.rows_cleared) > 0) {
        global.animation_timer = 30;

        for (var i = 0; i < array_length(global.rows_cleared); i++) {
            global.row_y = global.rows_cleared[i];

            instance_create_layer(global.grid_start_x, global.row_y, "Instances", obj_animation);

            with (obj_block_still) {
                if (y == global.row_y) {
					global.points += 10
                    instance_destroy();
                }
            }
        }

        global.shift_rows_pending = true;
        global.rows_to_shift = global.rows_cleared;
    }
}
```


##### Step

```gml
// control drop
allow_horizontal = true;

if global.drop_timer <= 0 || drop_it {
    allow_horizontal = false;

    var can_drop = true;
    for (var i = 0; i < array_length(blocks_arr); i++) {
        if collision_rectangle(blocks_arr[i].x, blocks_arr[i].y + global.drop_distance, 
                                blocks_arr[i].x + 25, blocks_arr[i].y + global.drop_distance + 25, 
                                obj_block_still, false, false) ||
            collision_rectangle(blocks_arr[i].x, blocks_arr[i].y + global.drop_distance, 
                                blocks_arr[i].x + 25, blocks_arr[i].y + global.drop_distance + 25, 
                                obj_border_bottom, false, false) {
            can_drop = false;
            break;
        }
    }

    if can_drop {
  
        y += global.drop_distance;
        for (var i = 0; i < array_length(blocks_arr); i++) {
            blocks_arr[i].y += global.drop_distance;
        }
    } else {
  
        if !has_collided {
            has_collided = true; 
            grace_timer = 1; 
        }
    }

    drop_it = false;
}

if has_collided {
    grace_timer--;
    if grace_timer <= 0 {
        global.switch_blocks = true;

        for (var i = 0; i < array_length(blocks_arr); i++) {
            instance_create_layer(blocks_arr[i].x, blocks_arr[i].y, "Instances", obj_block_still);
            instance_destroy(blocks_arr[i]);
        }

        global.blocks_spawned = false;
        global.switch_blocks = false;

        check_and_clear_rows();

        instance_destroy();
    }
}

if allow_horizontal && !has_collided { 
	
    // Move left
    if keyboard_check_pressed(vk_left) {
        x -= 25;
        for (var i = 0; i < array_length(blocks_arr); i++) {
            blocks_arr[i].x -= 25;
        }

        var collision_check = false;
        for (var i = 0; i < array_length(blocks_arr); i++) {
            if (blocks_arr[i].x < 75 || 
                collision_rectangle(blocks_arr[i].x, blocks_arr[i].y, blocks_arr[i].x + 25, blocks_arr[i].y + 25, obj_block_still, false, false)) {
                collision_check = true;
                break;
            }
        }

        if collision_check {
            x += 25;
            for (var i = 0; i < array_length(blocks_arr); i++) {
                blocks_arr[i].x += 25;
            }
        }
    }

    // Move right
    if keyboard_check_pressed(vk_right) {
        x += 25;
        for (var i = 0; i < array_length(blocks_arr); i++) {
            blocks_arr[i].x += 25;
        }

        var collision_check = false;
        for (var i = 0; i < array_length(blocks_arr); i++) {
            if (blocks_arr[i].x > 300 || 
                collision_rectangle(blocks_arr[i].x, blocks_arr[i].y, blocks_arr[i].x + 25, blocks_arr[i].y + 25, obj_block_still, false, false)) {
                collision_check = true;
                break;
            }
        }

        if collision_check {
            x -= 25;
            for (var i = 0; i < array_length(blocks_arr); i++) {
                blocks_arr[i].x -= 25;
            }
        }
    }
}

// Speed up on down press
if keyboard_check_pressed(vk_down) {
    global.drop_timer = 0;
    drop_it = true;
}

// Rotate on up press
if allow_horizontal && !has_collided && keyboard_check_pressed(vk_up) { 
    var next_rotation_state = (rotation_state + 1) mod 4;
    var valid_rotation = true;

    for (var i = 0; i < array_length(blocks_arr); i++) {
        var new_x = x + rotation_positions[next_rotation_state][i][0];
        var new_y = y + rotation_positions[next_rotation_state][i][1];

        if new_x < 75 || new_x > 300 || 
           collision_rectangle(new_x, new_y, new_x + 25, new_y + 25, obj_block_still, false, false) {
            valid_rotation = false;
            break;
        }
    }

    if valid_rotation {
        rotation_state = next_rotation_state;
        for (var i = 0; i < array_length(blocks_arr); i++) {
            blocks_arr[i].x = x + rotation_positions[rotation_state][i][0];
            blocks_arr[i].y = y + rotation_positions[rotation_state][i][1];
        }
    }
}
```


