---
layout: post
title: "Open_Pong: Screen Handler"
date: 2021-09-25
author: "AsianMario"
categories: projects
tags: [proj]
image: paused.png
---

  

# Making The System

  

There isn't a game in the world with only one screen so I had to make a screen handler! I had some ideas running in my head for a while but I decided to make a 'Screen' template class, something similar to the GameObject class. All screens would need to inherit the Screen class to be able to render and to be able to render & update. Next would be the ScreenHandler, this handles switching screens & initializing them. The ScreenHandler class also stores all the types of screens you can use. Each screen needs its on class with 3 functions; a render function, a update function & a delete function. Here is what it should look like:

{% highlight cpp %}


class GameScene : public Screen {
public:
	void drawScreen(Game* g) override;
	void updateScreen(Game* g) override;
	void remove(Game* g);
};
  

{% endhighlight %}


## In-Detail

  

The draw & update functions are basically the similar compared to other draw & update functions so there isn't much to explain there. I had to make a new 'delete' function so I can get rid of the screen when I don't need it. Here's what it looks like:

{% highlight cpp %}

void Game::removeScreen(Screen* S) {
	for (auto c = ScreenObject.begin(); c != ScreenObject.end(); c++) {
		if (*c == S)
		{
			delete S;
			ScreenObject.erase(c);
			return;
		}
	}
}

{% end highlight %}

This will stop rendering the screen & stop updating it because it simply won't exist anymore, **but** it does not actually affect that are inside each screen's 'draw' function. If I deleted the game screen it wouldn't render or update anymore but the objects will still be where they were & remain in the exact same state until a switch and initialize the screen again.

### Whats Next?

Even though i've made the base system I haven't fully made the menus, so that's my next objective! Although, I'm also currently fixing my font renderer due to it having a multitude of issues which need some attention.