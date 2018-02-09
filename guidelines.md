##### Richer Event Description AMR (version 0.1)


This is a very preliminary pilot of annotation of *temporal, causality and modality* information on top of AMR.  

We want to produce a rich representation of temporal structure on top of AMR.   Even more than raw text, AMR elements are lacking  in information about what an "event" is, whether those events happened, or when they happened.  We need to add this information to the AMRs.  So one thing to keep in mind during annotation is that we hope to end up with an annotation where if feels like those core questions -- what are the events, did they happen, and when did they happen -- are answered relatively well. 

Our second goal with this is that our annotations need to be *scalable and accurate*.  Because of this, we will sometimes be annotating things in a rather compressed manner  -- focusing upon moments where your insight as an annotator is essential, and reducing the amount of time you spend on things that should be obvious.  

We assume that you have annotated AMRs before [(see the guidelines here for a refresher.)](https://github.com/amrisi/amr-guidelines/blob/master/amr.md).  You will also often be directed to the [guidelines for RED](https://github.com/timjogorman/RicherEventDescription), which is similar to the annotation we are doing now, but was done over text. 


### Getting Events with DoctimeRel and Modality

#### The "Event Container" trick
We are focusing on an idea called an "Event Container".  All this really means is that we are thinking about how events have subevents, and that those subevents generally *inherit* information from their larger event. 

Imagine you have a document discussing two people going to dinner.  There may be many events there -- there might be an event or ordering a mean, a main course, there might be a situation where they pay the check.  Those events, too, might have subevents, such as fighting over the bill or finishing your meal.  One thing we are capturing is this "subevent" structure -- marking that these events are part of the larger "dinner" event.

The other thing that we are doing with these subevents is that we are using them to make assumptions about how events work.  If we see another event -- such as "scheduling dinner" -- we might want to annotate that that event happened before the dinner.   If we know that all these other events, such as "paying the check", are subevents of that dinner, then we don't need to mark any temporal relations between "scheduling dinner" and "paying the check", because we can infer that "we scheduled dinner before having dinner" and "paying the check was part of dinner".   We will sometimes refer to this "temporal inference" as *temporal closure*.  This means that you will be focusing on marking these "subevent" relations, so that we can mostly infer temporal relations, instead of making you manually label them.  

We will also be assuming that, unless you label things otherwise, these subevents will inherit the *modality* of their containing event as well.  If someone describes an entirely hypothetical dinner, then knowing that the dinner is hypothetical generally implies that the main course of that dinner is also hypothetical.  

#### How this combines with AMR

This approach means that there are many events in RED-AMR where the *only* information we need about them is which container they are part of.    Because of this, the primary things we are annotating are "event container" objects, each of which have a box for a single "main event" and a collection of subevents.  




## A basic event annotation

Let's start with a simple example from the Little Prince.  This will be useful for a number of these discussions.  We have one clear, dynamic event here -- "giving up":

```
That is why , at the age of six , I gave up what might have been a magnificent career as a painter .
(c2 / cause-01
      :ARG0 (t2 / that)
      :ARG1 (g / give-up-07
            :ARG0 (i / i)
            :ARG1 (c / career
                  :mod (m / magnificent)
                  :topic (p / person
                        :ARG0-of (p2 / paint-02)))
            :time (a / age-01
                  :ARG1 i
                  :ARG2 (t / temporal-quantity :quant 6
                        :unit (y / year)))))
```

We will press "e" and create an "EventContainer".  This has a "Event" box, "Subevents" box, "Doctime" menu, and "Modality" menu, and "AspectEtc" menu.  Press 1 (to select the "Events" box) and click on the event "g". You then can change the DocTime (relation beteen this event and the document utterance time) and the Modality, but they are already correct -- it's "Before" and "Actual". 

##### What about states and situations?!

We want to be independent of the raw syntax of the sentence -- this is a meaning representation, after all!  But that means that we will run into lots of states and activities, and don't have the normal luxury of having rules like "only deal with verbs".  So that's the question -- does "at the age of six" count as an eventuality --- a thing to keep track of temporally?  It *has a specific temporal structure* and therefore we want to capture it! 

So pull up a new EventContainer, put "a" in its mention box, but this time, we want to change the "AspectEtc." to be "State". 

You should start getting the feeling of how we are annotating "features" using these containers.  There will be a bunch of shortcuts to make this go faster, which we will cover in a bit. 

##### What about "painter?!"

That's fine!  Ignore habitual words that define a particular entity in terms of what they "tend to" do.  However, you SHOULD label those events if they refer to a particular event.  Even paint-01 in "painter" *would* be a valid mention in a context where it implies a particular event -- as in "we all worked on this car -- Mary fixed the engine, John reappolstered it and I was the painter".  

#### So How are we defining eventualities?

Edge cases that get worse in AMR:

**Events that characterize participants**: These are events only if they refer to specific events.  Simple test -- if you could add additional context that implies that the event never actually happens, then it's definitely not an event.  E.g. "He's a math teacher now, but hasn't successfully taught anyone math yet", or "the new firefighter has yet to fight any fires".  So basically, don't use these relational career predicates as actual "events", but keep an eye out for things like "9/11 first responders" or "Grammy winner Rob Thomas"

**Properties**: We have lots of predicates referring to states! Claiming that things are "good", "sad", "happy", etc.  When are these "events"?  We generally say that they are NOT.  

**Relations and Negations**: Cause-01 is not an event for our purpses!  Also, markers to negation, likelihood, or possibilitiy --"possible-01",  "didn't chose to", "didn't bother with", etc. -- aren't real events.


#### Event Coreference:

Note that some of the events are colored more darkly.  These are EVENT CHAINS.   You don't need to do event  coreference (it's already done), and each "event chain" will act as a single entity -- so you don't need to label it multiple times.  


