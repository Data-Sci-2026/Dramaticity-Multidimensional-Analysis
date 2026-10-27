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

show_df
```

                                                                                                                                                        show_script
    1                                                            You’re flossing to me? Are you flossing to me? I’m the only one here. Ah. [groans] Okay. Guys, no.
    2                                                                                                                                               [blows whistle]
    3                                                                                                                                                  House rules!
    4                                                                                                                                                [all laughing]
    5                                                                                                              What are we doing? It’s a tradition, you’ll see.
    6                                                                                                                                       Upstairs now! Let’s go!
    7                                                                                                                                Oh, my God. Thank you so much.
    8                                                                                                                               ♪ Never around the same about ♪
    9                                                                                                                                          ♪ She’s got it all ♪
    10                                                                                                                                Strong board this year, guys.
    11                                                                                                                                               Yeah. Awesome.
    12                                                                                                                                      [Paul] Okay, we mix it.
    13                                                                                                                                                 [Issa] Okay.
    14                                                                                                                                        Homeowner goes first.
    15                                                                                                                                                 [Issa gasps]
    16                                                                                                                                        [Samir clears throat]
    17                                                                                                                                            Paul Baker. Yeah.
    18                                                                                                      Your first rule. Can’t believe I have to say this, but…
    19                                                                                                                                     No fucking on the couch.
    20                                                                                                                     Yes, but no. Okay, thematically similar.
    21                                                                                                                          Stop leaving condoms in the toilet.
    22                                                                                                                                                   Copy that.
    23                                                                                                                                              And the shower.
    24                                                                                                                                         [Paul] That’s on me.
    25                                                                                                                               And in the sink that one time.
    26                                                                                                                                             My bad, big bro.
    27                                                                                                            See, I told you we have to stop using protection.
    28                                                                                                                                                          No.
    29                                                                                                                                                        What?
    30                                                                                                                                                    So ruled?
    31                                                                                                                                                    So ruled.
    32                                                                                                                                                    So ruled.
    33                                    And then you drink from the Springsteen on Broadway mug, which is the most sacred drinking vessel in the house. Drink up.
    34                                                                                                                                     [Issa] Go! Go! Go! Whoo!
    35                                                                                                                                           Oh. It’s bad, bad.
    36                                                                                                                                            Okay, who’s next?
    37                                                                                                                                                  Issa. Issa.
    38                                                                                                                                                          Me?
    39                                                                                      Lots of options here. Um… Opening it up to the floor under Article 3-6.
    40                                                                                                             Oh, I know. She’s got to stop saying the F slur.
    41                                                                                                                                     I’m allowed to say that.
    42                                                                                                                                  No, I grandfathered her in.
    43                                                                                                            It’s fine. She has to admit her parents are rich.
    44                                                                                                                        They’re upper middle class. It’s not…
    45                                                                                                                              I cannot have this fight again.
    46                                                                                                                           Then, she has to let us track her.
    47                                                                                              Oh. Yeah, guys, please. I keep losing her in multi-floor Zaras.
    48                                                                        Okay, totally. You guys can track me right now. I just have… Wait, where is my phone?
    49                                                                                                                   Okay, so we will track her with an AirTag.
    50                                                                                                                                                  Wait, what?
    51                                                                                                                                                    So ruled?
    52                                                                                                                                              [all] So ruled.
    53                                                                                                                                                   Bruce her.
    54                                                                                                                                            [Anton] Attagirl.
    55                                                                                                                                            [Samir] Drink up.
    56                                                                                                                                       Anton. Yes. Okay, yes.
    57                                                                                                                    You have to break your dry spell. [gasps]
    58                                                                                                                                                    Fuck off.
    59                                                                                                                       Because you’re getting kind of snippy.
    60                                                                                                                                              Fuck off, girl.
    61                                                                                                                 And also your standards are impossibly high.
    62                                                  Okay, smart, kind, so gay that it turns a corner and becomes straight again. What is the problem with that?
    63                                                                                                                                                [Issa] Anton…
    64                                                                                                                                                    So ruled?
    65                                                                                                                                                    So ruled.
    66                                                                                                                                             Fine. I’m horny.
    67                                                                                                                              [Issa] Paul Baker! Okay, Samir…
    68                                                                   I know, I know. We don’t have to do the whole thing for me, ’cause I already know I gotta…
    69                                                                                                                                             [all] Get a job!
    70                                                                                                                                     I gotta get a job, yeah.
    71                                                                                                                                               [all clapping]
    72                                                                                                                                                      Billie.
    73                                                                                                                                                         Yes.
    74                                                                                                                 You must stop visiting your old high school.
    75                                                                                                                                           Give me the gavel.
    76                                                                                                                                   No. No. No more check-ins.
    77                                                                                                No more knock, knocks. I mean, what even is a “knock, knock”?
    78                                                                                                                          Knock, knock. Just in the area. Hi.
    79                                                                                                                                          It’s strange, love.
    80                                                                                                                                           [Samir] Thank you.
    81                                                                                                                    If I stop going, they’ll think I’ve died.
    82                                                                                                                                 That’s okay. Okay, so ruled?
    83                                                                                                                                              [all] So ruled.
    84                                                                                                                                        [Samir] Starting now.
    85                                                                                                                                         You got this, Bills.
    86                                                                                                                              No more knock, knocks. No more…
    87                                                                                                                                                Knock, knock.
    88                                                                                                                 Hey, Billie Schaeffer. How the hell are you?
    89                                                                                             Yeah, good. Just in the neighborhood, thought I’d, uh, swing by.
    90                                                                                                        Yeah, please, come on in. Oh, my God. I’d be honored.
    91                                                                                                                                          Hey, guys, this is…
    92                                                                                                                          Hey, Kelsey. [whistles] Headphones.
    93                                                                                                                                        Guys, this is Billie.
    94                                                                                             Billie is, no joke, one of Garfield’s best and brightest alumni.
    95                                                                                                                               We had these desks. [chuckles]
    96                                       In fact, actually, Billie here was editor in chief of the paper for three years, was it? Yeah, wow. Three years, yeah.
    97                                                                                                          Three years. God, best years of my life. [chuckles]
    98                                                                                                              So, she pretty much taught me how to do my job.
    99                                                                                                                           Oh, my God. Stop. How’ve you been?
    100                                                                                               How’s-how’s, uh, the big, bad world? Everything is great. Um…
    101                                                                                                                      Everything is going according to plan.
    102                                                                                                                                    So… Great. Good for you.
    103                                                                                                                          I mean, I was gonna head out, but…
    104                                                                                                             I mean, if you guys have any questions, I mean…
    105                                                                                                      No, not if you’re on your way out. You go. We’re fine.
    106                                                                                                                                    I mean, I have a minute.
    107                                                                                                                                Okay. I have a minute. Yeah.
    108                                                                                   Okay, okay. Yeah, that’d be great. We would love it. We’ll take it. Guys.
    109                                                                                                                                 Hit me with your best shot.
    110                                                                                                                                        We’re on a deadline.
    111                                                                                                                    Ooh. Kelsey, that’s a little rude, okay?
    112                                                                                                 So we’re gonna call that strike one for the day, all right?
    113                                                                                                                                                 Yeah, Josh.
    114                                                                                            Hi. Josh Rapaport. Do you have any tips for interviewing people?
    115                                                                                                                                       Do I have tips? Yeah.
    116                                                                                                                                          Yeah, I have tips.
    117                                                                                                              So, sophomore year, I taught myself shorthand.
    118                                                                                                            That’s right. Google “Hot to Jot” Cheryl Koenig.
    119                                                                                                                                    Game changer. [chuckles]
    120                                                                                                                                   Yes, over here. Question?
    121                                                                                                          Um, didn’t you stop by my Spanish class last week?
    122                                                                                                Yeah. Yeah, you were there last week. I remember seeing you.
    123                                                                                                               No, I don’t think I did. Did I? I don’t know.
    124                                                                                                                 Yeah, you asked if we were up to vosotros .
    125                                                                                                                               Don’t think that that was me.
    126                                                                                                                        Do we have any other questions? Yes.
    127                                                                                                              Yeah, why do you care if we’re up t vosotros ?
    128                                                                                                 No, you know what? Why don’t we keep it on track, shall we?
    129                                                                                                               Avery is right. She’s here a lot. It’s weird.
    130                                                                                                           Okay, so that feels like strike two. [sighs] Why?
    131                                                                                                         How is Rachel Sheffield always on a fucking cruise?
    132                                                                                                                                         I got an interview.
    133                                                                                   What? I got an interview at, like, a… at a company. No, I can’t hear you.
    134                                                                                                                           I have an interview at a company.
    135                                                                                                                                     Like a fintech company.
    136                                                                                                                                                 Wait, what?
    137                                                                                                                                             That’s amazing.
    138                                                                                                                                I know. Picture me, fintech.
    139                                                                                                                      Can you see it? I’m at a desk. Mm-hmm.
    140                                                                                                                   Bunch of staplers, little ones, big ones.
    141                                                                                                                            Staple remover as well. ‘Course.
    142                                                                                                                          I’m sitting there, “Hi, I’m Samir.
    143                                                                                                                  I work at fintech.” My voice is different.
    144                                                                                                            It’s “in fintech.” But yes, I fully pictured it.
    145                                                                                                                           Wow. [stammers] It’s great. Okay.
    146                                                                                                                                              This is crazy.
    147                                                                                                                  I just gotta look up what the “fin” means.
    148                                                                                                                                       Mm-hmm. Look that up.
    149                                                                           So now it’s between Abby P and Abby F for the lead of Popcorn Forest the Musical.
    150                                                                                                     But it’s just, like, Abby F’s parents are in a commune.
    151                                                                                                                                   She isn’t well. And done.
    152                                                                                                                    You are officially airtagged. I love it.
    153                                                                                            Wait. It’s actually so chic. Should we take it for a test-drive?
    154                                                                                                                                [gasps] Yeah. Go to Sephora.
    155                                                                                                                                Somewhere scary. Oh, my God.
    156                                                                                                                         I love how you’re speaking. Uh-huh.
    157                                                                                                                      That sounds so fun. I love it. [gasps]
    158                                                                                                                     What? Do we know him? Oh, my God. Look.
    159                                                                                                               No, but that’s, like, literally Anton’s type.
    160                                                                                                                              Well, he’s with a girl though.
    161                                                                                                              Well, I started to, like, notice people again.
    162                                                  And it’s the first time since Michael that I feel like I’m ready to get back out there. Michael. He’s gay.
    163                                                                                                                                        He’s gay. [chuckles]
    164                                                                                                  And this is why I’m like, “Everyone should be in therapy.”
    165                                                                                                                       [friend] I know. Wait, are you still…
    166                                                                                                                                           What does he say?
    167                                                                                                             I can’t. Tomorrow, I’m doing that park cleanup.
    168                                                                                                                         [friend] All right. Yeah, you know.
    169                                                                                                                              Hey, you want to walk with me?
    170                                                                                                                                              [friend] Yeah.
    171                                                                                                             Oh, shit, they’re leaving. Wait, what do we do?
    172                                                                                                                                            No, babe, don’t…
    173                                                                                                                                        Don’t. Don’t! Don’t!
    174                                                                                                                                  No, you did not. I had to.
    175                                                                                                                          You did not have to. I really did.
    176                                                                                                          We are in a post-de Blasio, pre Avatar III moment.
    177                                                                                                                                            We have to live.
    178                                                                                                        What does that have to do with this? No, look at me.
    179                                                                                                                                            We have to live.
    180                                                                                                                              That’s, like, a hundred bucks.
    181                                                                                                                     Okay, the mailbox is so full, you guys.
    182                                                                                                          We actually have to think about opening that soon.
    183                                                                                                                     He’s here! Welcome. We’ve been waiting.
    184                                                                                                      Whatever it is, no. I have work. No, no, no, so do we.
    185                                                                                                     We have been working to break your dry spell, actually.
    186                                                                                                                                        Allow me to explain.
    187                                                                                                                      Okay, see me at a coffee shop outside.
    188                                                                                     I’m wearing this gorgeous burgundy camisole, and my nipples look great.
    189                                 You took the AirTag that we all chipped in on for your house rule, and you put it on a random guy that you profiled as gay.
    190                                                                                                                           Or bi. No, he was definitely gay.
    191                                                                                                                              But yes. Okay. I have to work.
    192                                                                                                             No, no, Anton, please sit down. Please. [sighs]
    193                                                                                                  We’ve been working on this all morning. Just five minutes.
    194                                                                                                                 Thank you. So, now… Anton, full commit sit.
    195                                                                                                                                           Okay, look at me.
    196                                                                                                             Now look at Paul Baker. Now tell him he’s cute.
    197                                                                                                                                          You’re cute. Okay.
    198                                                                                                                                           Look at that dot.
    199                                                                                                                 We saw that dot. We heard that dot’s voice.
    200                                                               We saw that dot’s bicep ripple in the sun as he drank his dirty, little, slutty, little chai.
    201                                                                                                                                       [whispers] So slutty.
    202                                                   I am telling you, with full confidence, this dot could be more than just your first fuck in eight months.
    203                                                                                                                    This dot could be the love of your life.
    204                                                                                                                                      The love of your life.
    205                                                                                                                                                    [scoffs]
    206                                                                                                 I’ve always wanted to read her. Take it. For real, take it.
    207                                                                                                                                 You’ll love Elena Ferrante.
    208                                                                                             You don’t have to walk me out. Please. It’s the least I can do.
    209                                                                                                                            Hey, sorry about the kids. Yeah.
    210                                                                                                              They’re real shits this year. They really are.
    211                                                                                          Well, come back next year. All the COVIDy ones are graduating, so…
    212                                                                                                           No, thanks. But this is kind of my farewell tour.
    213                                                                                                                                                   Oh. Yeah.
    214                                                                                                         And not in the Elton way, where I keep coming back.
    215                                                                                                            But in like a Bowie way, where I die at the end.
    216                                                                                                                 Oh. Oh, oh. Who do we have here? [chuckles]
    217                                                                                                                                       The myth, the legend.
    218                                                                                                                                         [♪ wistful music ♪]
    219                                                                                                                                   Billie? You okay, Billie?
    220                                                                                                                                           Billie, you okay?
    221                                                                                                                                 Yeah. Oh, gosh. What can I…
    222                                                                                                                 Would you like some water or something? No.
    223                                                                                                                  I’ll take it down. No, don’t take it down.
    224                                                                                                                              Okay, I’ll leave it there. Um…
    225                                                                                                    Do you… [sighs] Do you want some gossip? Teacher gossip?
    226                                                                                                      Madame Daubreu’s been embezzling from the French Club.
    227                                                                                                                 [sobs] But then how will they go to Quebec?
    228                                                                                                                                                  [sobs] Um…
    229                                                                                                                   Do you want to help me grade some essays?
    230                                                                                                                                Yeah, I think I’d like that.
    231                                                                                                                                                      Great.
    232                                                                                                                             Ooh. Dot’s leaving The Whitney.
    233                                                                                                           [Issa] And you know he was there for the art too.
    234                                                                                                         Like, that was way too long for just a status tote.
    235                                                                                                                                                 Oh, my God.
    236                                                                                                                          Wow, Dot’s flying down 9th Avenue.
    237                                                                                                                      Wow, Dot’s in incredible shape. Anton.
    238                                                                                            Okay, can you guys do this in truly any other part of the house?
    239                                                                             I have to go off mute in a second and say, “Uh-huh,” or else I’ll get an email.
    240                                                                                Yeah, I’m gonna go prep for my interview upstairs. It’s the one for fintech.
    241                                                                                                      “Fin” stands for “financial,” just so we’re all aware.
    242                                                                                                                                               Totally. See?
    243                                                                           Some people have actual work, instead of wasting the day stalking a gay stranger.
    244                                                                                                                               [Issa] Dot’s at Cafe Corinne!
    245                                                                                                                                 [Paul] Is that good or bad?
    246                                                                                                  That’s amazing. That’s, like, Anton’s favorite restaurant.
    247                                                                                                It’s, like, dim lighting, long communal sinks, amazing soap.
    248                                                                                                                                [Paul gasps] Good luck, Dot.
    249                                                                                                       You’re gonna have to wait in line for, like, an hour.
    250                                                                                             [Paul] Oh, my God. Dot’s getting in. He must have a connection.
    251                                                                                                                       [Anton’s colleague] …to Anton. Anton?
    252                                                                                                                                                     Uh-huh.
    253                                                                                                              [Anton’s colleague] Great. And we’ll get that…
    254                                                                                                                                    Who is this perfect man?
    255                                                                                                                                 [♪ whimsical dance music ♪]
    256                                                                                                        Yeah, so obviously it’s a new space for me, fintech.
    257                                                                                 But I am a quick learner and I’ve been brushing up on Python since college.
    258                                                                                                                                            Okay, very nice.
    259                                                                          So, yeah, it’ll be a jump, but, like, Harrison Ford started as a carpenter, right?
    260                                                                                                                                            [chuckles] Yeah.
    261                                                                                                                                         [Adam] That’s good.
    262                         Well, look, Samir, we’re building this team out fast, and I think I speak for all of us when I say we love you in the role as a PM.
    263                                                                                                                                                         Oh.
    264                                                                                                                                  So, uh, yeah, let’s do it.
    265                                                                                                                                                  Wha… Yeah?
    266                                                                                                                                                 [Adam] Yes.
    267                                                                                                                                      Like, that… that’s it?
    268                                                                                                                          That’s it. You got the job, Samir.
    269                                                                                                                                              Good job. Wow.
    270                                                                                                                 Yeah, that’s great. Well… No, that’s great.
    271                                                                                                                                                   Wow, wow.
    272                                                                                                                                     Oh, come on. That’s it?
    273                                                                                                                                 [stammers] Thank you, guys.
    274                                                                                                                               We just said you got the job.
    275                                                                                                                                [Lindsay] You can celebrate.
    276                                                                                                                                                    Come on.
    277                                                                                                                                      The interview is over.
    278                                                                                                    [Lindsay] Loosen up. The job is yours. Okay. Yeah. Yeah.
    279                                                                                                                                               I mean, yeah.
    280                                                                                                                              [Adam] Come on. It’s exciting.
    281                                                                                                                  [Lindsay] Celebrate a little. Ow! Oh, man.
    282                                                                                                                                Yeah, there we go. Good job.
    283                                                                                                                                                         Ey.
    284                                                                                                                                           Uh-oh. Watch out.
    285                                                                                                                                    Ey. This guy. [whooping]
    286                                                                                                                                    [gasping] Ah! [chuckles]
    287                                                                                                                  Okay. Think I might have you guys on mute.
    288                                                                                                                So how does it work, uh, with vacation days?
    289                                                                                                                             Okay, wait, I think this is AI.
    290                                                                                                                      Yes. I mean, they’re all AI, but just…
    291                                                                                                             How good is the AI that I’m grading? [chuckles]
    292                                                                                                  So, any chance you wanna tell me what’s going on with you?
    293                                                                                                                             I just lost my job, so… [sighs]
    294                                                                                                                                    Fuck, Billie, I’m sorry.
    295                                                                                                                                         I did have a hunch.
    296                                                                                               [sighs] Well, you know, what with all the… the knock, knocks.
    297                                                                                              [sighs] That’s… I know, I… I mean, I’ve been trying, you know?
    298                  Pretending like it doesn’t bother me and trying to lean into it and telling myself that it’s, like, this great opportunity to rebrand but…
    299                                                                                                                      [clicking stapler] It’s not, you know?
    300 Like, I’m 24, I’m unemployed, I’m about to be priced out of blonde, and now, on top of that, I’m in 15 grand of medical debt because of some hospital bill.
    301                                                                                                                                            Thank you. Yeah.
    302                                                                                                And it’s like, I don’t know, maybe that’s just what happens.
    303                                                                                                                                  Maybe that’s what life is.
    304                                                                You go from being all gold stars and potential to just some bitch who peaked in high school.
    305                                                                                                    Look, hey, I’ve seen “peaked in high school,” all right?
    306                                                                                                                                               Max Rapaport.
    307                                                                                                                                   He’s so bald now. I know.
    308                                                                                                                                            [both chuckling]
    309                                                                                                                                 I’m not worrying about you.
    310                                                                                                        Whatever you’re going through, you will get through.
    311                                                                                                                                                   You will.
    312                                                                                                                                            Yeah, I hope so.
    313                                                                                                                                                   You will.
    314                                                                                                     You just have to, you know, take it one step at a time.
    315                                                                     Okay, like, for example, what did the hospital say when you called them about the bill?
    316                                                                                                                                      You can call them? Oh.
    317                                                                                                                         [Anton] I feel like Dot’s an uncle.
    318                                                                                                                                                 [Issa] Mmm!
    319                                                                                    But, like, he’s not annoying about it. He just, like, loves his nephews.
    320                                               Yeah. And I feel like Dot has, like, a toolbox, but all the tools are actually, like, clicked in their place.
    321                                                                                                                                [Issa gasps] Oh. 100%. Yeah.
    322                                                                                         Okay, question for the group. Do we think Dot has a dick or a cock?
    323                                                                                                                                            Oh. Interesting.
    324                                                                                          ‘Cause some guys have dicks, and then some guys have, like, cocks.
    325                                                                                                                              I feel like Dot has a schlong.
    326                                                                                                                          You know? Like, it’s, like, heavy.
    327                                                                                                                                          [Issa] Yeah. Yeah.
    328                                                                                                                                I feel like… Mindwipe? Yeah.
    329                                                 I feel like it’s been hard to break my dry spell or whatever because I, like, hate a random hookup. Mm-hmm.
    330               But obviously if it’s not random, then it feels like it means something, and then if it, like, means something, then I could get Jacob again.
    331                                                                                                                Anton’s ex. Broke Anton’s heart. Dead to us.
    332                                                                                                                                   Amazing shoulders though.
    333                                                                                                                                      [whispers] Iss. Sorry.
    334                                                                                                                        I can’t avoid it forever, obviously.
    335                                                                                                                                               I just… Yeah.
    336                                                                                                                          Okay, you guys, I called mindwipe.
    337                                                                                                                        Oh. Totally, yeah. Oh. Sorry. Sorry.
    338                                                                                                                             Oh, my God! Samir, how’d it go?
    339                                                                                                                                          Uh, I got the job.
    340                                                                                                                                                  Hey! What?
    341                                                                                                                                 And then I mooned them. Oh.
    342                                                                                                                                        Then I lost the job.
    343                                                                                                                                                       What?
    344                                                                                                                                                 [Samir] So…
    345                                                                                                                                                       Yeah.
    346                                                                                                                                   So… [mouths] Mooned them.
    347                                                                                                                                      Maybe I’m a socialist.
    348                                                                                                                              Ah. No, Samy, dude, you gotta…
    349                                                                                           You can’t let all that with the… with the haters, didn’t get the…
    350                                                                                             Oh, my God. Dot’s home. Dot’s home. What? Google that building.
    351                                                                                                    East 79th, baby, East 79th. Oh, it’s a doorman building!
    352                                                                                                                              Oh, my God! I’m gonna be sick.
    353                                                                                      The start of my life has begun, my love. This is the start of my life!
    354                                                                                                                                               [rat squeaks]
    355                                                         So I have the itemized bill in front of me and I did get, um, yeah, all three of those scopes done.
    356                                                                                                      But you’re looking at the procedure calculator online.
    357                                                            And, um, I’m on the hospital website and the procedure calculator is showing a much lower price.
    358                                                                                                                Yeah, I’m looking at 1,700 for an endoscopy.
    359                                                                                                                           And a thousand for a colonoscopy.
    360                                                                                                                 And more like a thousand for a colonoscopy.
    361                                                                                                                    Now ask about the hardship payment plan.
    362                                                                                                                  Yeah, and let me ask you something, Cindy.
    363                                                                                                                 Do you have a, um… a hardship payment plan?
    364                                                                                                                                      Mm-hmm. Because I, uh…
    365                                                                                                                                       I just got fired, so…
    366                                                                                                                                               Wait, really?
    367                                                                                                                         [whispers] 2,200. Take it, take it.
    368                                                                                                                       And that’s over six months? [gasping]
    369                                                                                                                         Yes. Yeah. Oh, my God. Yeah, I can…
    370                                                                                                                             I can figure that out, I think.
    371                                                                                                                                      Yes, thank you, Cindy.
    372                                                                                                                               Thank you so much. Okay, bye.
    373                                                                                                                                                       Whoo!
    374                                                                                                                                  Holy shit! Oh my God! Hey.
    375                                                                                                                    Hey, you were amazing. You were amazing.
    376                                                                                     Oh, my God! Okay, 2,200 is still a lot of money. We’ll figure that out.
    377                                                                                                     But, oh, my God, how did you figure out how to do that?
    378                                                                                                                   There’s a lot of years between 24 and 42.
    379                                                                                                                  Plus I once paid ten grand for an MRI, so…
    380                                                                                                                                                     [sighs]
    381                                                                                                                 Hey, wait till I get my hands on your rsum.
    382                                                                                                                                     Oh. Oh. Uh… [unzipping]
    383                                                                                                        I don’t know that we should do this in here, Billie.
    384                                                                                                                                           [whimpers, pants]
    385                                                                                                 No, no, no, leave them, leave them. I’ll give them all B’s.
    386                                                                                                                       [Issa] Wait, I feel insane right now.
    387                                                                                           If Dot’s job transfers him to London, you are not going with him.
    388                                                                                                                    I’m just saying, maybe I do go with him.
    389                                                                                                                                            [doorbell rings]
    390                                                                                                              And I’m just saying, where the fuck do I live?
    391                                                                                                   Well, we can’t have a dedicated room for you in the flat.
    392                                                                                   That’s something we cannot do. I’m sorry. Okay, then find a bigger house.
    393                                                                                                  I don’t understand. Finder’s fee. I got you guys together.
    394                                                                                                                                         Finder’s fee? Yeah.
    395                                                                                                                        How much do I owe you? I’ll pay you.
    396                                                                                                                     Hey, man. Uh, ID? Oh, yeah, come on in.
    397                                                                                                     [Anton] I need time with him alone. Dot wants me alone.
    398                                                                                                                                      What are you watching?
    399                                                                                                                         This, uh, squirrel is water skiing.
    400                                                                                                                      [Anton and Issa continue indistinctly]
    401                                                                                                                    Hey, what’s the interview like for this?
    402                                                                                                                       For what? For doing deliveries? Yeah.
    403                                                                                                                                           You just sign up.
    404                                                                                                                                          [Issa] Oh, my God.
    405                                                                                           If anything, I can move in as godmother after you guys have kids.
    406                                                                                                                    I mean, that’ll be soon. I’m just… Okay.
    407                                                                                                                                             Oh, shit. What?
    408                                                                                                                                            Dot’s in Queens.
    409                                                                                                                                                [Issa gasps]
    410                                                                                              That’s Harold’s pub. Oh, my God. That’s like two minutes away.
    411                                                                                                                                                     [gasps]
    412                                                                                                                                      What? What? Go to him.
    413                                                                                                                     Come on, Anton. This is destiny. Anton!
    414                                                                                     This is a once-in-alifetime opportunity. This is the love of your life.
    415                                                                                                    Okay, okay, okay, okay, okay. I’ll go. I’ll go. I’ll go.
    416                                                                                                                      Yeah? I’ll go. Yes, I’m going. I’m go…
    417                                                                                                                      Okay. I’m gonna get a coat. I’m going.
    418                                                                                               You’re gonna meet your person. Oh, my God. What is happening?
    419                                                              So I obviously love you in a structured shoulder, but this one makes you look like Casablanca.
    420                                                                                                                            Never seen it. Yeah, me neither.
    421                                                                                                                                       Okay, that one. Okay.
    422                                                                                            [Issa] Oh, my God. Oh, my goodness. Okay. I’m gonna go meet Dot.
    423                                                                                           [Issa] You’re gonna meet Dot. I’m gonna be face to face with Dot.
    424                                                                                                              With the love of your life. Oh my God. [gasps]
    425                                                                                                                        [whispers] Deep breath, deep breath.
    426                                                                                                                    Hey, go fuck his brains out, crazy girl.
    427                                                                                                                                              [both sighing]
    428                                                                                                                        There he goes. [whispers] We did it.
    429                                                                                                                       ♪ You build me up you break me down ♪
    430                                                                                                        ♪ Then you whisper to yourself “It’s just a phase” ♪
    431                                                                                                                                           ♪ Just a phase… ♪
    432                                                                                                                                   [♪ upbeat rock continues]
    433                                                                                                                                               [music stops]
    434                                                                                                                                 [siren wailing in distance]
    435                                                                                                                                                       Shit.
    436                                                                                                                                                     [sighs]
    437                                                                                                                                              [phone chimes]
    438                                                                                                                                        [message tone beeps]
    439                                                                                                                                        [message tone beeps]
    440                                                                                                                             Again with this fucking cruise.
    441                                                                                                                                 [Issa] That picture’s good.
    442                                                                                                                                       [Samir] You think so?
    443                                                                                                                                  [Issa] That’s really good.
    444                                                                                                                   [Samir] That’s a guy you trust with food.
    445                                                                                                                                             [Issa] Totally.
    446                                                                                                                   [Samir] He’s not gonna have a fry, right?
    447                                                                                                                           What’s going on? Samir got a job.
    448                                                                                                                              He’s employed! Hello! Fintech?
    449                                                                                                     No, I-I mooned them. It was crazy, but I am officially…
    450                                                                                                     Ba-ba-ba. “Terms and conditions.” Bada bing, bada boom.
    451                                                                                                                                 I’m doing deliveries. Whoo!
    452                                                                                                                                         [all cheering] Yay!
    453                                                                                                                                                    Attaboy.
    454                                                                                                                                          [Issa] You did it.
    455                                                                                                                                             [Samir] Did it.
    456                                                                                                                     [Paul] Aw. We’re so proud of you, Samy.
    457                                                                                                                                          [Issa] Oh, my God.
    458                                                                                                                                               [Billie] Yay!
    459                                                                                                                      First one! First one! First one! Whoo!
    460                                                                                                                                                 Okay. Yeah.
    461                                                                                                                                              [all laughing]
    462                                                                                                                                              [Billie] Nice.
    463                                                                                                           Wait, so what did you… Wait, what’s in your hair?
    464                                                                                                                       You have something in your… Like… Oh.
    465                                                                                                                                   Yeah. You have something.
    466                                                                                                                                                       What?
    467                                                                                                                                           [♪ wistful music]
    468                                                                                                                                      [smacks lips] Nothing.
    469                                                                                                                                   ♪ Those schoolgirl days ♪
    470                                                                                                              ♪ Of telling tales and biting nails are gone ♪
    471                                                                                                                                   ♪ If you wanted the sky ♪
    472                                                                                                                 ♪ I would write across the sky in letters ♪
    473                                                                                                                    ♪ That would soar a thousand feet high ♪
    474                                                                                                                                     ♪ “To sir, with love” ♪
    475                                                                                                                                   ♪ If you wanted the sky ♪
    476                                                                                                                 ♪ I would write across the sky in letters ♪
    477                                                                                                                    ♪ That would soar a thousand feet high ♪
    478                                                                                                                                     ♪ “To sir, with love” ♪

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
