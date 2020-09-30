![image](https://user-images.githubusercontent.com/71992673/94647701-e03ebf00-02a5-11eb-8167-82b74b849a1b.png)

Day 1 - Installing GitHub, VS Code, and finding a way to make it work (Part 2)

## Introduction

The Why? Well this is my first time installing and configuring GitHub and VS Code. I thought I had the hang of it yesterday, but I did not get the desired result. Back at it again today, hence the 'Part 2' in the title. 

## Prerequisite

The base knowledge, in my opinion, is how to google things you don't know about. I know almost zero regarding GitHub and VS Code outside of knowing that these are dev tools and given my experience that is rooted heavily in infrastructure I have not used these tools before. 

## Use Case

- The use case would be the first day on the job as a junior DevOps Engineer. Here's a laptop, install GitHub, programming tool of choice, create a repo, and document the steps. Not sure if this is actually what would happen in a real-life scenario, but I will go with it. 

## Cloud Research

- In the mini-tutorial I document my steps, errors, and fixes to resolve those errors. Some of the fixes were to restart the VS Code app and in other cases google and find out what others did to resolve the issue. 

## Try yourself

Below is my mini-tutorial. This follows the Youtube [video](https://www.youtube.com/watch?v=smA_MGTgcYM&list=PLEF6pxCxNXw2hkVzyAKOL5J1AqtCejVG_&index=2&ab_channel=100DaysOfCloud&t=271s) created by @madebygps. I started documenting the steps below are around the 0:52 mark in the video. This is on a Mac. 

### Step 1 — Setting up the GitHub Account

My first roadblock was I was getting an error when I attempted to use the '100DaysOfCloud' for the repository name.  It initially gave me an error saying I can’t use the tag. The fix (for me anyways) was to register my GitHub account first and then the tag worked. Yesterday, when I first created the repository, I did not check the box include all the branches. I ended up have an incomplete README.md file. I deleted everything, started fresh, checked that box, and it loaded the complete README.md file, but I hit another snag as you will see later on in this tutorial. 

### Step 2 — Install Visual Studio Code (VS Code)

She mentions using visual code. I have a Mac and have not installed that app before. 
I did a google search and downloaded the app from here https://code.visualstudio.com/download. I then created a 100DaysOfCloud folder on my desktop similar to what she has setup. Initially I tried using the built in Terminal app in Mac and navigated to the 100DaysOfCloud folder on the desktop and ran the git clone command, but it failed. If that worked I probably wouldn’t have installed Visual Studio.

### Step 3 — Configure VS Code to talk to GitHub

Once the install was complete. I opened Visual Studio and in the left column there is a source control button that looks like this:

![image](https://user-images.githubusercontent.com/71992673/94648656-d5852980-02a7-11eb-8dd3-b0401ab04d0b.png)

The first time you open it advises you to install git. Another google search for git took me here https://sourceforge.net/projects/git-osx-installer/. Once I installed it I had to close and restart Visual Studio. I clicked on the source control button again and this time it gave me an option to either Open Folder or Clone Repository. I assume this is the same command as git clone so I clicked on Clone Repository. Another box pops up to provide the URL or source:

![image](https://user-images.githubusercontent.com/71992673/94648739-fc436000-02a7-11eb-8e13-232fc9e8c7db.png)

I copied the URL from GitHub into the box and then navigated to my 100DaysOfCloud folder on my desktop for the repository location. 

![image](https://user-images.githubusercontent.com/71992673/94648808-2dbc2b80-02a8-11eb-8944-570148edbd49.png)

In the video, @madebygps uses a terminal to create a new branch and then opens VS Code. I already have VS Code open so I went to the Menu - Terminal - New Terminal and it opens a Terminal window at the bottom. I entered the commands per the video below:

![image](https://user-images.githubusercontent.com/71992673/94649100-d2d70400-02a8-11eb-888e-7ab48be246cf.png)

The one mistake I had yesterday was I had a space between 'Day' and '1'. I think that is why none of my changes were saved or committed to GitHub. This time as you can see above its all together. @madebygps does this step, but I thought I could have the space. Maybe if I put it like this 'Day 1' it could work. Maybe someone can confirm that for me. 

I then followed the terminal commands in the rest of the video starting at 4:09.

## ☁️ Cloud Outcome

I learned that making mistakes is part of the process and sometimes you have to repeat the process multiple times before you figure out why its not working. The video definitely helps, but if you know little to nothing about how GitHub works google is your friend. I do not see myself as a Dev, but the Cloud is about software. One of my goals is to be proficient in IaC (Infrastructure as Code) so knowing how to use the tools I installed is a must. 

## Next Steps

I think my next step will be to see if I can get what I'm writing right now in VS Code to commit, save, and merge to my GitHub LOL! Once I'm past that step I'm going to try one of the Unicorn in the 100DaysOfCloud list. 

## Social Proof

[Tweet](https://twitter.com/harristha1/status/1311190433418993664?s=20)
