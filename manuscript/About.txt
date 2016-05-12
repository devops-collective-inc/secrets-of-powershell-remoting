# Secrets of PowerShell Remoting
Principle author: Don Jones
Contributing author: Dr. Tobias Weltner
With contributions by Dave Wyatt and Aleksandar Nikolik
Cover design by Nathan Vonnahme

---

Introduced in Windows PowerShell 2.0, Remoting is one of PowerShell's most useful, and most important, core technologies. It enables you to run almost any command that exists on a remote computer, opening up a universe of possibilities for bulk and remote administration. Remoting underpins other technologies, including Workflow, Desired State Configuration, certain types of background jobs, and much more. This guide isn't intended to be a complete document of what Remoting is and does, although it does provide a good introduction. Instead, this guide is designed to document all the little configuration details that don't appear to be documented elsewhere.

---

This guide is released under the Creative Commons Attribution-NoDerivs 3.0 Unported License. The authors encourage you to redistribute this file as widely as possible, but ask that you do not modify the document.

**Was this book helpful?** The author(s) kindly ask(s) that you make a tax-deductible (in the US; check your laws if you live elsewhere) donation of any amount to [The DevOps Collective](https://devopscollective.org/donate/) to support their ongoing work.

**Check for Updates!** Our ebooks are often updated with new and corrected content. We make them available in three ways:

* Our main, authoritative [GitHub organization](https://github.com/devops-collective-inc), with a repo for each book. Visit https://github.com/devops-collective-inc/
* Our [GitBook page](https://www.gitbook.com/@devopscollective), where you can browse books online, or download as PDF, EPUB, or MOBI. Using the online reader, you can link to specific chapters. Visit https://www.gitbook.com/@devopscollective
* On [LeanPub](https://leanpub.com/u/devopscollective), where you can download as PDF, EPUB, or MOBI (login required), and "purchase" the books to make a donation to DevOps Collective. You can also choose to be notified of updates. Visit https://leanpub.com/u/devopscollective

GitBook and LeanPub have slightly different PDF formatting output, so you can choose the one you prefer. LeanPub can also notify you when we push updates. Our main GitHub repo is authoritative; repositories on other sites are usually just mirrors used for the publishing process. GitBook will usually contain our latest version, including not-yet-finished bits; LeanPub always contains the most recent "public release" of any book.
