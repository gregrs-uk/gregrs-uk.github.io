---
layout: post
title: "Plain-text tools for teachers and writers"
date: 2017-12-19
---

As a teacher, I write and edit a lot of text. This might take the form of structured notes for pupils or myself, scripts for the flipped-learning videos that I create, or pupils' end-of-term reports. It makes sense to use software that makes this process as efficient as possible. In this post, I'll explain why I've come to use the Markdown format and the free, cross-platform tools Vim and Pandoc as part of my workflow.

## Markdown

Most people would write and edit text using a [WYSIWYG](https://en.wikipedia.org/wiki/WYSIWYG) word processor such as Microsoft Word or OpenOffice Writer. By contrast, Markdown is a [WYSIWYM](https://en.wikipedia.org/wiki/WYSIWYM) ('what you say is what you *mean*') plain-text format. Unlike other file formats, plain text can be read and edited on any kind of computer and always looks the same.

A few [simple rules](https://daringfireball.net/projects/markdown/syntax) define what certain parts of your document mean: for example headings, lists, bold or italic emphasis and hyperlinks.

```
# Here is a heading

Now we have a list:

* Item One
* Item Two
* Item Three

We can make certain words *italic* or **bold** and create hyperlinks, like this 
one to [Wikipedia](http://en.wikipedia.org/).
```

The Markdown format allows you to focus on the content of what you're writing and encourages you to think about the structure of your document rather than spending time tweaking its format. As a bonus, it can be [version controlled](https://en.wikipedia.org/wiki/Version_control) using [Git](https://git-scm.com), which can help you to track changes and work collaboratively, for example using [GitHub](https://github.com) or [BitBucket](https://bitbucket.org).

## Vim

Working with a plain-text format such as Markdown allows you to take advantage of software originally designed for the efficient manipulation of text by software developers. In my opinion the best of these programs is [Vim](https://en.wikipedia.org/wiki/Vim_(text_editor)).

Vim takes time to learn but if – like me – you spend a lot of time writing and editing text, the investment is worth it. The reason why it's not immediately obvious how to use the program but also the reason why it can be so efficient is that it's designed to be operated entirely from the keyboard; this means that no time is wasted using the mouse to navigate but it can take some practice before the key commands are embedded in your muscle memory. The program can also make repetitive tasks – such as copying report paragraphs to be pasted into a web interface – much easier through the use of macros.

Vim can be extended using plugins; some of the ones I use frequently allow [distraction-free writing](https://github.com/junegunn/goyo.vim), folding of Markdown text to temporarily hide parts of a document, and the use of the [Solarized colour scheme](http://ethanschoonover.com/solarized/vim-colors-solarized).

## Pandoc

Markdown and Vim are a great combination when writing and editing text, but when you want to print a document or share it with others you'll often need to convert it into another format that looks pretty. Enter [Pandoc](https://pandoc.org), which can be used to convert Markdown documents into an HTML file for the web, a PDF (using [LaTeX](https://www.latex-project.org)) for printing or sharing, or even a Microsoft Word document. Pandoc is a command-line program and I often run it within Vim using a command such as `!pandoc "%" -so "%".pdf`.

To customise the appearance of the output file, it's possible to use a [YAML block](https://pandoc.org/MANUAL.html#extension-yaml_metadata_block) at the top of your Markdown file, for example:

```
---
fontfamily: charter
fontsize: 12pt
...
```

You can also supply command-line options to Pandoc. Here's a short BASH script I often use to convert Markdown documents into PDFs by running a command such as `~/md2pdf.sh my-document.md`:

```
NAME=`basename "$1" .md`
pandoc "$1" -o "$NAME".pdf \
	-V papersize:a4 \
	-V geometry:margin=1in \
	-V fontfamily:charter \
	-V fontsize:12pt
```

## Final thoughts

Writing using the Markdown format isn't always appropriate but I generally try to stick to plain-text tools where possible, for example the [LaTeX exam class](https://www.ctan.org/pkg/exam) for worksheets and exams, the [LaTeX beamer class](https://www.ctan.org/pkg/beamer) for presentations and [LilyPond](http://lilypond.org) for sheet music. This means that I still get the benefits of editing using Vim and version control using Git.

One area in which I unfortunately haven't been able to make use of Vim is in typing emails, which naturally I do a lot. I have experimented with [Mutt](https://en.wikipedia.org/wiki/Mutt_(email_client)) – a command-line mail client that allows the use of Vim – but found that because many of my colleagues send HTML mail, I was wasting time formatting quoted sections (which had been automatically converted to plain text using [w3m](https://en.wikipedia.org/wiki/W3m)) into appropriate paragraphs when replying. For now I'm sticking with Mail.app.

It might also be worth mentioning that I've converted to using the [Colemak keyboard layout](https://en.wikipedia.org/wiki/Keyboard_layout#Colemak) rather than QWERTY. As with Vim there was a steep learning curve but I can now touch-type comfortably and – a year on – I'm a faster typist than I ever have been previously.

If you have any thoughts about using plain-text formats such as Markdown or tools such as Vim and Pandoc, why not get in touch with me on [Twitter](https://twitter.com/gregrs_uk)?
