---
title: "Raylib-Tetris: Last Finishing Touches"
date: 2020-09-01
---
## Adding The Last Important Bits
So I've added almost all the core elements of the game but we still need the points system, in my code on my GitHub repo you may see some pieces of code as notes, these notes actually work but were not used because they had to be removed when I was uploading to Itch.io as I faced some issues compiling them for html 5. The points system is very simple, each line cleared equals 100 points, so I just have to add 100 after it detects that a line is cleared, but unlike python cpp cant combine a string and a integer into one element. So to fix this I wrote this:

    		std::stringstream scoreInt;
		scoreInt << scoreShow << scorePoints;
		DrawText(scoreInt.str().c_str(), 370, 240, 20, RAYWHITE);
I used stringstream to bind the integer variable and the string variable together, I then made it a C string not a cpp string, this would make it easier to use because raylib is originally built on C not cpp. 

    				// row clearing
				for (int i = 0; i < 24; i++) {
					bool rowIsFilled = true;
					for (int j = 0; j < 10; j++) {					
						if (playGrid[i][j] == ' ')
						{
							rowIsFilled = false;
						}
					}

					if (!rowIsFilled)
						continue;

					/*EndDrawing();
					blinkRow(playGrid, i);
					BeginDrawing();*/
					for (int j = 0; j < 10; j++)
						playGrid[i][j] = ' ';

					for (int x = i; x > 0; x--)
						for (int y = 0; y < 10; y++)
							playGrid[x][y] = playGrid[x - 1][y];
					scorePoints += 100;
				}
This is the piece of code for row clearing and as you can see at the bottom it just adds 100 to the integer variable for points everytime you clear a line. You can also see a piece of code which is written as a note, this is a piece of code that would blink the line after it was filled, I had to remove it because it didnt work well in HTML5 (blinkRow was a function that I made). The second part would clear the row that was filled and move the rows above down. Even though the special effect is not part of the compiled version I'll talk about it anyways.

    void blinkRow(char grid[24][10], int row) {
	float posY = ((float)684 / 24) * row;
	float time = 1.0f;
	while (time > 0.0f) {
		BeginDrawing();
		time -= GetFrameTime();
		if (((int)(time * 10)) % 2 == 0) {
			for (int j = 0; j < 10; j++) {
				float posX = ((float)285 / 10) * j;
				DrawRectangle(50 + posX, 50 + posY, ((float)285 / 10), ((float)684 / 24), BLOCK_COLOR_MAP[grid[row][j]]);
				DrawRectangleLines(50 + posX, 50 + posY, ((float)285 / 10) + 1, ((float)684 / 24) + 1, RAYWHITE);
			}
		}
		else {
			DrawRectangle(50, 50 + posY, 285, ((float)684 / 24), BLOCK_COLOR_MAP[TEMPLATE_CHAR_MAP[(TEMPLATES)(rand() % 7)]]);
		}
		EndDrawing();
		}
	}
This would essentially pause the time and blink the row by different colors from the color map by random, this would also be used to find where the blink the row is. The smaller the float time the faster it would end. Now let's get to the last tasks to finish.

## Making A Basic Interface
 First, I added text on top of the grid that said 'TETRIS', the colour of text would change corresponding to the color of the current falling block: 

    		
		DrawText("TETRIS", 50, 30, 20, BLOCK_COLOR_MAP[TEMPLATE_CHAR_MAP[curTemplate]]);
		
Next, I wrote something that could actually let the user restart the game:

    		if (!gameStarted) {
			if (!gameOver)
				PrintCenterText("SPACE TO START", screenWidth, screenHeight);
			else {
				scorePoints = 0;
				if ((int)comSec % 2 == 0)
					PrintCenterText("GAME OVER", screenWidth, screenHeight);
				else
					PrintCenterText("SPACE TO RESTART", screenWidth, screenHeight);
			}
			if (IsKeyPressed(KEY_SPACE)) {
				comSec = 0.0;
				gameStarted = true;
				if (gameOver) {
					// clear the grid
					for (int i = 0; i < 24; i++)
						for (int j = 0; j < 10; j++)
							playGrid[i][j] = ' ';
Here I also did code to start and restart the game, before this I made a boolean variable called `gameStarted` and `gameOver` which is default set to false. Next, I would clear the grid by just changing everything to an empty space. Finally, I added a few instructions on the side on how to play the game. So... I'm done! I'll talk about what I did next in the next post, see you!
