+++
title = 'Long Overdue OctoPrint Refresh'
date = 2025-01-08
draft = true
+++

## Project Overview

[Opening paragraph - why you did this. Mix of Python EOL warning being the trigger, but also realizing you hadn't touched your OctoPrint config in 6 years and it was time to modernize]

I've been running my 3D printer on the same OctoPrint instance since I bought it, six years ago. Somehow, I've never had to replace the pi or reimage it. This thing has been running almost 24/7 for SIX YEARS. No clue how it made it this far. A week ago I started getting messages that OctoPrint 1.12.0 and on wouldnt support Python 3.7 and 3.8. Rather than trying to upgrade Python in place, I figured it was time to clean it up.

While I was at it, I added a fan for the Pi (using a 3B+ that was getting pretty toasty), and fixed my OctoRelay setup to control the printer power. In Octoprint, I also wanted a UI update to expand on the relatively basic stock UI of Octoprint. The [UI Customizer](https://plugins.octoprint.org/plugins/uicustomizer/) plugin was super easy and did exactly what I wanted. Finally, after lasting so long on hopes and prayers, and going through multiple janky upgrades and fixes, I realized I never named my printer. As a huge nerd who names all of his network devices after sci fi ships, I picked Rocinante.

## What I Learned

- I never really considered the underlying dependancies of systems I manage going EOL. Luckily Octoprint 1.11.0 added a health checker that looks for issues in the underlying environment. If I hadnt seen that, I would have been scratching my head later on when the next upgrade failed or my plugins stopped working.
- Running a raspberry pi directly under a heated bed in a sealed enclosure wasn't the smartest choice I've ever made. The printed case it was running in was a little blackened and it felt way too hot when I pulled it out. Adding a fan should help out a little, but I'm probably going to want to move it outside the enclosure at some point.

## How I Did It

[The meat of the article. Tell the story chronologically but focus on decision points and interesting bits, not step-by-step instructions. Break it into natural narrative sections with regular paragraphs - no need for formal headers within this section unless it really helps the flow]

[Cover: the backup process, discovering it was a Pi 3B+ not Pi 2 B+, how easy the Raspberry Pi Imager made it, the restore process, adding the cooling fan,  plugin, renaming to The Roci]

## What's Next

- SMB share mounting for direct Cura-to-OctoPrint workflow
- Enclosure temperature monitoring via MQTT
- LED lighting control integration
- [Anything else you're thinking about]