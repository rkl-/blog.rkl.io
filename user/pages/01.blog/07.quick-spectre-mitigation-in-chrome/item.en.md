---
title: 'Quick spectre mitigation in chrome'
media_order: 'hacker-2300772_1920.jpg,activate strict site isolation.png'
published: true
date: '10-01-2018 19:10'
taxonomy:
    category:
        - blog
    tag:
        - security
        - spectre
        - meltdown
        - chrome
        - javascript
        - v8
        - hacking
        - epic
---

[Meltdown](https://meltdownattack.com/) and [Spectre](https://meltdownattack.com/) are the most currently hyped topics in the it security world. The vulnerability is on the hardware level, the processor itself. So far I understand after reading the official published whitepapers, a successful meltdown attack is possible, if the attacker is able to run code on the victims machine. A victims machine also means a cloud provider in this case. With meltdown it's an ease to copy the whole physical memory with a maximum speed of around 500 KB/s in the best case and approximately 100 KB/s in the worst case. In all cases, the most worst one is for the victim, because all secrets like passwords and crypto currency private keys are potentially in risk to get leaked to the adversary. You have read correctly, your bitcoins, ethers, adas and what ever can be stolen by a patiently attacker. In case of an infrastructure like Amazons and
Microsofts cloud solutions, this is an epic disaster. Meltdown is the ideal attack to infiltrate such large systems.

Fortunately, meltdown can be mitigated with a software concept, where the kernel (the heart of an operating system like Linux, Android, Windows or Mac) splits the memory for the space where the user applications run more strictly from the memory the kernel only should have access to. The concept is currently heavy developed by the Linux developers and is known under the name [KPTI: Kernel Page Table Isolation](https://en.wikipedia.org/wiki/Kernel_page-table_isolation). This is not an end solution to mitigate meltdown, but the fasted one the community can provide. Also Microsoft and Apple are implementing a similar approach.

While meltdown is perfect for hacking large virtual server environments, there is another attack vector named spectre. Spectre is really interesting and uses similar approaches like meltdown to trick the CPU to leak sensitive information. Further, spectre is extremly dangerous for regular pc users, because this attack can be bootstraped also within the victims browser, and I'm absolutly sure, such attack scenarios will occour in the near future.

What does this mean? It's simple, if you visit a website with a special JavaScript, the spectre attack could already running. The main difference here is, that currently spectre can only be used to leak data from the memory of the affected process. Some of you will know, that this is simple with a local running malware too. Thus, hijacking the memory of an running application is not new. However, spectre is really dangerous for browsers which compiles JavaScript instructions just in time ([JIT](https://en.wikipedia.org/wiki/Just-in-time_compilation)) to native CPU instructions for optimizations, e.g. Googles chrome browser uses the [V8 engine](https://developers.google.com/v8/) for this.

This simply means that, while your browser has such an untrusted website open, you adversary can dump your chrome memory in the worst case with approximately 10 KB/s. This is enough to fetch your in chrome stored passwords! Spectre also can not be mitigated by KPTI. Google and other browser manufactures are heavily working on patching there software as a quick solution.

The good news are, that Googles chrome already has an experimental security feature called `Strict site isolation` on board, and I highly recommend that you enable it right now! The idea is, that every process which renders your pages, can only have one page assigned. This means under the hood, that every page has it's own process. You may remember, with spectre it's only possible to read memory from an infected process. Thus, with this setting enabled, a spectre attack which we described above can only dump the memory of the page which bootstrapped it. The remaining memory of your browser should be safe for now (and I really hope so).

To activate `Strict site isolation`, open this url in your chrome browser: [chrome://flags/#enable-site-per-process](chrome://flags/#enable-site-per-process) and click on activate under **Strict site isolation**.

![activate strict site isolation](activate%20strict%20site%20isolation.png)

After that, confirm the restart of chrome and this feature is enabled.

Some last words to this meltdown and spectre topic. All current mitigations are no end solutions. It is highly up to the hardware manufacturers to patch their products. If you were never paranoid in your life, this is a good time to start.