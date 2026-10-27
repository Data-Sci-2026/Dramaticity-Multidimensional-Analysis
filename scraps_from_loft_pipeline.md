# Scraps from the Loft Pipeline
Zach Shultz
2026-10-25

prereqs

``` r
library(tidyverse)
```

    ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ✔ dplyr     1.2.1     ✔ readr     2.2.0
    ✔ forcats   1.0.1     ✔ stringr   1.6.0
    ✔ ggplot2   4.0.3     ✔ tibble    3.3.1
    ✔ lubridate 1.9.5     ✔ tidyr     1.3.2
    ✔ purrr     1.2.2     
    ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ✖ dplyr::filter() masks stats::filter()
    ✖ dplyr::lag()    masks stats::lag()
    ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(rvest)
```

    Attaching package: 'rvest'

    The following object is masked from 'package:readr':

        guess_encoding

## Intro

This document is a place for the pipeline to create the Scraps from the
Loft data frame.

## Reading in the Data

The code below shows how to do this with html filling in for a link to
the page with the transcript. For the purposes of this document, we can
look at the test page I used, the Adults Episode 4 transcript.

``` r
html <- "https://scrapsfromtheloft.com/tv-series/adults-s01e04-house-rules-transcript/"

show <- read_html(html)
  #reads in the website
```

## Finding the right Elements and Transformation

Using selector gadget, I was able to find the correct elements I wanted
to target fairly quickly as shown in the code below.

``` r
show_script <- show |>
  html_elements(".single-content p") |>
  #find all elements on the web page that are actually dialogue
  html_text2()
  #turn those elements into only text

show_df <- data.frame(show_script) 
  #turn our character vector into a df
```

I also transformed this into a data frame. While this probably wasn’t
strictly necessary, it made it easier to borrow code from the [Santa
Barbara Pipeline](santa_barbara_pipeline.qmd) which saved time and
energy. Hopefully, it also makes these twin pipelines easier to follow.

## Cleaning the Data

The next step was to eliminate the first four lines. Luckily, in every
transcript the first four lines are dedicated to the same structure,
making it easy to leave only the text we want.

``` r
show_df <- show_df |>
  slice(-(1:4))
  #removes the first four rows which are not actually dialogue from the show
```

``` r
clean <- function(string) {
  string |>
  str_replace_all("\\[.*?\\]", "") |>
  #remove a [ followed by any number of any characters and then a ]
  str_replace_all("♪[^♪]*♪", "") |>
  #removes a ♪ followed by any number of characters followed by ♪
  str_replace_all("\\([^)]*\\)", "")
  #removes a ( followed by any number of characters followed by )
}
```

## The Final Function and Iteration

``` r
get_clean_script <- function(html) {
  
  show <- read_html(html)
  #reads in the website
  
  series <- str_remove(html, "https://scrapsfromtheloft.com/tv-series/") |>
            #removes the first part of the html link which is identical accross transcripts
    str_replace("-s\\d.+-", "-") |>
    #replaces -s followed by a digit followed by any number of characters and then a - with a single -
    str_remove("/") |>
    #removes remaining /
    str_replace_all("-", "_")
    #replaces all -'s with _'s
  
  show_script <- show |>
  html_elements(".single-content p") |>
  #find all elements on the web page that are actually dialogue
  html_text2()
  #turn those elements into only text
  
  show_df <- data.frame(show_script) 
  #turn our character vector into a df
  
  show_df <- show_df |>
  slice(-(1:4))
  #removes the first four rows which are not actually dialogue from the show
  
  #defines a function for cleaning each string in our df
  clean <- function(string) {
  string |>
  str_replace_all("\\[.*?\\]", "") |>
  #remove a [ followed by any number of any characters and then a ]
  str_replace_all("♪[^♪]*♪", "") |>
  #removes a ♪ followed by any number of characters followed by ♪
  str_replace_all("\\([^)]*\\)", "") |>
  #removes a ( followed by any number of characters followed by )
  str_replace("^[A-Z]+ ?[1-9]?:", "")
}
  
  show_df <- show_df |>
    mutate(show_script = 
             clean(show_script)) |>
    #cleans up all the lines with the function above
    mutate(show_script = 
             trimws(show_script)) |>
    #removes extra white space
    filter(!show_script == '') 
    #removes lines with no text
  
  show_df |>
    mutate(script = "x",
         .before = show_script)
  #add a column fill in for unifying these features

  show_df <- show_df |>
  summarize(show_script = str_flatten(show_script, " "))
  #returns a single cell data frame of all of the dialogue combined with spaces
  
  show_df <- show_df |>
  mutate(id = series,
         #adds a new column that gives the id based on the series
         .before = "show_script") |>
  rename("clean_words" = show_script)
  #renames to clean_words so we can rbind this thing with the santa barbara df eventually making one big df!!!
  
  show_df
}

