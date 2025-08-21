---
Status: Finished
Author:
  - John Ousterhout
Publishing/Release Date: 2019-01-22
Publisher: Yaknyam Press Palo Alto CA
Link: https://www.amazon.com/Philosophy-Software-Design-John-Ousterhout-ebook-dp-B07N1XLQ7D/dp/B07N1XLQ7D/ref=mt_other?_encoding=UTF8&me=&qid=
Type: Book
"\bGenre":
  - IT
---
  

“This book is about how to design software systems to minimize their complexity.”

Excerpt From  
A Philosophy of Software Design  
John Ousterhout  
  
[https://itunes.apple.com/WebObjects/MZStore.woa/wa/viewBook?id=0](https://itunes.apple.com/WebObjects/MZStore.woa/wa/viewBook?id=0)  
This material may be protected by copyright.  

# Symptoms of complexity

## **Change amplification**

The first symptom of complexity is that a seemingly simple change requires code modifications in many different places. For example, consider a Web site containing several pages, each of which displays a banner with a background color. In many early Web sites, the color was specified explicitly on each page, as shown in Figure 2.1(a). In order to change the background for such a Web site, a developer might have to modify every existing page by hand; this would be nearly impossible for a large site with thousands of pages. Fortunately, modern Web sites use an approach like that in Figure 2.1(b), where the banner color is specified once in a central place, and all of the individual pages reference that shared value. With this approach, the banner color of the entire Web site can be changed with a single modification. One of the goals of good design is to reduce the amount of code that is affected by each design decision, so design changes don’t require very many code modifications.

## **Cognitive load**

The second symptom of complexity is cognitive load, which refers to how much a developer needs to know in order to complete a task. A higher cognitive load means that developers have to spend more time learning the required information, and there is a greater risk of bugs because they have missed something important. For example, suppose a function in C allocates memory, returns a pointer to that memory, and assumes that the caller will free the memory. This adds to the cognitive load of developers using the function; if a developer fails to free the memory, there will be a memory leak. If the system can be restructured so that the caller doesn’t need to worry about freeing the memory (the same module that allocates the memory also takes responsibility for freeing it), it will reduce the cognitive load. Cognitive load arises in many ways, such as APIs with many methods, global variables, inconsistencies, and dependencies between modules.  
System designers sometimes assume that complexity can be measured by lines of code. They assume that if one implementation is shorter than another, then it must be simpler; if it only takes a few line  

## **Unknown unknowns**

The third symptom of complexity is that it is not obvious which pieces of code must be modified to complete a task, or what information a developer must have to carry out the task successfully. Figure 2.1(c) illustrates this problem. The Web site uses a central variable to determine the banner background color, so it appears to be easy to change. However, a few Web pages use a darker shade of the background color for emphasis, and that darker color is specified explicitly in the individual pages. If the background color changes, then the the emphasis color must change to match. Unfortunately, developers are unlikely to realize this, so they may change the central bannerBg variable without updating the emphasis color. Even if a developer is aware of the problem, it won’t be obvious which pages use the emphasis color, so the developer may have to search every page in the Web site.  
Of the three manifestations of complexity, unknown unknowns are the worst. An unknown unknown means that there is something you need to know, but there is no way for you to find out what it is, or even whether there is an issue