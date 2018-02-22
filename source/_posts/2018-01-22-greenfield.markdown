---
layout: post
title: "Slackcasa - chatting web app"
date: 2018-01-22 10:23:16 -0800
comments: true
categories: sideproject
---
#### Demo
[Try Slackcasa](https://slackk-casa.herokuapp.com/)

[Github](https://github.com/austinshin/slackk-casa)
##### Summary
- Created a full stack web application clone from scratch of [Slack](https://slack.com/).
- Worked with Trello (ticketing system), 3 other team members and implemented features for one week.

<!-- more -->

#### Stack
- React
- Express
- PostGreSQL
##### Features Added
- Login Authentication via PassportJS
- Passport Encryption via Bcrypt
- Route handling via React-Router
- Realtime Live Chat system between clients via Native JS Websockets
- E-mail notifications via Nodemailer and CronJob
- Unique workspaces (chatrooms) for users to join/chat
- Styling done via Bootstrap

## The Project Phase
Interesting decisions/challenges encountered.
1. Creation of a web application with skills learned from before for the first time.
2. Working as a group to create a vision for the application and working to make that vision come true.
3. Creating a ticketing system.
4. Modularizing and documenting code.
5. Git Workflow
6. Websockets, Postgres, Passport

This was my first time working with multiple team members to create a project.
The application goal was to create an interactive, seamless, lagless environment with users connecting and chatting to each other.
My goal was to practice communicating effectively, create a modular codebase such that other members could implement features without running into merge conflicts, and learn new technologies.

##### **Planning Phase**
My team members and I spent the entire first day talking about the project. I proposed the idea we try to plan as much as possible so that we have a good sense of the big picture. By creating a vision it helps when thinking about features to implement. You can then ask yourself, does this feature work towards the vision? If not, do we need to reconvene and reevaluate our vision? From past experience, having an end goal helps keep people focused and not lose their sense of direction.

By the end of the day, we had multiple features to implement ticketed in the perspective of a user. **As a user I'd like to have this feature...** i.e. As a user I'd like to be able to login. This obviously led to multiple features added on top of that, but having everything written out and ticketed allows for anyone to pick up the 'ticket' and start working towards it. Also, by separating everything out and planning most of it meant it was easy to keep things documented and modular.

Topics like
- 'What is our schema for pgsql going to look like?'.
- 'What are our variables name going to be?'
- 'What form is our data going to be sent in when communicating with the front end and back end?'

We took many pictures and uploaded them for easy viewing and later reference.
It was in a sense exhausting, but felt well worth it. Everyone felt on the same page which is super key in group projects.

#### _**Implementation**__
It turned out that some of these technologies we were working on were new to everyone. Since everyone wanted to learn and we had four people, it made sense to do some pair programming. Adopting the driver-navigator system we proceeded to split up the work on front end and database/back end. We switched around partners to learn about websockets and how they worked on both the front and back end.

#### _**Websockets**__
Websockets are really cool. I've worked on a chat system before but that was using native RESTFUL API where messages were updated via a setInterval. Now with websockets, everything was going to be realtime. It solves the problem of browsers being able to communicate to servers and servers to other browsers what seems to be instantly. This bi directional connection was the key to making our chat system work so it was naturally very important we implement it properly and in a way where we can reuse it when we need to implement other features.

We chose native websockets over sockets.io to get a better understanding of how websockets work.

#### _**PostgreSQL**__
Why postgres over MongoDB or MYSQL? Well, it made sense to use an ORM, but also a database in which you can join tables because of the nature of our project's schema. I already knew how to use MYSQL so I wanted to undertake a challenge of learning something new. The answer was PostgreSQL. It was fairly easy to pick up and intuitive to use. It shared many similarities as MYSQL (unsurprisingly).

As the mvp fell into place, my team members and I started to split off and work on separate features. I wanted to work on a notification system in which text and email notifications would be sent. I realized while I was planning it how much depth there would be when creating such a system. Having one type of notification system is hard enough, but add two? That's another layer of complexity. Then I thought about, what if you want to turn off notifications... or what if you wanted only specific notifications. I didn't have much time left, so I worked on what I could. Twilio or bandwidth is good to use for text notifications and I worked on it in the past. I decided to use nodemailer to send e-mail notifications and played around with cronjob to filter out certain emails and send them on a interval. It was cool (and spammy)!

### Wrap Up ###
- I learned a huge deal about planning and how long it could take. The project was seemingly small, but it took a whole day. I wonder what it'd be like in a big company with a big app.
- Keeping things modular allows you to keep to problem small, while maintaing a vision. It means you can implement new features separately without breaking the function of the app.
- Documentation is key because it means your team members can read and understand the code. This also means good code styling and function names.
- As a result of point 3, our project was picked the most by others in our class to reuse for their next project.
- Learning new technologies is always fun, and I was surprised at how good I've gotten at it. I'm aiming to get faster and better at it.