get_clean_script(html)
```

                     id
    1 adults_transcript
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    clean_words
    1 You’re flossing to me? Are you flossing to me? I’m the only one here. Ah.  Okay. Guys, no. House rules! What are we doing? It’s a tradition, you’ll see. Upstairs now! Let’s go! Oh, my God. Thank you so much. Strong board this year, guys. Yeah. Awesome. Okay, we mix it. Okay. Homeowner goes first. Paul Baker. Yeah. Your first rule. Can’t believe I have to say this, but… No fucking on the couch. Yes, but no. Okay, thematically similar. Stop leaving condoms in the toilet. Copy that. And the shower. That’s on me. And in the sink that one time. My bad, big bro. See, I told you we have to stop using protection. No. What? So ruled? So ruled. So ruled. And then you drink from the Springsteen on Broadway mug, which is the most sacred drinking vessel in the house. Drink up. Go! Go! Go! Whoo! Oh. It’s bad, bad. Okay, who’s next? Issa. Issa. Me? Lots of options here. Um… Opening it up to the floor under Article 3-6. Oh, I know. She’s got to stop saying the F slur. I’m allowed to say that. No, I grandfathered her in. It’s fine. She has to admit her parents are rich. They’re upper middle class. It’s not… I cannot have this fight again. Then, she has to let us track her. Oh. Yeah, guys, please. I keep losing her in multi-floor Zaras. Okay, totally. You guys can track me right now. I just have… Wait, where is my phone? Okay, so we will track her with an AirTag. Wait, what? So ruled? So ruled. Bruce her. Attagirl. Drink up. Anton. Yes. Okay, yes. You have to break your dry spell. Fuck off. Because you’re getting kind of snippy. Fuck off, girl. And also your standards are impossibly high. Okay, smart, kind, so gay that it turns a corner and becomes straight again. What is the problem with that? Anton… So ruled? So ruled. Fine. I’m horny. Paul Baker! Okay, Samir… I know, I know. We don’t have to do the whole thing for me, ’cause I already know I gotta… Get a job! I gotta get a job, yeah. Billie. Yes. You must stop visiting your old high school. Give me the gavel. No. No. No more check-ins. No more knock, knocks. I mean, what even is a “knock, knock”? Knock, knock. Just in the area. Hi. It’s strange, love. Thank you. If I stop going, they’ll think I’ve died. That’s okay. Okay, so ruled? So ruled. Starting now. You got this, Bills. No more knock, knocks. No more… Knock, knock. Hey, Billie Schaeffer. How the hell are you? Yeah, good. Just in the neighborhood, thought I’d, uh, swing by. Yeah, please, come on in. Oh, my God. I’d be honored. Hey, guys, this is… Hey, Kelsey.  Headphones. Guys, this is Billie. Billie is, no joke, one of Garfield’s best and brightest alumni. We had these desks. In fact, actually, Billie here was editor in chief of the paper for three years, was it? Yeah, wow. Three years, yeah. Three years. God, best years of my life. So, she pretty much taught me how to do my job. Oh, my God. Stop. How’ve you been? How’s-how’s, uh, the big, bad world? Everything is great. Um… Everything is going according to plan. So… Great. Good for you. I mean, I was gonna head out, but… I mean, if you guys have any questions, I mean… No, not if you’re on your way out. You go. We’re fine. I mean, I have a minute. Okay. I have a minute. Yeah. Okay, okay. Yeah, that’d be great. We would love it. We’ll take it. Guys. Hit me with your best shot. We’re on a deadline. Ooh. Kelsey, that’s a little rude, okay? So we’re gonna call that strike one for the day, all right? Yeah, Josh. Hi. Josh Rapaport. Do you have any tips for interviewing people? Do I have tips? Yeah. Yeah, I have tips. So, sophomore year, I taught myself shorthand. That’s right. Google “Hot to Jot” Cheryl Koenig. Game changer. Yes, over here. Question? Um, didn’t you stop by my Spanish class last week? Yeah. Yeah, you were there last week. I remember seeing you. No, I don’t think I did. Did I? I don’t know. Yeah, you asked if we were up to vosotros . Don’t think that that was me. Do we have any other questions? Yes. Yeah, why do you care if we’re up t vosotros ? No, you know what? Why don’t we keep it on track, shall we? Avery is right. She’s here a lot. It’s weird. Okay, so that feels like strike two.  Why? How is Rachel Sheffield always on a fucking cruise? I got an interview. What? I got an interview at, like, a… at a company. No, I can’t hear you. I have an interview at a company. Like a fintech company. Wait, what? That’s amazing. I know. Picture me, fintech. Can you see it? I’m at a desk. Mm-hmm. Bunch of staplers, little ones, big ones. Staple remover as well. ‘Course. I’m sitting there, “Hi, I’m Samir. I work at fintech.” My voice is different. It’s “in fintech.” But yes, I fully pictured it. Wow.  It’s great. Okay. This is crazy. I just gotta look up what the “fin” means. Mm-hmm. Look that up. So now it’s between Abby P and Abby F for the lead of Popcorn Forest the Musical. But it’s just, like, Abby F’s parents are in a commune. She isn’t well. And done. You are officially airtagged. I love it. Wait. It’s actually so chic. Should we take it for a test-drive? Yeah. Go to Sephora. Somewhere scary. Oh, my God. I love how you’re speaking. Uh-huh. That sounds so fun. I love it. What? Do we know him? Oh, my God. Look. No, but that’s, like, literally Anton’s type. Well, he’s with a girl though. Well, I started to, like, notice people again. And it’s the first time since Michael that I feel like I’m ready to get back out there. Michael. He’s gay. He’s gay. And this is why I’m like, “Everyone should be in therapy.” I know. Wait, are you still… What does he say? I can’t. Tomorrow, I’m doing that park cleanup. All right. Yeah, you know. Hey, you want to walk with me? Yeah. Oh, shit, they’re leaving. Wait, what do we do? No, babe, don’t… Don’t. Don’t! Don’t! No, you did not. I had to. You did not have to. I really did. We are in a post-de Blasio, pre Avatar III moment. We have to live. What does that have to do with this? No, look at me. We have to live. That’s, like, a hundred bucks. Okay, the mailbox is so full, you guys. We actually have to think about opening that soon. He’s here! Welcome. We’ve been waiting. Whatever it is, no. I have work. No, no, no, so do we. We have been working to break your dry spell, actually. Allow me to explain. Okay, see me at a coffee shop outside. I’m wearing this gorgeous burgundy camisole, and my nipples look great. You took the AirTag that we all chipped in on for your house rule, and you put it on a random guy that you profiled as gay. Or bi. No, he was definitely gay. But yes. Okay. I have to work. No, no, Anton, please sit down. Please. We’ve been working on this all morning. Just five minutes. Thank you. So, now… Anton, full commit sit. Okay, look at me. Now look at Paul Baker. Now tell him he’s cute. You’re cute. Okay. Look at that dot. We saw that dot. We heard that dot’s voice. We saw that dot’s bicep ripple in the sun as he drank his dirty, little, slutty, little chai. So slutty. I am telling you, with full confidence, this dot could be more than just your first fuck in eight months. This dot could be the love of your life. The love of your life. I’ve always wanted to read her. Take it. For real, take it. You’ll love Elena Ferrante. You don’t have to walk me out. Please. It’s the least I can do. Hey, sorry about the kids. Yeah. They’re real shits this year. They really are. Well, come back next year. All the COVIDy ones are graduating, so… No, thanks. But this is kind of my farewell tour. Oh. Yeah. And not in the Elton way, where I keep coming back. But in like a Bowie way, where I die at the end. Oh. Oh, oh. Who do we have here? The myth, the legend. Billie? You okay, Billie? Billie, you okay? Yeah. Oh, gosh. What can I… Would you like some water or something? No. I’ll take it down. No, don’t take it down. Okay, I’ll leave it there. Um… Do you…  Do you want some gossip? Teacher gossip? Madame Daubreu’s been embezzling from the French Club. But then how will they go to Quebec? Um… Do you want to help me grade some essays? Yeah, I think I’d like that. Great. Ooh. Dot’s leaving The Whitney. And you know he was there for the art too. Like, that was way too long for just a status tote. Oh, my God. Wow, Dot’s flying down 9th Avenue. Wow, Dot’s in incredible shape. Anton. Okay, can you guys do this in truly any other part of the house? I have to go off mute in a second and say, “Uh-huh,” or else I’ll get an email. Yeah, I’m gonna go prep for my interview upstairs. It’s the one for fintech. “Fin” stands for “financial,” just so we’re all aware. Totally. See? Some people have actual work, instead of wasting the day stalking a gay stranger. Dot’s at Cafe Corinne! Is that good or bad? That’s amazing. That’s, like, Anton’s favorite restaurant. It’s, like, dim lighting, long communal sinks, amazing soap. Good luck, Dot. You’re gonna have to wait in line for, like, an hour. Oh, my God. Dot’s getting in. He must have a connection. …to Anton. Anton? Uh-huh. Great. And we’ll get that… Who is this perfect man? Yeah, so obviously it’s a new space for me, fintech. But I am a quick learner and I’ve been brushing up on Python since college. Okay, very nice. So, yeah, it’ll be a jump, but, like, Harrison Ford started as a carpenter, right? Yeah. That’s good. Well, look, Samir, we’re building this team out fast, and I think I speak for all of us when I say we love you in the role as a PM. Oh. So, uh, yeah, let’s do it. Wha… Yeah? Yes. Like, that… that’s it? That’s it. You got the job, Samir. Good job. Wow. Yeah, that’s great. Well… No, that’s great. Wow, wow. Oh, come on. That’s it? Thank you, guys. We just said you got the job. You can celebrate. Come on. The interview is over. Loosen up. The job is yours. Okay. Yeah. Yeah. I mean, yeah. Come on. It’s exciting. Celebrate a little. Ow! Oh, man. Yeah, there we go. Good job. Ey. Uh-oh. Watch out. Ey. This guy. Ah! Okay. Think I might have you guys on mute. So how does it work, uh, with vacation days? Okay, wait, I think this is AI. Yes. I mean, they’re all AI, but just… How good is the AI that I’m grading? So, any chance you wanna tell me what’s going on with you? I just lost my job, so… Fuck, Billie, I’m sorry. I did have a hunch. Well, you know, what with all the… the knock, knocks. That’s… I know, I… I mean, I’ve been trying, you know? Pretending like it doesn’t bother me and trying to lean into it and telling myself that it’s, like, this great opportunity to rebrand but… It’s not, you know? Like, I’m 24, I’m unemployed, I’m about to be priced out of blonde, and now, on top of that, I’m in 15 grand of medical debt because of some hospital bill. Thank you. Yeah. And it’s like, I don’t know, maybe that’s just what happens. Maybe that’s what life is. You go from being all gold stars and potential to just some bitch who peaked in high school. Look, hey, I’ve seen “peaked in high school,” all right? Max Rapaport. He’s so bald now. I know. I’m not worrying about you. Whatever you’re going through, you will get through. You will. Yeah, I hope so. You will. You just have to, you know, take it one step at a time. Okay, like, for example, what did the hospital say when you called them about the bill? You can call them? Oh. I feel like Dot’s an uncle. Mmm! But, like, he’s not annoying about it. He just, like, loves his nephews. Yeah. And I feel like Dot has, like, a toolbox, but all the tools are actually, like, clicked in their place. Oh. 100%. Yeah. Okay, question for the group. Do we think Dot has a dick or a cock? Oh. Interesting. ‘Cause some guys have dicks, and then some guys have, like, cocks. I feel like Dot has a schlong. You know? Like, it’s, like, heavy. Yeah. Yeah. I feel like… Mindwipe? Yeah. I feel like it’s been hard to break my dry spell or whatever because I, like, hate a random hookup. Mm-hmm. But obviously if it’s not random, then it feels like it means something, and then if it, like, means something, then I could get Jacob again. Anton’s ex. Broke Anton’s heart. Dead to us. Amazing shoulders though. Iss. Sorry. I can’t avoid it forever, obviously. I just… Yeah. Okay, you guys, I called mindwipe. Oh. Totally, yeah. Oh. Sorry. Sorry. Oh, my God! Samir, how’d it go? Uh, I got the job. Hey! What? And then I mooned them. Oh. Then I lost the job. What? So… Yeah. So…  Mooned them. Maybe I’m a socialist. Ah. No, Samy, dude, you gotta… You can’t let all that with the… with the haters, didn’t get the… Oh, my God. Dot’s home. Dot’s home. What? Google that building. East 79th, baby, East 79th. Oh, it’s a doorman building! Oh, my God! I’m gonna be sick. The start of my life has begun, my love. This is the start of my life! So I have the itemized bill in front of me and I did get, um, yeah, all three of those scopes done. But you’re looking at the procedure calculator online. And, um, I’m on the hospital website and the procedure calculator is showing a much lower price. Yeah, I’m looking at 1,700 for an endoscopy. And a thousand for a colonoscopy. And more like a thousand for a colonoscopy. Now ask about the hardship payment plan. Yeah, and let me ask you something, Cindy. Do you have a, um… a hardship payment plan? Mm-hmm. Because I, uh… I just got fired, so… Wait, really? 2,200. Take it, take it. And that’s over six months? Yes. Yeah. Oh, my God. Yeah, I can… I can figure that out, I think. Yes, thank you, Cindy. Thank you so much. Okay, bye. Whoo! Holy shit! Oh my God! Hey. Hey, you were amazing. You were amazing. Oh, my God! Okay, 2,200 is still a lot of money. We’ll figure that out. But, oh, my God, how did you figure out how to do that? There’s a lot of years between 24 and 42. Plus I once paid ten grand for an MRI, so… Hey, wait till I get my hands on your rsum. Oh. Oh. Uh… I don’t know that we should do this in here, Billie. No, no, no, leave them, leave them. I’ll give them all B’s. Wait, I feel insane right now. If Dot’s job transfers him to London, you are not going with him. I’m just saying, maybe I do go with him. And I’m just saying, where the fuck do I live? Well, we can’t have a dedicated room for you in the flat. That’s something we cannot do. I’m sorry. Okay, then find a bigger house. I don’t understand. Finder’s fee. I got you guys together. Finder’s fee? Yeah. How much do I owe you? I’ll pay you. Hey, man. Uh, ID? Oh, yeah, come on in. I need time with him alone. Dot wants me alone. What are you watching? This, uh, squirrel is water skiing. Hey, what’s the interview like for this? For what? For doing deliveries? Yeah. You just sign up. Oh, my God. If anything, I can move in as godmother after you guys have kids. I mean, that’ll be soon. I’m just… Okay. Oh, shit. What? Dot’s in Queens. That’s Harold’s pub. Oh, my God. That’s like two minutes away. What? What? Go to him. Come on, Anton. This is destiny. Anton! This is a once-in-alifetime opportunity. This is the love of your life. Okay, okay, okay, okay, okay. I’ll go. I’ll go. I’ll go. Yeah? I’ll go. Yes, I’m going. I’m go… Okay. I’m gonna get a coat. I’m going. You’re gonna meet your person. Oh, my God. What is happening? So I obviously love you in a structured shoulder, but this one makes you look like Casablanca. Never seen it. Yeah, me neither. Okay, that one. Okay. Oh, my God. Oh, my goodness. Okay. I’m gonna go meet Dot. You’re gonna meet Dot. I’m gonna be face to face with Dot. With the love of your life. Oh my God. Deep breath, deep breath. Hey, go fuck his brains out, crazy girl. There he goes.  We did it. Shit. Again with this fucking cruise. That picture’s good. You think so? That’s really good. That’s a guy you trust with food. Totally. He’s not gonna have a fry, right? What’s going on? Samir got a job. He’s employed! Hello! Fintech? No, I-I mooned them. It was crazy, but I am officially… Ba-ba-ba. “Terms and conditions.” Bada bing, bada boom. I’m doing deliveries. Whoo! Yay! Attaboy. You did it. Did it. Aw. We’re so proud of you, Samy. Oh, my God. Yay! First one! First one! First one! Whoo! Okay. Yeah. Nice. Wait, so what did you… Wait, what’s in your hair? You have something in your… Like… Oh. Yeah. You have something. What? Nothing.

## Deciding on TV shows and Iteration

To compile a list of TV shows I would want to include, I compiled a list
of the available TV shows that would be relevant to the study. Their
relevance was determined using criteria in the [Project Plan
document](project_plan.md). I also included the genre of each show
(derived from the Wikipedia site of the TV show) which allowed me to
ensure a nearly even number of drama TV shows and comedy TV shows. Since
there were only 27 comedy TV shows, I included all of them. I then
randomly selected 33 of the remaining drama TV shows with the following
code.

``` r
set.seed(20)
#just chose a random seed so that we get the same random result every time we run this
dramas <- tv_show_df |>
  filter(str_detect(genre, "drama") & !str_detect(genre, "comedy")) |>
  #filters only shows that included the genre drama and not the genre comedy
  slice_sample(n = 32)
  #randomly selects 32 of these

dramas
```

Finally a compiled a list of the actual html links for each episode.
Unfortunately, since each link referred to a specific episode and there
was no clear way to know which episodes were included, this process had
to be done manually. Once this list of links was compiled, I was able to
iterate using the following mapping code.

``` r
x <- episode_links |>
  map(get_clean_script) 

y <- do.call(rbind, x)

y
```

## Overview of the Data

The resulting data frame is made up of the sixty observations of
individual scraped webpages The TV show title is listed under the `id`
column. In the `clean_wrds` column, the clean text of each page is
included. Importantly, these are divided up roughly evenly my genre. I
intend to add the genre information to the ID and to get a summary of
how many tokens are in each observation as immediate next steps.
