---
title: "On how I build with AI: Moving to visual inputs"
date: 2026-11-05
published: true
layout: post
image: /assets/images/from-text-to-visuals-ai-experiment.png
---
My approach to building with AI will see a change from tomorrow. I have already moved from text to dictation. And now, it is going to be visual inputs.

A few months ago, I was typing to convey what I wanted. In the beginning, I wrote well-formed sentences. But then the rate at which results came back was very slow.  I remember that my fingers also used to hurt sometimes, and in those moments I used to write messily. I found that the output didn't depend on whether I wrote good English or bad English. So, I stopped typing well-constructed sentences and changed to typing my thoughts as fast as I could without any regard to punctuation anymore.

But my fingers still hurt! And around that point, I shifted to giving dictation. And this was much better. Now I do it 90% of the time. I speak my raw thoughts out loud. The speed of work has also increased.

But, what happened yesterday was very interesting for me. I got a flash of an idea. Whenever that happens, I try to put it down somewhere while it's fresh; otherwise I cannot recollect it with that level of fidelity.

So, I have a Google Sheet where I write my ideas. And I've often felt that I am unable to clearly recollect the image I originally had in mind by reading the text therein. My Claude Ultra plan was also exhausted else I would have dictated there to give it some initial shape. These two constraints gave me that moment of pause, and I then decided that I will make a video of it or something so that it stays visual. 

I opened, thought of Grok Imagine, then finally visited ChatGPT as they have generous limits. I asked it to generate visuals of my idea. It first fetched images from the internet and relied on text to convey its understanding. Then I asked it to draw. And I am so happy about this approach because in under three turns, the images it drew showed 90% of what I conveyed. And my idea is now imprinted in my mind too.

When I used to dictate, the results appeared to me after the code was written. Since backend code ideas can be abstract, this idea of some sort of visual representation never occurred to me. Building the frontend has been a bit easier because the final visuals can be seen and one can work backwards from them. But yesterday's experience brought me the visuals of final product before it even exists.

Lately, Claude has started showing me flowcharts etc for backend ideas in its artifacts. So now I have decided that backend ideas will be represented in flowcharts etc first before any line of code is written. The frontend ideas will be working prototypes or animations. The whole product will then be composed visually using the above two. It can be images, videos, animations, anything. These will serve as the reference. Only then will any line of code be written. The model will be asked to refer to these visuals for alignment.

In the build-and-see approach, I have often found myself spending time on minor issues that were not taken into account due to the model's own assumptions of unsaid things (refer to Thariq's article on unknowns [https://x.com/trq212/status/2073100352921215386](https://x.com/trq212/status/2073100352921215386)). And when these issues happened after a four hour run of sessions, it became a real pain. So, I now think that if I start with visual inputs, I MAY get accurate results faster. And if that happens, it will unlock something new for me.

We think visually anyway. And writing this piece has brought me a very good insight. The two factors responsible for the slow rate of work are our input and ingestion speeds. I think inherently I am working on increasing the input speeds by shifting from good text to messy, to dictation and now visual. Ingestion speed depends on both the human and the model. Opus 5 didn't feel good because its expression brought confusion. It is not the case with Opus 5.5, and less so with Claude Projects where the outputs focus only on goals and completed work, without unnecessary intermediate details.

I think the frontier now is that we somehow make our ingestion speed closer to the model's output speed and the gap between output and input be minimal. The goal is real-time communication between the model and the human, just as it happens between two humans on the same wavelength.