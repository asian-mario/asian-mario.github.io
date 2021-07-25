---
layout: post
title: "Open_Pong: Improvements, Fixes & Scoring"
date: 2021-07-26
author: "AsianMario"
categories: projects
tags: [proj]
image: pongChamp.jpg
---

## Improvements & Fixes

For the past week I have been working on fixes and improvements to the game which I will explain in-depth. First up, all objects move at deltatime (the time in between frames) which allows all the object movements to still get calculated at the stated pace even if the frames drop or increase for any reason. I have implemented this by simply multiplying it's movement with the result of delta time. Quick example:

{% highlight cpp %}
//----------Ball Movement-------------
position += velocity \* (float) deltaTime;
{% endhighlight %}

Next up, decay and bouncing has been added to the paddle movement, the paddle no longer stops instantly as soon as you let go of the key and insteads gradually slows down (not too slowly to the point where it becomes uncontrolably). The velocity/force from the decay is reversed when it hit the top and bottom bounds of the game area giving an effect of a bounce. This also works if you already reach the top and bottom bounds by converting the length of the keypress to a total velocity.

Along with this, I have scaled up every object to reduce the risk of miscalculations due to the how low-value some the variables had (e.g 0.85f -> 85.0f). Because of this, I also found out I was using the wrong camera projection, instead of using a ortographic lens (used for 2D/2.5D) I still used a perspective lens (3D) which caused distortion and even more miscalculations when it came to hit detection. This also has been fixed with a two new ROML functions that is able to produce and calculate a Orthographic projection matrix with a clip bound and without a clip bound.

{% highlight cpp %}
glm::mat4 createOrto(float left, float right, float bottom, float top, float nearClip, float farClip) {
//Making it a zero matrix
glm::mat4 ortoM = glm::mat4(1.0f);

    	ortoM[0][0] = 2 / (right - left);
    	ortoM[1][1] = 2 / (top - bottom);
    	ortoM[2][2] = -2 / (farClip - nearClip);
    	ortoM[3][0] = -(right + left) / (right - left);
    	ortoM[3][1] = -(top + bottom) / (top - bottom);
    	ortoM[3][2] = -(farClip + nearClip) / (farClip - nearClip);

    	return ortoM;
    }

    glm::mat4 createOrto(float left, float right, float bottom, float top) {
    	//Making it a zero matrix
    	glm::mat4 ortoM = glm::mat4(1.0f);


    	ortoM[0][0] = 2 / (right - left);
    	ortoM[1][1] = 2 / (top - bottom);
    	ortoM[2][2] = 1;
    	ortoM[3][0] = -(right + left) / (right - left);
    	ortoM[3][1] = -(top + bottom) / (top - bottom);

    	return ortoM;
    }

{% endhighlight %}

Although this did fix the majority of collision/physics error I am planning to revamp the circle-to-box collisions to make it more versatile and more accurate due to how the ball can react weirdly when is collides with the paddle at less than ideal spots. I also plan it to become completely 'modular' since currently I am giving it a fixed height at the moment which isn't ideal if I plan to create power-ups that can rotate the paddle or affect its height. More improvements and fixes were made that were pretty minor which don't really need to be mentioned (mostly cleaning up code).

## Text Rendering & Scoring

This was pretty hard to do, after going back and forth deciding whether to render a simple Bitmap or to go with something more modern such as freetype & stb_truetype. In the end I chose stb_truetype although there were a abundance of guides on how to do it and I ended up learning from a German guide which led me to researching frequently for any code which I didn't get. Although, the modules and methods were quite old but I managed to pull through by using my current knowledge to make it work. Also due to the lack of truetype guides or tutorials on the internet that are easily available I plan to make one soon for anyone who wants to render 2D text in OpenGL.

After sorting out everything and debugging I started making a scoring system which depended if the ball had triggered out of the 'out-of-bounds' detections. The process wasn't too hard but I had to do some tweaking due to how I set up the text rendering. Here's a small snippet summary of what it looks like (sorry about the poor formatting):

{% highlight cpp %}
glm::mat4 createOrto(float left, float right, float bottom, float top, float nearClip, float farClip) {
//Making it a zero matrix
glm::mat4 ortoM = glm::mat4(1.0f);

//---------gameFont.h--------------
class gameFont{
public:
int score = 0;
glm::vec3 pos;
string scorestr;
const char\* text;
//etc.

}

//---------gameFont.cpp-----------
gameFont::gameFont(glm::vec3 pos) {
this->pos = glm::vec3(pos);
this->scorestr = to_string(score);
this->text = scorestr.c_str();
}

//---------Ball.cpp--------------

    if (checkRight == true) {
    	position = glm::vec3(0.0f);
    	velocity = glm::vec3(-60.0f, -rand() / control, 0.0f);
    	g->texts[0]->score++;
    }

    if (checkLeft == true) {
    	position = glm::vec3(0.0f);
    	velocity = glm::vec3(60.0f, rand() / control, 0.0f);
    	g->texts[1]->score++;
    }

{% endhighlight %}

I ended up making a whole class for the scoring and any text rendering (as you can see above) I needed just to make it neater in main.cpp, I also made a new gameObject for it to make it easier for other functions and classes to call or edit any aspect of it. After all the detection and scoring worked I made a reset or exit option after the game ends (either player scores 5 points).

Visual Result (note: the score reset bug has been fixed):

<iframe width="480" height="360" src="https://www.youtube.com/embed/CZx2U4x7I-8" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

This was a bit long but I hoped you enjoyed reading!
