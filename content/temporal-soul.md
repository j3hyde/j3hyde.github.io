Title: Temporal Soul:  The One Month Experiment Turned Two Plus Year Odyssey
Category: projects
Date: 2017-01-10 07:52:38
Status: published

Just as I started to write this I went to the commit log for the project.  The first commit was 24 November 2014.  Over two years have gone by and my "see what can be done in a month" project has become somewhat of a monster.  I went from drawing out concepts on paper to building a few JavaScript prototypes to building a game engine using [Win2D](http://github.com/microsoft/win2d) and [FMOD](http://fmod.org).  I'm still not finished but at least I'm focusing more on content and wiz-bang effects.  Although really, I should just release it.

# Resisting Release

I am the kind of person who tends to want things to be "just right."  Some might say that it is perfectionism and they would probably be correct.  The problem with a project like this is that I can spend maybe five hours per week on the project — ten at most.  Yet I have this need/desire/requirement for the game to look good; to play well; to be interesting and engaging.  The problem is that despite designing the game to be simple, it was probably still too big for where I am in my game-making journey.

So here I stand with a mobile game that is functionally in-tact, built for an effectively dead platform, and lacking content.  But that's okay because it's "almost finished."  Just one more tweak....

# The Game

Okay, so what is it?  I wanted to build something with a temporal mechanic.  I liked the idea of designing something around cycles.  In audio you can hear beat frequencies as two tones get near enough in frequency.  I wanted the game to center around that.  One interpretation is to design a game using audio in that way — and I may still.  However, in this case I wanted the mechanic to be more physical.

In the end I opted to use gamepieces that indicate their state:  A simple n-sided polygon where the gamepiece's wavelength is n.  Since the game's focus is on frequency and phase I used a simple matching mechanic (insert "match three eye roll" here).  Two adjacent gamepieces of differing frequency will eventually have a value of zero at the same time.  When this happens the pieces are removed, scored, and replaced with two new ones.

A couple challenges in the game are that pieces with wavelengths that are harmonic (e.g. one is wavelength 3 and another is 6) will only match if their phases are aligned.  For example:

<table>
<tr><th>Player Turn</th><th>3-Sided Gamepiece</th><th>6-Sided Gamepiece</th></tr>
<tr><td>1</td><td>1</td><td>4</td></tr>
<tr><td>2</td><td>2</td><td>5</td></tr>
<tr><td>3</td><td>3</td><td>6</td></tr>
<tr><td>4</td><td>1</td><td>1</td></tr>
</table>

As the player progresses, eventually the two pieces will reset to 1 and be matched.  However, if they are out of phase:

<table>
<tr><th>Player Turn</th><th>3-Sided Gamepiece</th><th>6-Sided Gamepiece</th></tr>
<tr><td>1</td><td>1</td><td>5</td></tr>
<tr><td>2</td><td>2</td><td>6</td></tr>
<tr><td>3</td><td>3</td><td>1</td></tr>
<tr><td>4</td><td>1</td><td>2</td></tr>
<tr><td>5</td><td>2</td><td>3</td></tr>
<tr><td>6</td><td>3</td><td>4</td></tr>
</table>

In that case they'll never match.

# But is it Fun?

That is my ultimate worry.  I have become a game designer by necessity.  I am not good at it but I will learn eventually.  Really I just want to ship it.  Fortunately, I think the mechanic is interesting and engaging.  For the pace of this type of game I think that is enough.  The fun comes from conquering something that is difficult to grasp or master.

In each level of the game the spawn rate for each "order" of tile (the wavelength mentioned above) is changed and the number of tiles you must remove is changed.  In some cases you might only get a 3-order tile once every six turns and yet you need those tiles to finish the level.  This adds some tension — and I dare say some aspect of fun — to the game.

# Current State / Future Additions

Currently I am doing more to add texture to the display of the game.  Although I want a simple asthetic, currently it is perhaps too simple.  I have also worked with someone on sound design for this game and I think the sound design is adding a nice atmosphere.  This produces a far richer experience than just the simple visuals aalone.  Lastly, I have been writing some dialog to go with the game.

Ultimately there is a story to be told and I hope that that even a short playthrough will be emotionally satisfying for the player.
