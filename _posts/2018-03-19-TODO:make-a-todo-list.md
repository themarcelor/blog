---
layout: post
author: Marcelo Costa
---
just sharing some notes on how to use Emacs’ org-mode.

Ignore the paradoxically ludic title. In this article I’m just sharing some notes on how to use Emacs’ org-mode.

In this post we will cover how to:

1. Create your TODO list
2. Track the progress of your tasks
3. Schedule tasks
4. Show your agenda

Create your TODO list

Start by playing with a sample todo list. Just add an item with an asterisk (*) and you can hold ​`​alt` and press ENTER to add a new item under the same column. If you wish promote or demote that item you can hold `alt` and use the arrow keys to add or remove asterisks (shifting between columns).

![org_mode_basic](https://themarcelor.github.com/blog/assets/img/org_mode_basic.png)

Here are some useful commands:

TAB to collapse / expand a single section.
SHIFT + TAB to collapse / expand all sections.
SHIFT → & SHIFT ← (right and left arrows) to set TODO and DONE marks to the item on that line.
M – → & M – ← (alt + right and left arrows) to promote / demote a given item.
C-c /      filter tree of TODO items (e.g., “t” will only show the items marked as “TODO”).
M – x org-sort-entries to organize the list in [alphabetic, numeric, creation-date, etc.] order.

—

Track the progress of your tasks
Put a [ 0 % ] next to the first item at the top of the list and it will be automatically updated as you mark “TODO” items as “DONE”.

![org_mode_percent](https://themarcelor.github.com/blog/assets/img/org_mode_percent.png)

If you create checkboxes with the `-  [  ] ` notation, you can tick these checkboxes with C-c C-c.

Schedule tasks
C-c [   adds file to the front of the agenda file list.

C-c C-s    schedule

org_mode_scheduling

C-c C-d    deadline

Show your agenda
M-x org-agenda

( a )  Agenda for the week.

Under ” a ” (All tasks for the current week)

org_mode_agenda

Press ” f ” (forward) to go to the next week.

Press ” b ” (backwards) to go to the previous week.

( t )   all TODOs.

( s )  Search for keywords.

—

Just a quick cheat sheet here, hope you enjoy!

Bonus tip: The org-jira mode
Installation:

M-x list-packages
C-s   (search for “org-jira”)
Press ” i ” (install) to mark the org-jira  for installation.
Press “x” to execute the installation of the marked packages.
Add the following to your ” ~/.emacs.d/init.el ” file:
(setq jiralib-url “<jira_url>”)

(require ‘org-jira)

Then, once you restart Emacs, you can try:
M-x org-jira-get-issues

