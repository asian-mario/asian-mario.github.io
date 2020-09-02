---
title: "Raylib-Tetris: Spawning and Movement"
date: 2020-08-29
---
## Raylib-Tetris: Spawning and Movement (29 AUG - 30 SEPT) 
After basic hit detection, auto-moving the blocks and basic 'spawning', I got onto the basic fundamentals of tetris, spawning the blocks and moving them.
### Arraysception
To make the blocks i of-course had to make them with 2D arrays, so I got all the images of tetrominos and made them in a variable which i called `const char TEMPLATES_ARR[6][2][4]`, I would have 6 2D arrays which had 2 rows and was 4 columns wide. Next, I would focus on actually spawning these arrays inside the playGrid, so what i did was make a enum class called 'TEMPLATES'. I would then name each template the same name as their tetromino blocks, e.g `S_BLOCK_TEMPLATE`, here is the full list of template names (I had Invalid templates to essentially say it wasnt possible to spawn it, will be explained further) :

    enum class TEMPLATES
    {
		I_BLOCK_TEMPLATE,
		J_BLOCK_TEMPLATE,
		L_BLOCK_TEMPLATE,
		O_BLOCK_TEMPLATE,
		S_BLOCK_TEMPLATE,
		T_BLOCK_TEMPLATE,
		Z_BLOCK_TEMPLATE,
		INVALID
	}

After that, I made a boolean function called drawBlockTemplate, the parameters were:

`bool drawBlockTemplate(char grid[24][10], TEMPLATES t, int row, int col)`
**Note: grid[24][10] is just essentially playGrid**
In short, this function would basically make the templates, which at this point were still 'x' on arrays to actual blocks.

    bool drawBlockTemplate(char grid[24][10], TEMPLATES t, int row, int col) {
		bool overlap = false;
		for (int i = row; i < row + 2; ++i)
			for (int j = col; j < col + 4; ++j) {
				float posX = ((float)285 / 10) * j;
				float posY = ((float)684 / 24) * i;
				if (TEMPLATE_ARR[(int)t][i - row][j - col] == 'x') {
					if (grid[i][j] != ' ')
						overlap = true;
					grid[i][j] = 'x';
				}
		
			{
		return !overlap;
	{



Then, I made another function (but this time it was a void function) called `spawnNextBlock`, then map out the available space I had since I was using arrays to store my templates. I had 5 available column spaces in total, in regular tetris they would spawn randomly across the top row, I randomized the spawning by using cpp's `#include <random>` library and wrote down the following code:

    TEMPLATES spawnNextBlock(char grid[24][10], TEMPLATES t) {
	    //decide where to spawn
	    int row = 0
	    int col = rand() % 6; //generate a random between 0-5
	{	
This piece of code would determine where to spawn, these integer variables will be used later on inside the function. Next, I would work on drawing the actual object onto the playGrid itself.

    	bool ableToDraw = drawBlockTemplate(grid, t, row, col);
	if (!ableToDraw)
		return TEMPLATES::INVALID;
This piece of code inside the function would decide if the function could actually draw the template by using my previous function, if it wasn't able to draw the block it would return `TEMPLATES::INVALID` which meant, it would return nothing. Lastly, I would decide the next template:

    	int templatenumber = rand() % 7; 
	return (TEMPLATES)templatenumber;
This would essentially choose a random template from `TEMPLATES` (does not include INVALID template) to spawn next onto the playGrid.

## List Of Tasks That Have Been Completed

 - [x] Making blocks
 - [x] Making them fall automatically
 - [x] Making templates for all tetrominos
 - [x] Make them spawn automatically onto the playGrid
 - [x] Basic wall hit detection
 - [ ] Block to block hit detection
 - [ ] Basic Movement
 - [ ] Rotation
 - [x] Basic Aesthetics
 - [ ] Show next block
 - [ ] Make a 'hold block' function
 - [ ] Show held block
 - [ ] Points system
 - [ ] Special Effects?
 - [ ] Restart, Game Over
 - [ ] Make a 'Press Space To Start' text before starting the game

### Movement
Now lets start doing some basic movement, left and right and the instant drop, this was pretty quick and painless so let me run you through the basic code for movements.

Lets first start off with moving to the right:

    		if (IsKeyPressed(KEY_RIGHT)) {
			bool gotSpace = true;
			for (int i = 0; i < 23; i++) {
				for (int j = 9; j >= 0; j--)
					if (playGrid[i][j] == 'x')
						if (j == 9 || (playGrid[i][j + 1] != ' ' && playGrid[i][j + 1] != 'x'))
							gotSpace = false;
			}

			if (gotSpace)
				for (int i = 23; i >= 0; i--) {
					for (int j = 9; j >= 0; j--)
						if (playGrid[i][j] == 'x') {
							playGrid[i][j + 1] = 'x';
							playGrid[i][j] = ' ';

						}

				}
		}

Here I used raylib's key detection to determine if the user pressed the specified key or not, the first part of this piece of code would decide if you have enough space to move (e.g ig they hit the wall on the right side of the playGrid) as you can I checked every column and every row for 'x' and see if they were at a certain column, if they were in that column it wouldn't do anything and move on. If it had space it would move the 'x' one column to the right and clear the previous column the 'x' was in.

Now, I won't show the code for moving to the left since you can visualize from the code that its essentially moving the other way.

Finally, the insta drop:

    		if (IsKeyPressed(KEY_SPACE) && droppingSameBlock) {
			if (fastMeme) {
				PlaySound(goFastMusic);
				goFast = true;
			}
			else {
				lastTimePerDrop = timePerDrop;
				timePerDrop = 0.0;
			}
		}

Ignore the 'fastMeme' in the if statement, it was part of a joke feature that isn't part of my repo. So this could would change timePerDrop to '0.0' meaning it would drop instantly. Well thats it for today and this is the last checklist for today.

## List Of Tasks That Have Been Completed

 - [x] Making blocks
 - [x] Making them fall automatically
 - [x] Making templates for all tetrominos
 - [x] Make them spawn automatically onto the playGrid
 - [x] Basic wall hit detection
 - [ ] Block to block hit detection
 - [x] Basic Movement
 - [ ] Rotation
 - [x] Basic Aesthetics
 - [ ] Show next block
 - [ ] Make a 'hold block' function
 - [ ] Show held block
 - [ ] Points system
 - [ ] Special Effects?
 - [ ] Restart, Game Over
 - [ ] Make a 'Press Space To Start' text before starting the game
