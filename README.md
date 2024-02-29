# acenturyandabit's dev blog
Welcome to my dev blog! I'm acenturyandabit, a professional software developer with experience in C++, Python, Typescript, and making wonderful things come to life.

I write about my work and my opinions on software development; and I read and anthologize others' works. Come read my stuff! If you'd like to stay up to date, please send an email to [acenturyandabit.gmail.com](mailto:acenturyandabit.gmail.com) with your desired subscription frequency (every post; monthly; annually) and I will mail you when a new post comes out. 

I've got a few articles in the works, but I figured this project needs a README, so my first post is going to be about just that :)


# Give a dev a README 🐟, or teach a dev to README 🎣?
A README is the landing page of a project. Making a great landing page is typically in the domain of marketing teams, but if you are leading the development of a project (whether it's a large project used by thousands, or something small to help you land your first job), then you may be the best developer-facing marketing team your project will get. 

> A good README is a landing page that concisely gives a value proposition and a call to action.

A good landing page has two main functions: presenting the value proposition of the product, and providing a call to action that visitors can jump on. A good landing page also needs to deal with the realities of its readership - it is competing for a limited amount of attention, so it should be clear and concise. And so, so should a README!
## README decay occurs; effort needs to be invested to keep the README effective.
In my experience as a programmer, I've often been pointed at repositories with an inviting README.md file, only to find that the README either:

- Is just a blank file, or contains a single line with a heading of the name of the repository
- Hasn't been updated since the project took a major turn 3 years ago, and now describes something completely different
- Is a 30KB file which is the dumping ground for _all_ of the project's documentation, including developer docs, user guides, design docs and issue docs; 
- Is a README from a boilerplate with a whole lot of information about the boilerplate... but nothing about the project itself

This happens because even though a README is a landing page, a README is also code; and like any other code, READMEs naturally decay in value as the scope increases or changes.

The decay is extra pronounced if the README isn't maintained _for its purpose_ - to provide a solid introduction to the code for developers. Efforts to maintain the README should serve this purpose.

Since effort is a limited resource, one ought to allocate effort to the README proportional to the amount of utility the README provides. That is to say, if there are no new developers encountering the project, the README may not need to be updated in a long time.
## Making a boilerplate? Keep your READMEs extra lean! 
One of github's most prolific boilerplates is Create React App. Its README is 3.28KB over 70 lines; the total code in the boilerplate excluding the README is around 4.5 KB; making the README almost half the size of the project. (Prior to 2018 it was even larger - a whopping 126KB).

There are 247 public repositories with one of the title lines of CRA's README: "This project was bootstrapped with Create React App"; of which 13 of them contain the README mostly unmodified. 

CRA isn't the only culprit here - I have seen internal company boilerplates which also provide incredibly verbose READMEs. And I get the urge to extol the virtues of the boilerplate and give usage instructions - a lot of work goes into making a good boilerplate - but the moment the boilerplate has been instantiated, the project has a life of its own. It doesn't need a fat README to get it started - that belongs in the boilerplate cutter's README only.

Really the CRA README should be blank; with a single title and "This project was bootstrapped with CRA [link to docs]", and "Use npm run to start". When devs see the README file, they should think: "Hey, this boilerplate has given me some free real estate, time to write my own README!" rather than "I better not touch that README, it looks very official, maybe there's something I need there". This is the best a boilerplate README can do; especially since the boilerplate can't know in advance how it is going to be used.

Overall, boilerplates should be concise and fit for purpose; and arming folks with the skills to write a README is much more effective than providing a README template.

----
That's all for this time folks! If you enjoyed this, please star this repository and consider subscribing by sending an email to [acenturyandabit.gmail.com](mailto:acenturyandabit.gmail.com) with your desired subscription frequency. See you next time!