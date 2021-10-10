---

layout: post

title: "Open_Pong: On Click Debugging"

date: 2021-10-09

author: "AsianMario"

categories: projects

tags: [proj]

image: GUI.png
---

  

# Cleaning Up

  

Before I dive into making the on click debugging I needed to cleanup and sort out how my game objects are stored, because right now doing `g->gameObject[42]` isn't great. I used a C++ map which would have a name that would store a pointer to a game object. Since I wanted it to be accessed globally I did need to make a class for it which wasn't much of a hassle.

{% highlight cpp %}

// GOList Class
 class GOList {
public:
	map<std::string, GameObject*> GOList;
};

{% endhighlight %}

Very simple. Next, I created a new cpp vector in `game.h` to store the map called `OBJList`, even though there was only one game object map this just made everything a little bit more neat. Then, I made a object in `main.cpp` called ObjectList (doesn't really matter what I called it) then did `g.OBJList.push_back(&ObjectList);` to store it. To add GameObjects to the map I simply did this: `ObjectList.GOList.insert(pair<std::string, GameObject*>(paddle1.name, &paddle1));` and... done! Although, I did need to add a new variable called `name` in `GameObject.h`.

### Bounding Boxes & Co-ordinate Confusion
Since I need to show the object properties when the mouse clicks on the GameObject I need to make a bounding box for every important mesh (except particles & powerup boxes). Of course, it's not ideal to manually create a bounding box for each mesh I made a function called `createBoundingBox();` in `GameObject.h`. I plan to make AABB (Axis-Aligned Bounding Boxes) which means they don't rotate and in theory much simpler. Ideally, the ends of the AABB would match the most extreme vertices from the mesh, I did something a little bit simpler:

{% highlight cpp %}

glm::vec4 GameObject::createBoundingBox() {
	glm::vec4 boundingBox = glm::vec4(1.0f);

	//RIGHT
	boundingBox.x = position.x + scale.x;
	//LEFT
	boundingBox.y = position.x - scale.x;
	//UP
	boundingBox.z = position.y + scale.y;
	//DOWN							   
	boundingBox.w = position.y - scale.y;

	return boundingBox;
}
{% endhighlight %}

Even though this would create bounding boxes that are a little bit bigger than the mesh it wouldn't be soo inaccurate it would cause a problem. Then I create another variable in `GameObject.h` called `BoundingBox`, its a vec4 variable which, you guessed it! Stores the mesh's bounding box. You'd think we're ready to implement the on click debugging now but there's a couple more stuff to do before we go on and do that. Next up, I have to translate mouse/screen co-ordinates to world co-ordinates. Since I decided to scale everything up early on due to mathematical errors the world co-ordinates are now 0-100 instead of 0-1, not a big issue though. I couldn't find the equation to translate screen co-ordinates to world co-ordinates until I stumbled on a 3D ray picker article which I modified a bit to make it work for 2D. Looks like this:

{% highlight cpp %}
void Screen::screenToWorldCord() {
	float x = ((2.0f * xpos) / 1920 - 1.0f) * 100.0;
	float y = (1.0f - (2.0f * ypos) / 1080) * 100.0;
	
	xpos = x;
	ypos = y;
}
{% endhighlight %}
Finally, we can move onto on click debugging.

### On Click Debugging
Finally, we're here! On click debugging! A bit complicated but i'll explain the best I can. All the way back in my blog post called 'GUI Update' I explained how I used IMGUI, probably best if you read it before reading the rest of this. Instead of creating debug windows one by one in my game scene I made it spawn everytime the mouse hovers over a mesh which will then show its properties which you can edit. I had to edit the `createDebugMenu();` function by changing its parameters from a GameObject to an OBJList so it make's it easier to implement. It looks like this now:

{% highlight cpp %}
void GUI::createDebugMenu( GOList* obj, string name, glm::vec3 orgPos, glm::vec3 orgRot, glm::vec3 orgScale) { //name has to match the same name in the GOList map
	{
		GameObject* objDebug = obj->GOList.at(name);
		ImGui::Begin(name.c_str());

		bool resetScale = false;
		bool resetPos = false;
		bool resetRot = false;

		ImGui::Text("Mesh Controls");
		ImGui::Dummy(ImVec2(0.0f, 5.0f));
		ImGui::SliderFloat("Scale X", &objDebug->scale.x, 0.0f, 200.0f);
		ImGui::SliderFloat("Scale Y", &objDebug->scale.y, 0.0f, 200.0f);
		ImGui::SliderFloat("Scale Z", &objDebug->scale.z, 0.0f, 200.0f);

		if (ImGui::Button("Reset Scale"))
			resetScale = true;

		if (resetScale == true) {
			objDebug->scale = orgScale;
		}

		ImGui::Dummy(ImVec2(0.0f, 5.0f));

		ImGui::SliderFloat("Translation X", &objDebug->position.x, -200.0f, 200.0f);
		ImGui::SliderFloat("Translation Y", &objDebug->position.y, -200.0f, 200.0f);
		ImGui::SliderFloat("Translation Z", &objDebug->position.z, -200.0f, 200.0f);
		if (ImGui::Button("Reset Position"))
			resetPos = true;

		if (resetPos == true) {
			objDebug->position = orgPos;
		}

		ImGui::Dummy(ImVec2(0.0f, 5.0f));

		ImGui::SliderFloat("Rotation X", &objDebug->rotation.x, -360.0f, 360.0f);
		ImGui::SliderFloat("Rotation Y", &objDebug->rotation.y, -360.0f, 360.0f);
		ImGui::SliderFloat("Rotation Z", &objDebug->rotation.z, -360.0f, 360.0f);
		if (ImGui::Button("Reset Rotation"))
			resetRot = true;

		if (resetRot == true) {
			objDebug->rotation = orgRot;
		}


		ImGui::Dummy(ImVec2(0.0f, 5.0f));
		ImGui::Text("Application average %.3f ms/frame (%.1f FPS)", 1000.0f / ImGui::GetIO().Framerate, ImGui::GetIO().Framerate);

		ImGui::End();
	}
}
{% endhighlight %}

Then, I made a new function called `onClickDebug`, this function will be later called in the main game scene. First, I check what scene i'm on, on click debugging only works in the Game scene. Then I make a new mouse state to check what i'm pressing on my mouse. Then I get the mouse co-ordinates then convert them to world co-ordinates. I then check if the mouse has been pressed, if it has then it will run a check on all game objects and see which mesh the mouse clicked on. Function to check if mouse is in a mesh bounding box:

{% highlight cpp %}

bool Screen::collisionBB(Game* g, GameObject* GO) {
	if (xpos <= GO->boundingBox.x && xpos >= GO->boundingBox.y && ypos <= GO->boundingBox.z && ypos >= GO->boundingBox.w) {
		return true;
	}
	else {
		return false;
	}
}
{% endhighlight %}
Finally, I render the debug window of the mesh. I need to make a few more improvements such as permanently rendering debug screens until the mouse clicks on something else. Current changes will be pushed to github and a video devlog will come along soon. Thanks for reading!


(also here's the function for the on click debug function)
{% highlight cpp %}

void GUI::onClickDebug(Game* g) {
	if (g->ScreenHandler[0]->screen == ScreenHandler::SCREENTYPE::GAME) {
		int state = glfwGetMouseButton(g->gameWindow, GLFW_MOUSE_BUTTON_LEFT);

		g->ScreenObject[0]->getCursorPosition(g);
		g->ScreenObject[0]->screenToWorldCord();

		cout << g->paddles[0]->boundingBox.x << endl;

		if (state == GLFW_PRESS) {
			for (GameObject* o : g->gameObjects) {
				bool mouseIntersect = g->ScreenObject[0]->collisionBB(g, o);
				if (o->name != "") {
					if (mouseIntersect == true) {
						g->debugGUI[0]->createDebugMenu(g->OBJList[0], o->name, glm::vec3(-0.75f, 0.0f, 0.0f), VEC3_ZERO, glm::vec3(0.015f, 0.2f, 1.0f));
					}
				}
			}

		}
	}
}
{% endhighlight %}
