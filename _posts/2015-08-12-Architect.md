---
layout: post
title:  "Road to Be an Architect"
date:   2015-08-12
categories: Career
excerpt: career
comments: true
---

* content
{:toc}

## Learn how to become an software architect

[source](http://www.roberthalf.com/technology/blog/how-to-become-a-software-architect)

Q: What tips do you have for others who want to become software architects?

_Try to work with people better than yourself_. Read their code, look at their designs, question their decisions and absorb as much information as you can. Keep looking for mentors: in-person is best but books, blogs and social media also are great ways of finding mentors.

_Teach others_. If you want to learn something really well, teach it to someone else. Blogging, pair programming and whiteboard design sessions are all good ways of teaching. It's counterintuitive, but you'll receive way more than you give.

_Be willing to take responsibility_. When the next project comes up, ask to take the lead. It shows you have initiative and that you're willing to do more. Learn the strengths of the people on your team, and work to make them succeed.

Q: What new processes did you have to learn?

**Technical skills:**

_Be able to model your ideas_. Whether a UML document or the back of a napkin, you need to be able to describe the major components of a system and explain them to a 10-year-old. If you can do that, you can describe it to a team of developers or your CEO who's never seen an IDE.

_Lead by example_. If you're still coding, work responsibly. Follow coding conventions, write tests and document your work. I've seen architects who take the "fun" parts for themselves, and throw scraps to the rest of the team. Remember everyone wants to be proud of their work and learn, and it's your responsibility to help them do that.

**Non-technical skills:**

_Be able to communicate ideas to both technical and non-technical people_. 

_Watch presentations_. Attend conferences, or watch them online. Notice how the best speakers make the talk interesting. They're not just reading the slides; they're engaging the audience and telling a story. An architect has to "sell" their vision to the company's leaders – and they tend to be people with little time and short attention spans.

Q: What advice would you give to a mid-level programmer whose eventual goal is to become a systems architect?

If you're the smartest person on the team, seek others to work with. _Look for talented developers/architects_ that are willing to mentor and absorb as much as possible. Also:

_Read code_. Open source has made this so easy – find projects you're interested in and study the code. Look at how things are broken into classes, access patterns and overall structure of the project.

_Write code_. The only way to get better at coding is doing it. I've found that a good practice is to take a single idea and rewrite it using different styles, languages and techniques. Write it once using classes, another in a purely functional manner and as many ways you can think of. You'll learn what works best and how to solve problems in different ways.

_Build bigger things_. Look for projects that challenge your current skills. It's easy to do what you know, but it won't help you grow. An architect needs a wide variety of skills: front-end, back-end, distributed systems, operations and all the options around data storage. Test new technologies thoroughly before bringing them into projects – it's easy to follow the latest fad, but make sure it's really going to help your project.

_Meet people_. Attend local user groups and conferences, talk to other developers about what they're working on. You never know where the next opportunity could come from.

_Have a side project_. 

_Be open-minded_. Constantly question what you know and look for ways to improve.

Q: When designing the architecture for a new system, what is your thought process?

_Break large problems into smaller ones_. It sounds simple, but I think this is one of the primary traits needed to architect anything. See the big picture, and then dive deeper. Start with the high-level responsibilities, and continue breaking things down until everything has its purpose. 

Q: What tools do you use when designing the architecture for a new system?

I usually _start with whiteboard sessions with the entire team_ and have product managers, QA, operations and developers throw in opinions. Work toward a high-level picture of the system that everyone can agree upon. Further _design sessions happen with just the development team to work out details_. Visio is great for UML, database diagrams, flowcharts, network topology, etc. You'll need different designs depending on the audience (i.e. operations will want machines, network diagrams, firewalls, etc.). Balsamiq is good for mockup UI's – they look as if they're drawn by hand – which is great to let everyone get an idea of how things will look. It's fast to create and simple to change, perfect for early designs. Working code works great as well! I usually work very closely to create the first pages, services, and tests for the application. These serve as a model for the team moving forward.

Q: As an architect, which software methodology do you prefer?

Definitely **Agile**, but in its "pure form" that is light on processes. **Focus on solving the user's problem** – everything else is second. I like small iterations (one week) early in the project to make sure everyone stays on the same page, and then **two-week iterations** as things hit their stride. Agile accepts that change as part of building a system and embraces it. Automate as much as you can: developer environment setup, continuous integration and continuous delivery if you're ready for it. Writing the scripts to automate the process serves two purposes: working documentation and a process that is fast and repeatable. I'm not hardcore TDD, but I think **testing is important**. Having tests mean you can move quickly, stay flexible, refactor and change as needed. No one person needs to have the entire system in their head – the tests describe how it’s supposed to work and make sure that it does. Keep an eye on your test times – **slow tests and complicated setups usually mean there are opportunities to improve parts of the code.**
