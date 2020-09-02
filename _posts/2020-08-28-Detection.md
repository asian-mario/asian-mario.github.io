---
title: "Raylib-Tetris: Hit Detection"
date: 2020-08-28
---
## Making Walls Into Actual Walls
Following from the previous post, when the block hits the bottom it just goes straight through, so i have made a plan. The 2D array that i use are 24 rows tall and 10 rows wide, (note: i will call the arrays playGrid in this post and future posts) so i decided that if the 'x' on the playGrid hits the 24th row it will stop moving by changing them into another character. For example:

 

	for (int j = 10; j > 0; j--){
		if (playGrid[24][j] == 'x'){
			playGrid[24][j] = 'z';
		}
	}
		
This in short, would detect if there was 'x' in the playGrid and change it into another letter therefore it could no longer move.

It looks simple but i worked the ground hit detection for quite a while and i had some things to do today so this is all i have right now.

# Changelog
- Added bottom hit detection for blocks
