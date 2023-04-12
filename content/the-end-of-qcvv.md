+++
title = "The End of QCVV"
date = 2023-04-11

[taxonomies]
tags = ["quantum"]
+++

For much of the history of quantum computing, researchers have been obsessed with a little subfield known as _QCVV_ (an initialism only the government could love — but more on that later). A couple decades down the road, we can get some clues about quantum computing as an industry by taking a closer look at that obsession, where it came from, and why it's not quite as relevant now.

The obvious first question: what in the hell does "QCVV" even mean? The name dates back to [US government grant agency announcements](https://web.archive.org/web/20140410210830/http://www.arl.army.mil/www/pages/8/Quantum_Computing_BAA_Final_June2013.pdf) from the early 2010s, and expands to "quantum characterization, verification, and validation." That's a bit of a misnomer, though. By contrast with classical verification and validation, which is concerned mostly with formal proofs of correctness, QCVV is mostly concerned with benchmarks to assess the quality of a given quantum device.

In the early 2010s, such benchmarks were extremely important to grant agencies looking to make decisions about what research groups to fund, and to assess how much progress different groups were making. Back then, if you wanted to ask a question like "how good is this quantum device," there was more or less no real consensus in the field as to how to get an answer, let alone what a reasonable answer might look like.

In practice, this meant that each group would use different sets of metrics from everyone else, making comparisons _across_ groups a nigh-impossibility. It's not that there weren't any known techniques or benchmarks, it's that no one could agree on which ones to use — whether to report process fidelity, diamond norms, tomographic reconstructions of superoperators, some combination of all of them, or even something entirely different.

Having been a relatively junior researcher at the time, late in my PhD program, my experience was that a lot of that confusion ultimately boiled down to a lack of consensus about what we were even trying to assess in the first place. From about 1985 through to the early 2000s, quantum computing was seen mostly as an academic curiousity; sitting where we are today, quantum computing is yesterday's buzzword. Between those two extremes was a field struggling to define itself as the prospect of commercial and military viability forced a primarily academic pursuit to grapple with questions of practicality.

QCVV, then, was to some extent an offshoot of that identity crisis; an attempt to reconcile what it meant for quantum computing research to "succeed." From the perspective of the US government, there was an stunningly clear answer. They wanted quantum computers to exist so that they could run stuff like Shor's algorithm on them, with the hope the breaking public-key cryptography schemes that power the Internet we know and love. That simple demand turns out to have one hell of an implication, feeding directly back into the history of QCVV. In particular, Shor's algorithm takes a lot of qubits to run, as well as a pretty long runtime. Running long programs on large devices takes incredibly small error rates if you want answers to be even remotely accurate.

Back of the envelope, consider running a program that takes 1,000 distinct instructions (commonly known as _gates_) on each of 1,000 distinct qubits. That is, insofar as computing scales go, embarassingly small — much smaller than what would be needed to run any practical quantum application. Even so, to get an average of about one error each time you run the program, that would require an error rate of no more than one in a million. Back in 2006, [it was estimated](https://arxiv.org/pdf/quant-ph/0601097.pdf) that it would take a number of qubits that scales roughly one and a half times the length of some key in order to break that key using Shor's algorithm. Recommended key lengths at the time were at least 2,048 bits, so given what we knew in the early 2010s, you'd at _best_ need about 3,000 qubits running a program that's about 27 billion gates long. Putting that altogether, you'd have needed error rates on the scale of one in a hundred trillion to be able to reliably use Shor's algorithm to break keys that were common at the time.

It's only relatively recently that quantum devices have been able to reliably error rates around 1%, so sitting in the 2010s, it was obvious that there were quite a few orders of magnitude worth of improvement needed to meet what the US goverment might consider "success." A trillion-fold improvement in error rates seems ridiculous on the face of it, leading some researchers to discount the possibility of quantum computing outright.

A decade earlier, in the early 2000s, however, the [_fault-tolerance threshold theorem_](http://arxiv.org/abs/0904.2557) guaranteed that if quantum computers were "good enough," you could use more and more qubits on your device to implement _logical qubits_ that had as small an error rate as you want. That is, once you hit the threshold, requirements on error rates could be exchanged for requirements on qubit count. The threshold theorem gives a precise definition for what is "good enough" to meet that threshold, but in practice, that definition wound up being incredibly hard to _measure_ in an actual physical device.

The goal of the original QCVV grant program was then to come up with techniques and benchmarks that could be used to answer the question as to whether a given physical device was closer or further to the fault-tolerance threshold than some other device. With such a precise question as motivation, the early 2010s saw an explosion of different techniques developed to try and connect experimental observations to mathematical definitions of the fault-tolerance threshold theorem in rigorous ways. Personally, much of my own involvement came in the form of trying to put experimental protocols, sometimes notorious for dodgy stats and hand-wavy appeals to mathematical definitions, on [a more firm statistical basis](https://arxiv.org/abs/1404.5275).

It's difficult to overstate here just how much QCVV protocols and techniques became the focal point of intense arguments and infighting. After all, grant agencies were originally interested in QCVV to make decisions about which groups deserved funding and which groups should have their funding cut. For academic researchers, decisions about QCVV could easily make or break careers. Entire groups could lose their funding, even, putting their graduate students and postdocs into extremely precarious positions. It's no small wonder, then, that friendships and collaborations faced the same kind of existential pressure in a community nearly wholly without a healthy idea of work/life balance.

I promised you right in the overly sensationalized title of this post, though, that there was an "end to QCVV," so there is clearly more to the story than a few overly obsessed researchers duking it out on arXiv as to exactly what benchmarks should govern academic success. Perhaps ironically, the next chapter of quantum computing went pretty much the same as the one that led to the creation of QCVV as a subfield, starting with a question that served to crystalize discussions in the field.

By the late 2010s, the success of commercial demonstrations such as IBM's "Quantum Experience" created a shift away from questions about eventual fault-tolerance towards questions about what could be done with prototype quantum devices that could be used over the web in 2016. For the first time in quantum computing history, error rates of around 1% could be achived reliably enough to offer up as a web service, instead of requiring a small army of PhD students.

Whereas before, the US government and other grant agencies around the world were largely undecided on which kinds of quantum devices to invest in — superconducting qubits, ion trap qubits, silicon dot qubits, or even something like NV center devices — corporate research programs each tended to be more committed to their own platforms. Data that could help decide between different platforms became correspondingly less important as a result, pushing research discussions away from using QCVV to assess fault-tolerance. By 2017, corporate interest in quantum computing had advanced to the point that there was even a conference, [Q2B 2017](https://q2b2017.qcware.com/), focused tightly on the potential business impact of quantum computing.

In 2018, that shift intensified with the publication of a very popular paper arguing that more attention should be focused on what we could do with non–fault tolerant devices. The focus, it was argued, shouldn't _just_ be on making practical quantum computers, but on finding tasks that quantum devices could definitely beat classical devices at. This goal went under the startlingly racist term of "[quantum supremacy](https://www.nature.com/articles/d41586-019-03781-0)," and had the apparent advantage over the fault-tolerance goal of possibly being attainable in only a few years.

Benchmarks, techniques, and protocols for assessing which of a set of candidate platforms might one day achieve fault-tolerance were suddenly less relevant to a world in which decisions about platforms are much less volatile and in which the immediate goal is less about projected far-future devices than about what can be done today, or at least in the immediate future. The history of that shift is also inherently a history of how quantum computing has progressively been considered more applied throughout its history, as well as how the definition of "applied" has become increasingly more business-driven.

---

Thanks for reading the first post in my new newsletter! You can subscribe and help support me going forward at [_Forbidden Transitions_](https://buttondown.email/xgranade). I promise that not all of my posts will be about quantum computing; I look forward to sharing my thoughts about tech more generally, tiny stories that I've written, and random stuff that's none of the above. In short, if you subscribe, you'll help me share more of _me_.