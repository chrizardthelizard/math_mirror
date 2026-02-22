Title: Publishing a page to github pages
Date: 2026-02-14
Category: blog
Author: Chris Blais
Summary: Notes on Using Pelican

## command to generate and run local page
You do not need to remove or clean the output if you have this set in your config: 
```
DELETE_OUTPUT_DIRECTORY = True
```

simply run this command: 
```
pelican ; pelican --listen
```

## preview:
http://localhost:8000/ 

## publishing on github pages: 
[Writeup on making custom home page and other QOL stuff](https://gist.github.com/JosefJezek/6053301)
[pelican doc](https://github.com/getpelican/pelican/blob/b5e20d7f6d8520bb277c7c261eed8c33a53ae4de/docs/tips.rst)

## Useful documentation:
[Running and testing locally](https://docs.getpelican.com/en/latest/quickstart.html)

[publishing to github](https://docs.getpelican.com/en/latest/tips.html)
