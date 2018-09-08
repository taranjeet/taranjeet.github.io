---
layout: post
title:  "First Open Source Contribution"
date:   2015-08-18 18:30:23
categories: open-source-contribution django-admin-tools django
---

Hi!
Well this post came out of excitment. Yeah First Open Source contribution is really exciting. That Green arrow on Codechef and not this __purple Merged__ box can bring someone back from life too. Just Kidding. But yeah seeing all this really encourages and motivates me to do more.

So How did I achieved my First open source contribution?

Well I have previously submitted Pull Request(PR) to Sympy and Scrapy, both of which were not merged due to some merge conflicts. This time it wasn't the case. My PR got merged within half an hour. It was a documentation fix.

I was working on __Django Admin Tools__. It is a django app which beautifies the django admin panel plus you can customize your django admin panel to suit your needs. While working on that I was following there docs. Everything was going fine, but I was not able to load static files of django admin tools into my project. I seriously had no idea what was going wrong. In fact I went through the whole tutorial t see what I was missing. But it was not written in the tutorial. May be because it was too obvious for a django developer. But sometimes we surely miss those obvious things because they are taken for granted. So I thought of contributing that to the docs.

Ok, let me tell you what was the case. In Django you have ```STATICFILES_FINDER``` which tells the location from where to collect the static files. Normally its value is
```
STATICFILES_FINDER = (
    'django.contrib.staticfiles.finders.AppDirectoriesFinder',
    'django.contrib.staticfiles.finders.FileSystemFinder',
)
```

But in my case ```django.contrib.staticfiles.finder.AppDirectoriesFinder``` was commented out. So whenever I was running ```python manage.py collectstatic --no-input``` I was not able to collect the static files from the django admin tools app. But after sometime I figured this out and then I thought that this should be there in the docs. So this formed my way for contributing to the django admin tool docs. 

Now I will tell you in brief how you can contribute to the repositories and submit a PR. First of all fork the project. Forking the project will make a copy of that project into your Github account. If the project url was ```github.com/projectDeveloperName/projectName```, then forking it will make like ```github.com/yourGithubUsername/projectName```. Once you have forked the project, you can clone the project or download a zipped copy of that project. In case of zipped file, it is obvious that you have to extract it. Once in the folder by ```cd``` command, change the files which ever you want to, run it locally and make sure that the changes are as desired by you. Once done, now do a normal git push by
```
git add -A
git commit -m "a proper commit message telling what the commit is all about. This will be the title of your PR"
git push origin master
```
Now you can see in your forked repo the changes that you have made. Its time that you now submit these changes to the master repo from where you have forked the repo. To do this you will have to create a __differential__. Differential as the name says shows the difference between your previous code and the updated code. It shows it line by line for each file. __+__ means that you have added those lines, __-__ means that you have deleted those lines. Now create a PR and submit it to the project. If you have anything in detail to write about mention it in that only.

Ending this post but the most important thing that I forgot to mention about is [Deshraj](https://github.com/deshraj). He is the one from whom I have learned Django and without him I would never have been able to contribute. Thanks Deshraj.