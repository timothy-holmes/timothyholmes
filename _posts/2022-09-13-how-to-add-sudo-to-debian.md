---
layout: post
title:  "Add Sudo to Debian/Ubuntu"
categories: linux debian 
excerpt_separator: <!--more-->
---

I was surprised, when I learnt that sudo doesn't come installed by default. I had to go and look up how to get it installed, and configured.
<!--more-->

I like `sudo`. When I type sudo, it's a strong message that I'm doing something that could have consequences for the entire system. The alternative is execute commands logged in as a superuser. I can't handle that much power. 

Here's how.

1. Log in as a superuser... `su -`
2. Update the package repository. Always good practice before installing something new: `apt-get update`
3. Install sudo using the package manager: `apt-get install sudo -y`
4. Add your regular user account (eg. timothy) to the _sudo_ group: `usermod -aG sudo timothy`
5. Logout from your superuser terminal... `exit`
6. Confirm user is added to the _sudo_ group: `id timothy`
7. Create a new terminal and test the new functionality: `sudo -s`

(BONUS) Remove password prompt from sudo'd commands:

8. Add new line to sudo config file: `echo "timothy ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers` <!-- markdownlint-disable MD029 -->
