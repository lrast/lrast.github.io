---
layout: project_post
title: "Interactive Docs"
date: 2026-05-14
categories: [demos]
image: /assets/images/demos/interactive_doc_preview.jpg
---

This is a simple demo project, aimed at touching on a weakness in AI systems, from the perspective of a user.
Namely, the output of an LLM model is less certain, and less reliable than the original ground-truth data.
The approach here is to return ground-truth documentation and to ground generated code examples by making the immediately editable and runnable by the user.

![preview](/assets/images/demos/interactive_doc_preview.jpg)


## Technology

This project is mostly 'plumbing', and mostly vibecoded.
The web stack runs a flask backend and react frontend, with terminal connection over websockets.

The AI agents are managed using Pydantic AI, enforcing the output format and providing access to very simple (URL validation) tool-calls.
Other than that, agent is used in an 'open-loop' configuration.
We use e2b (e2b.dev) to provide sandboxed terminal instances. In this case the terminal is exposed to the user rather than the agents.
Finally, as we will address below, we use firecrawl (firecrawl.dev) to extract scrapes of webpages that are not available to be embedded as iframes.


## Challenges

The main challenge that this project faces is limited cross-site availability of some documentation, for example on pypi.org.
This is limited for a very good reason: imposters could attempt to redirect package installs to their own malicious packages.
However, it also imposes a challenge for the balance between ground-truth and generated content: ground-truth content may simply not be available, while scrapes, as we perform here, introduce a new source of noise into the displayed documentation.
This solution is reasonable for the present purposes, but will arise again whenever we want to 'weave' together documents from different sources.
Likely the most durable solution would be to move from assembling documents in a web-app display to a browser application, with full control over local manipulations.



## Links

Source on github: [https://github.com/lrast/interactive_docs](https://github.com/lrast/interactive_docs)

Live version hosted on render: [https://interactive-docs.onrender.com/](https://interactive-docs.onrender.com/) (this may take a minute to start up)
