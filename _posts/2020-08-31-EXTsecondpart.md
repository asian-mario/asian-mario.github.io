---
title: "Raylib-Tetris: Showing Next Blocks And Dark Mode"
date: 2020-09-01
---
## Raylib-Tetris: Showing Next Blocks And Dark Mode (AUG 31 EXT 2)
So let's start off with the simple things, I originally had a white backround and a black grid but I wanted it to be inverted which was a pretty simple thing to do, I just changed the color inside the the DrawRectangleLines function for the grid to white and the backround to black, it may seem unnecessary but at least it looks good. I've also seem some tetris games with ghost blocks or columns that was the size of one part of a tetromino block that would help the user know where the block would land. I chose to make the columns instead of the ghost blocks and I implemented it fairly easily. I did this by using the playGrid calculations (mentioned in the first and second raylib-tetris posts) and subtracted by one so it could fit inside the grid. I then matched the size of the grid to the size of one piece, I then separated the columns by doing a simple calculation. Now let's get to the more important tasks.

### Seeing The Future
With other tetris games on the interface it would show what the next three blocks are, but I couldn''t show the next three blocks due to how I set up the spawning, but I decided to do it anyway but instead of showing three blocks I'll just make it show one. So first, I made a function called `showBlock` it would have these parameters: `(int positionX, int positionY, TEMPLATES t)`. After creating the function and stating it's parameters I set up how it would display the block with this piece of code:

    	for (int curRow = 0; curRow < 2; curRow++) {
			for (int curCol = 0; curCol < 4; curCol++) {
				float offsetX = ((float)25) * curCol;
				float offsetY = ((float)25) * curRow;
		}
	}
I already drew the rectangle the blocks would already be in, the X and Y of the display area when divided by either the column or row would equal to 25. Next I had to make an if loop to check if the template array had an 'x' in it, then I placed into the grid with this piece of code:

    			// check if template grid has 'x'
			if (TEMPLATE_ARR[(int)t][curRow][curCol] == 'x') {
				// draw x onto the screen based on positionX + positionY
				DrawRectangle(offsetX + positionX, offsetY + positionY, 25, 25, BLOCK_COLOR_MAP[TEMPLATE_CHAR_MAP[t]]);
				DrawRectangleLines(offsetX + positionX, offsetY + positionY, 26, 26, RAYWHITE);

			}

  This would essentially use the template you've put in the parameters and put it on the rectangle, here is how I used it:
  

    		DrawText("Next Block:", nextBlockPreviewBoxX, nextBlockPreviewBoxY - 20, 18, RAYWHITE);
		DrawRectangleLines(nextBlockPreviewBoxX, nextBlockPreviewBoxY, 100, 50, RAYWHITE);
		if (nextTemplate != TEMPLATES::INVALID) {
			showBlock(nextBlockPreviewBoxX, nextBlockPreviewBoxY, nextTemplate);
		}
I've set these variables beforehand, you can see where everything is at in my repo.

# Tasks That Have Been Completed

 - [x] Making blocks
 - [x] Making them fall automatically
 - [x] Making templates for all tetrominos
 - [x] Make them spawn automatically onto the playGrid
 - [x] Basic wall hit detection
 - [x] Assign Blocks To Characters
 - [x] Assign Templates To Its Colour
 - [x] Block to block hit detection
 - [x] Basic Movement
 - [x] Rotation
 - [x] Basic Aesthetics
 - [x] Show next block
 - [ ] Make a 'hold block' function
 - [ ] Show held block
 - [ ] Points system
 - [ ] Special Effects?
 - [ ] Restart, Game Over
 - [ ] Make a 'Press Space To Start' text before starting the game

### Holding What You Didn't Want To Use
In classical tetris it did not have a 'hold block' feature but in newer releases and spin-offs it has, so I'm going to implement a 'hold block' feature into mine. 

    		//holding a template
		if (IsKeyPressed(KEY_H)) {
			TEMPLATES copyHold = holdTemplate;
			if (copyHold == TEMPLATES::INVALID) {
				holdTemplate = curTemplate;
				curTemplate = nextTemplate;
				ClearFallingBlock(playGrid);
				nextTemplate = spawnNextBlock(playGrid, curTemplate);
			}
			else {
				holdTemplate = curTemplate;
				curTemplate = copyHold;
				ClearFallingBlock(playGrid);
				spawnNextBlock(playGrid, curTemplate);
			}
			
		}

I chose the 'h' key to hold and release held blocks, first I made a 'copy' of the information from holdTemplate, then I assign different variables as holdTemplate, then I would clear the current one on screen and store it as copyHold. If I pressed it again (without INVALID::TEMPLATE) it would make the curTemplate to copyHold and spawn the next block. It isn't very complicated since this is basically just switching information between variables. I then did something similar on how I showed the next block but replaced the variables and text.

# Tasks That Have Been Finished

 - [x] Making blocks
 - [x] Making them fall automatically
 - [x] Making templates for all tetrominos
 - [x] Make them spawn automatically onto the playGrid
 - [x] Basic wall hit detection
 - [x] Assign Blocks To Characters
 - [x] Assign Templates To Its Colour
 - [x] Block to block hit detection
 - [x] Basic Movement
 - [x] Rotation
 - [x] Basic Aesthetics
 - [x] Show next block
 - [x] Make a 'hold block' function
 - [x] Show held block
 - [ ] Points system
 - [ ] Special Effects?
 - [ ] Restart, Game Over
 - [ ] Make a 'Press Space To Start' text before starting the game
