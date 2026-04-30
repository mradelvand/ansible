# My Learning Blog 

Welcome to my personal documentation space 

---

##  Explore by Category

- [Lab01 Ansible and Vagrant](archive.html#Lab01)
- [Lab02 Ansible and AWS](archive.html#Lab02)

> [📁 Blog Archive](archive.html).

---

###  About Me

Hi, I'm **Reza** — passionate about **Cloud + Network Automation **.  
This blog documents my continuous learning journey .  
Expect practical notes, architectural insights, and publishable mini-projects built from real-world experience.

---

### ❤️ A Dedication

> **To my father**  
> Who recently passed away — my passion for technology and learning comes from him,  
> and this blog is a way to honor his influence and memory.

*In his memory, I keep learning — one post at a time.*
— *Reza*

---

##  Latest Posts

{% for post in site.posts limit:5 %}
- [{{ post.title }}]({{ post.url | relative_url }}) — *{{ post.date | date: "%B %Y" }}*
{% endfor %}
---
