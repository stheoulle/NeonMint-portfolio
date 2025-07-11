---
layout:  /src/layouts/ProjectLayout.astro
title: 'Twitch DataViz'
pubDate: 2025-07-09
description: 'Side project to visualize Twitch data using Python and gephi.'
languages: ["python", "gephi"]
image:
  url: "images/projects/twitch.webp"
  alt: "Thumbnail of Twitch dataviz app."
--- 

# Twitch DataViz

Twitch DataViz is a side project I developed to visualize Twitch data, focusing on the relationships between streamers and their communities. The project uses Python for data processing and Gephi for visualization.

## Overview

The project focuses on collecting data from Twitch to examine how streamers are linked with their viewers and how it demonstrate the different content they broadcast. Using Python, the data is processed, and the relationships are visualized with Gephi.

A Python script retrieves all the followers of a streamer, and then these followers are compared to see if they follow each other. If they do, a connection is established between them.

This visualization helps identify communities, highlighting patterns where a streamer with different concepts on their channel attracts different communities.

In each graph, nodes represent the streamer's followers, and edges depict the relationships between them. The size of each node corresponds to the number of followers a person has.

All the graphs are stored on : <https://www.easyzoom.com/profile/134895>

🚀 *Developed by Light.*
