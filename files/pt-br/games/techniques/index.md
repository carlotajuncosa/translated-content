---
title: Técnicas para desenvolvimento de jogos
slug: Games/Techniques
tags:
  - NeedsTranslation
  - TopicStub
translation_of: Games/Techniques
---
{{GamesSidebar}}{{IncludeSubnav("/en-US/docs/Games")}}

This page lists essential core techniques for anyone wanting to develop games using open web technologies.

These are the Ideas and Technique for game devlopment

- [Using async scripts for asm.js](/pt-BR/docs/Games/Techniques/Async_scripts)
  - : Especially when creating medium to large-sized games, async scripts are an essential technique to take advantage of, so that your game's JavaScript can be compiled off the main thread and be cached for future game running, resulting in a significant performance improvement for your users. This article explains how.
- [Optimizing startup performance](/pt-BR/docs/Apps/Developing/Optimizing_startup_performance)
  - : How to make sure your game starts up quickly, smoothly, and without appearing to lock up the user's browser or device.
- [Using WebRTC peer-to-peer data channels](/pt-BR/docs/Games/WebRTC_data_channels)
  - : In addition to providing support for audio and video communication, WebRTC lets you set up peer-to-peer data channels to exchange text or binary data actively between your players. This article explains what this can do for you, and shows how to use libraries that make this easy.
- [Efficient animation for web games](/pt-BR/docs/Games/Techniques/Efficient_animation_for_web_games)
  - : This article covers techniques and advice for creating efficient animation for web games, with a slant towards supporting lower end devices such as mobile phones. We touch on CSS transitions and CSS animations, and JavaScript loops involving {{ domxref("window.requestAnimationFrame") }}.
- [Audio for Web Games](/pt-BR/docs/Games/Techniques/Audio_for_Web_Games)
  - : Audio is an important part of any game — it adds feedback and atmosphere. Web-based audio is maturing fast, but there are still many browser differences to negotiate. This article provides a detailed guide to implementing audio for web games, looking at what works currently across as wide a range of platforms as possible.
- [2D collision detection](/pt-BR/docs/Games/Techniques/2D_collision_detection)
  - : A concise introduction to collision detection in 2D games.
- [Tilemaps](/pt-BR/docs/Games/Techniques/Tilemaps)
  - : Tiles are a very popular technique in 2D games for building the game world. These articles provide an introduction to tilemaps and how to implement them with the Canvas API.
