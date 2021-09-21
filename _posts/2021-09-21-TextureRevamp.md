---
layout: post
title: "Open_Pong: Texture Revamp"
date: 2021-09-21
author: "AsianMario"
categories: projects
tags: [proj]
---

# Visual Overhaul

This post should've been made a couple weeks ago, currently working on a scene handler system. Since I wanted this to be a modern re-imagining of Pong, I wanted to have a nice colour palette instead of black & white. I choose a nice blue to dark purple gradient to be the colour palette of the game and implemented it in all of the textures in the game. It was pretty easy to implement although tedious due to the way I set up the way the lighting shader and how it works in the game. My plan was the change the colour of the object (except the ball) depending on where the object is in the game, e.g left = blue, right = dark purple. First I applied a gradient texture onto the barriers then for the paddles, since they wont move on their X-Axis I assigned them both a static colour that would match the gradient on the barriers. For the particles that spawn when you hit the barrier I changed the default texture color to white so I can let the shader change the color on the go without needing to switch textures. Depending on where the particle will spawn I will change the colour of the particle, I implemented this by having a colour vector that will change its value depending on the position of the spawning particle. Here is the snippet of code:
{% highlight cpp %}
	if (limitSpeed == true) {
		

		if (g->balls[0]->position.x <= -25.0f) {
			color = glm::vec4(0.66f, 0.66f, 0.992f, 1.0f);
		}

		if (g->balls[0]->position.x >= -24.9f && g->balls[0]->position.x <= 20.0f) {
			color = glm::vec4(0.46f, 0.46f, 0.956f, 1.0f);
		}

		if (g->balls[0]->position.x >= 20.1f && g->balls[0]->position.x <= 40.0f) {
			color = glm::vec4(0.48f, 0.125f, 0.76f, 1.0f);
		}

		if (g->balls[0]->position.x >= 40.1f) {
			color = glm::vec4(0.6f, 0.17f, 0.93f, 1.0f);
		}
	}

{% endhighlight %}
Of course the transition between the colours won't be perfect, to combat this if it doesn't match any of the criteria I set the color vector to '0' so it doesn't appear at all. I can do the same thing for the ball particles and all I needed to do was plug in the 'color' variable so it would change dynamically.
## Improvements & Fixes

After all the texture revamp I went ahead and worked on some more fixes. First off, I fixed the velocity limits of the ball so it decreased the chance of crazy behavior like going off screen, I also added a function that would boost the ball's velocity if the ball was travelling too slow, this fixed the problems with the ball being stuck in the middle of the screen while not hitting anything. Next, I added a few more powerups, two of them being "Butterfingers" & "Pong Smash!". Butterfingers would increase the decay of the paddle for the opponent paddle, making the controls slippier and harder. The Pong Smash powerup allowed the player to release the speed limit of the ball for a few seconds and would allow the player the hit the ball with full power . Unlike the others, the Pong Smash powerup isn't activated on-hit, you have to press a key to activate it ( which is 'U'). Once pressed, the particles behind the ball will turn red for a few to indicate the powerup is active.

### Whats Next?
Currently, I'm working on a scene handling system which can switch scenes and initialize scenes e.g from the Main Menu scene to the Game scene. Hoping to get it fully working in a week's time after this post. Hope you enjoyed it!