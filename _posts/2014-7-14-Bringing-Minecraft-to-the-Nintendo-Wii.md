---
layout: post
title: Bringing Minecraft to the Nintendo Wii - A Developer's Journey
---
{:refdef: style="text-align: center;"}
![WoxelCraft Logo](https://wiibrew.org/w/images/f/fe/WoxelCraft_icon.png)
{: refdef}

Back in 2015, a friend of mine mentioned that there had been a few attempts in the homebrew community to run Minecraft on the Nintendo Wii, known as Wiicraft.
Unfortunately, these attempts hadn't succeeded due to the console's limited hardware capabilities. 
Despite not owning a Wii or playing Minecraft at the time, I found myself immediately drawn to the challenge.
The idea of running a voxel-based world on such a limited platform was fascinating and sounded like a fun challenge.

A year earlier, I had started learning graphics programming with OpenGL and C++.
This helped a lot since the graphics API on the Wii, known as GX, shares similarities with OpenGL.
Given that the name Wiicraft was already taken, I called my project WoxelCraft.

### WoxelCraft

Initially, I set aside the multiplayer/network aspect of the game and focused on implementing scenes, UI components, a player controller, and the chunk management + renderer for the world.
At the time, I was working full-time as a game developer and found it very fulfilling to develop everything from scratch,
without having to rely on existing frameworks or game engines like Flash or Unity (yes, Flash was still a thing back then) which we used at work.

Since I didn't have access to a Wii, I did most of my testing using the Dolphin emulator.
This approach enabled rapid iterations and provided a convenient workflow.

![Main Menu](https://wiibrew.org/w/images/thumb/6/68/WoxelCraft_Menu_0.0.2.png/800px-WoxelCraft_Menu_0.0.2.png)

![Ingame](https://wiibrew.org/w/images/thumb/f/f5/WoxelCraft.png/800px-WoxelCraft.png)

### Multiplayer and the Unity Client

As the project progressed, I eventually reached the point where I started working on the multiplayer aspect. 
However, debugging the netcode and packets proved to be challenging. 
The Minecraft protocol documentation and tools like Wireshark helped a lot, but to improve my development experience and gain a better understanding of the netcode, 
I decided to implement a small Minecraft client using Unity. 

For those interested, the code for the Unity-based client can be found [here](https://github.com/kperdlich/Minecraft-Unity-Client). 

Eventually, I got myself a Wii console. With access to real hardware, I was able to transition from testing on the emulator to testing on the Wii itself. 
This was crucial for validating functionality and performance on the actual device. 

<iframe width="981" height="552" src="https://www.youtube.com/embed/e4zTgiIqJGQ" title="WoxelCraft: A Minecraft Client for the Nintendo Wii" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

As the project grow, it became clear that integrating multiplayer into a codebase that was essentially single-player oriented presented significant challenges and
the joy I once had in the project began to fade.
This experience reflected a common belief among game developers: <i>If your game is going to have multiplayer, it needs to be built from the ground up as a multiplayer game.</i>

### Wiicraft (2020)

In late 2019, I decided to reboot the project with focus on multiplayer right from the start.
At the time, I had a few weeks off before starting a new job, so the timing was perfect.
Serializing all the chunk data from the server onto the SD card and then loading and rendering the player-visible chunks in a timely manner was the most complex part of the game.
After a few weeks of work I had a working prototype:

<iframe width="981" height="552" src="https://www.youtube.com/embed/lv3MmPLz3wI" title="Wiicraft Alpha 1.0 - Minecraft Client for the Nintendo Wii" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

There were still a few crashes here and there, but overall, it demonstrated it was possible to have a Minecraft client running on the Wii connected to a server.
I made the repository public and added a config file (for server IP, username, etc.) so that others could potentially use it as well.

### Cross-Platform Engine
With the prototype, my plan was to extend it into a cross-platform engine to further optimize my workflow and enable the use of more development tools.
This involved building a renderer capable of supporting both GX and OpenGL 3.3.0.

![Cross-Platform-Engine-1](/images/Wiicraft_OpenGL_Engine-1.png)

![Cross-Platform-Engine-2](/images/Wiicraft_OpenGL_Engine-2.jpg)

However, as time went on, my interest in the project slowly faded.
The prototype showed that it was possible to run a Minecraft client on the Wii and my initial motivation for the project disappeared.

Nevertheless, I still believe releasing the project to the public was the right decision, despite its imperfections. 
Keeping it buried in a private repository wouldn't benefit anyone.

If you're interested in the project or Homebrew development in general, feel free to check out the following links:
- [Wiicraft](https://github.com/kperdlich/wiicraft)
- [devkitpro](https://devkitpro.org/)
