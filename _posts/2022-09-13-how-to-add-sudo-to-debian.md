# Add Sudo to Debian

I like `sudo`. When I type sudo, it's a strong message that I'm doing something that could have consequences for the entire system. The alternative is execute commands logged in as a superuser. I can't handle that much power. I was surprised, and then not so surprised, when I learnt that sudo doesn't come installed by default. I had to go and look up how to get it installed, and configured.

Here's how.

1. Log in as a superuser... `su -`
2. Update the package repository. Always good practice before installing something new: `apt-get update`
3. Install sudo using the package manager: `apt-get install sudo -y`
4. Add your regular user account (eg. timothy) to the _sudo_ group: `usermod -aG sudo timothy`
5. Logout from your superuser terminal... `exit`
6. Confirm you have been added to the sudo group: `id timomthy`
7. Create a new terminal and test the new functionality: `sudo -s`

TODO: add reason why update/upgrade is good practice.