## Capturing Temporal Containment

Let's say we move on to this:

```
Whenever I met one of them who seemed to me at all clear - sighted , I tried the experiment of showing him my Drawing Number One , which I have always kept .
(t / try-01
      :ARG0 (i / i)
      :ARG1 (e / experiment-01
            :ARG1 (s / show-01
                  :ARG1 (p2 / picture :wiki - :name (n / name :op1 "Drawing" :op2 "Number" :op3 "One"))
                  :ARG2 p
                  :ARG1-of (k / keep-01
                        :ARG0 i
                        :time (a / always))))
      :time (m / meet-02
            :ARG0 i
            :ARG1 (p / person
                  :ARG1-of (i2 / include-91
                        :ARG2 (t3 / they))
                  :ARG0-of (s3 / see-01
                        :ARG1-of (c / clear-08)
                        :ARG1-of (s4 / seem-01
                              :ARG2 i)))
            :mod (a2 / any)))
```


Now we can assume that we have a "meet-02" event, an "experiment-01" event, a "show-01" event, and a "try-01" event. But they're all part of the same event!  And they have the same modality and temporal information.  We'll encode them all in the *same event container*.    Just make one of these containers (press 'e') and select "meet-02" as the Event, and then add the rest as subevents.   In general, we are going to assume *inheritance* of modaly and temporal properties -- that those labels technically only label "meet-02",  but are inherited by default by the other events.

The secondary thing about this trick of "default inheritance" is that you can override it.  If you wanted to make "show" a hypotehtical event, you just make a second event container, select "shiow" as the main event, and give it a hypothetical modality.  That's it!  

## A bunch of shortcuts!

In reality, instead of using "EventContainer", you want to use a bunch of other relations that are *exactly the same* as Event Container, but which have different default arguments.  If you see an event that didn't actually happen, you should be able to label that with three quick motions --- pressing  "n", pressing "1", and clicking on the event you are negating.  


## Causal and Temporal Links

These work just as described in the RED annotation, and are largely self-explanatory. MOST causal and temporal links *don't need to be annotated* because they are already captured by AMR.  Just label the ones that are both informative and NOT captured.  






