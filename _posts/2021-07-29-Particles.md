---
layout: post
title: "Open_Pong: Particles Everywhere!"
date: 2021-07-29
author: "AsianMario"
categories: projects
tags: [proj]
image: particles.jpg
---

## Who doesnt love particles?

After finishing off all the current features I moved on to one of my favourite things to make, a particle system! Currently, the particle system can be emitted from any game object and can switch textures easily. The particles also have hit detection so whenever they hit something, individual particles will bounce of the object. At first, it was slightly complicated and hard to wrap my head around on how I could do it but after a while I got a hold of a reasonable idea and got to work.

{% highlight cpp %}
Particle::Particle(glm::vec3 position, glm::vec3 velocity, glm::vec3 scale, glm::vec4 color, float life) {
float random = ((rand() % 100) - 50) / 50.0f;
float randColor = 0.5f + ((rand() % 100) / 100.0f);

    this->position = glm::vec3(position + random);
    this->velocity = glm::vec3(velocity);
    this->scale = glm::vec3(scale);

    this->color = glm::vec4(color);
    this->life = life;

}

void Particle::draw(Game* g, VAO* vao, Texture* texture, Camera* camera) {
g->shaders[3]->Activate();
glBlendFunc(GL_SRC_ALPHA, GL_ONE);
glUniform2f(glGetUniformLocation(g->shaders[3]->ID, "offset"), position.x, position.y);
glUniform4f(glGetUniformLocation(g->shaders[3]->ID, "color"), color.x, color.y, color.z, color.w);
glUniformMatrix4fv(glGetUniformLocation(g->shaders[3]->ID, "proj"), 1, GL_FALSE, glm::value_ptr(g->cameras[0]->cameraMatrix));
texture->Bind();
vao->Bind();
glDrawArrays(GL_TRIANGLES, 0, 6);
vao->Unbind();
glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);

}
{% endhighlight %}

### Here. There. Particles Everywhere!

First off, I made a class which would draw and make a single particle. This class would then be used in the particle system to continiously spawn more particles using the same 'template'. The particle class takes the position, velocity, scale, color and life of the particle. To generate the particle from the position of the gameobject all I really need to do is feed it the position of the game object. The velocity determines how the particles travel, such as: if it'll be single file or scattered. Next up, is the particle draw function which uh... draws the particles. I also needed to make a new shader program that would render the particles, it was quite simple because it was quite similar to a normal mesh shader.

After this, I made a Particle System class that would assign a texture and limit a maximum amount of particles that could be rendered. The life variable would determine the amount of time the particle would be shown until it got deleted. The value would get subtracted by the delta time which makes sure the calculations were updated and correct no matter what. Once it dies it becomes a null pointer which makes it open to get filled in the memory. The particle system would check if any of the particles were null pointers and if they were they would be replaced by new particles. As for the collision detection I use the same detection method for the ball and paddle and was pretty effective. I plan to use the particles more other than just the ball trail to make the game more visually appealing.

Visual Result:

<iframe width="1280" height="720" src="https://www.youtube.com/embed/2Dp_QEFi4CE" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Hope you enjoyed reading!
