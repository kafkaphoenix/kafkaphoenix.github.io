---
title: My Journey into Game Development
description: A brief overview of my experiences, challenges, and milestones in the world of game development, from learning C++ to creating my own game engine.
slug: journey-into-gamedevelopment
date: 2026-01-22
categories:
    - About Me
tags:
    - About Me
---

## Starting out with C++

When I first tried to learn game development, it was during my university years. I already knew basic C/C++ from first-year courses. The first thing I completed was the book ["Beginning C++ Through Game Programming"](https://www.amazon.es/Beginning-C-Through-Game-Programming/dp/1305109910) by Michael Dawson. It was a good introduction to C++ programming concepts, but it was quite basic and didn’t cover graphics programming, only console applications.

After that, I moved to [SFML](https://www.sfml-dev.org/) and followed the book ["SFML Game Development"](https://www.amazon.es/-/en/Artur-Moreira-ebook/dp/B00DL0CFHC) by Artur Moreira. It was a good introduction to 2D game development using SFML, but after reading about half of the book (and combined with not using GitHub properly and not fully understanding pointers), I got stuck with an error for two days. I eventually found the issue: I was passing a reference by value instead of by reference (rookie mistake). That experience made it clear that I needed to deepen my understanding of C++ before moving forward.

That’s when I found [The definitive C++ book guide and list](https://stackoverflow.com/questions/388242/the-definitive-c-book-guide-and-list) on Stack Overflow and started reading ["C++ Primer (5th Edition)"](https://www.amazon.es/C-Primer-Stanley-B-Lippman/dp/0321714113) by Stanley B. Lippman, Josée Lajoie, and Barbara E. Moo. I didn’t finish the book, but together with Stack Overflow, cppreference, and other online resources, it helped me gain a much better understanding of modern C++.

The first truly cohesive tutorial I followed was [learnopengl.com](https://learnopengl.com/) by Joey de Vries. It covers the basics of OpenGL and graphics programming in a clear and concise manner, and I highly recommend it to anyone starting out. I restarted the tutorial thrice but never finished it completely (I plan to finish it alongside this project). I also briefly experimented with [SDL2](https://www.libsdl.org/) and followed some tutorials on [Lazy Foo’ Productions](https://lazyfoo.net/tutorials/SDL/), but I never completed a full project with it.

## First steps into game engine development and procedural generation

Up to this point, I had some experience with C++ and graphics programming, but I didn’t really understand game architecture or how to structure a game engine beyond a render loop and basic input handling. This is where I discovered ThinMatrix’s [OpenGL 3D Game Tutorial Series](https://www.youtube.com/watch?v=VS8wlS9hF8E&list=PLRIWtICgwaX0u7Rf9zkZhLoLuZVfUksDP). It was a great introduction to 3D game development using Java and LWJGL. I followed the entire series and built a simple 3D game engine from scratch. It taught me about game architecture, rendering techniques, gui, effects, and basic physics.

By binge-watching the series over a weekend, I didn’t fully understand all the topics. The engine also relied on Perlin noise to generate a plane with some mountains and trees, while my real interest was in voxel engines. On top of that, university exams were approaching, so I put game development on hold for a while.

Around that time, I also followed Sebastian Lague’s [Procedural Terrain Generation](https://www.youtube.com/watch?v=wbpMiKiSKm8&list=PLFt_AvWsXl0eBW2EiBtl_sxmDtSgZBxB3) and [Procedural Planet Generation](https://www.youtube.com/watch?v=QN39W020LqU&list=PLFt_AvWsXl0cONs3T0By4puYy6GM22ko8) series in Unity. While I enjoyed the concepts and finished the series, I realized that Unity wasn’t really for me, as I preferred working closer to the metal with low-level programming.

About a year later, I found [TheCherno](https://www.youtube.com/@TheCherno) YouTube channel and started watching his C++ series to revisit C++ concepts. By then, I was more comfortable with programming in general, so I jumped straight into his [Hazel code repository](https://github.com/TheCherno/Hazel/tree/master/Hazel) and began exploring the codebase while rereading [learnopengl.com](https://learnopengl.com/). I essentially started from the [learnopengl.com](https://learnopengl.com/) rendering code and basic game loop, then began copying ideas from Hazel and understanding them. However, once I managed to create a window with an OpenGL context and basic input handling, I got stuck while refactoring the window class. Combined with moving out of my parents’ home, I ended up dropping game development for another year.

## My first voxel game attempt

I think one major issue with many tutorials is what’s often called tutorial hell. I had the motivation, and the tutorials themselves were good, but once I got stuck, lost interest, or didn’t know what to do next or how to adapt the code, I would slowly drop the project and forget about it.

At this point, I had never actually tried to create a voxel engine. Since my Hazel-inspired engine was stalled, I decided to start a new project from scratch, this time focusing specifically on voxel engines.

Over the years, several YouTubers have created videos about developing their own voxel engines (the most famous is probably [Hopson](https://www.youtube.com/watch?v=GACpZp8oquU&list=PLMZ_9w2XRxiYzEuz4klbm8ZR7BfjueoN2)), but most of them don’t share the code, only share snippets, or publish a large GitHub repository with little explanation. Hopson even started a series but later canceled it because he wasn’t happy with how it was going. 

> I know he has a repository for his [one week challenge](https://www.youtube.com/watch?v=Xq3isov6mZ8) Youtube series, which I haven’t checked out yet, but I plan to during this project, as it seems simple enough to understand quickly.

But randomly I found HritikRC’s [How to code Minecraft](https://www.youtube.com/watch?v=1Dj-XL4XTxE&list=PLEtXCX1lakbhq_01JKJILx90wLfdwrJig) YouTube series. It’s far from perfect, and the JavaScript code is a bit messy, but it’s extremely easy to understand. It uses Three.js, which is a high-level library that abstracts most WebGL details. There’s no real rendering logic to implement, and the focus is purely on voxel engine concepts. You can also test each step easily in the browser. Unfortunately, after working on that project for about a week, I moved, and once again I stopped game development for a while.

## University final project

Then something changed: I had to complete my university final project (which I had never actually done, since I started working right after finishing my degree), and I thought revisiting game engine development would be a good idea. It also seemed like a good way to stay motivated, since I would be working on the project during my afternoons and evenings for half a year while working full-time. My plan was:
1. Resume the voxel engine idea by continuing where I left off (Hazel-style engine and [learnopengl.com](https://learnopengl.com/)).
2. Recreate the JavaScript voxel logic in C++ with OpenGL as a 3D demo using the engine.

After a few months, I had a semi-functional game engine with basic rendering, input handling, camera movement, model loading via Assimp, and configuration settings (enough to load a few 3D scenes, such as the [Sponza model](https://en.wikipedia.org/wiki/Sponza_Palace)). I had started exploring lighting and more advanced topics from [learnopengl.com](https://learnopengl.com/) when I got stuck on materials' reflections for a week. Frustrated, I put that course on hold and focused on finishing the engine.

I had also been hearing about Entity Component Systems (ECS) for years and wanted to experiment with the concept. I integrated [EnTT](https://github.com/skypjack/entt) and refactored the entire engine to use an ECS architecture, including loading scenes from files.

My goal was to have a prototype ready before contacting a professor for my final project, and he accepted. After discussing the scope, he suggested I focus on creating a debugging Graphic User Interface (GUI) with [ImGui](https://github.com/ocornut/imgui) and a simple game demo (a Flappy Bird clone) instead of a more complex 3D or voxel demo, given the limited timeframe. I worked on the GUI and game demo, which was both fun and a good learning experience for polishing the engine. In the end, everything went well, and I scored 9 out of 10.

## Newleaf and beyond

After that, I created a new engine called [newleaf](https://github.com/kafkaphoenix/newleaf), incorporating all the lessons I had learned from my previous projects. While it already included many improvements, I also left room for new ideas and future enhancements, as the engine is still very much a work in progress. After separating the engine code from the [demos](https://github.com/kafkaphoenix/newleaf_demos) into a dedicated repository, I decided to take a break and learn [Godot](https://godotengine.org/) to gain more experience with actual game development.

After eight years of going on and off with game development, different engines, and graphics programming, I’m still eager to create a voxel engine that scratches that itch. Over the past five years, I’ve made a few serious attempts to understand and build voxel engines, reading countless blogs and watching videos along the way. This blog and repository aim to summarize everything I’ve learned during that journey, so that someone in a similar situation can find a clear way to get started, a resource I struggled to find when I began.
