---
layout: post
title: "Adding n weeks to the defer and due dates of tasks in OmniFocus 2 for Mac"
date: 2019-04-12
---

I use [OmniFocus](https://www.omnigroup.com/omnifocus) every day to organise my life. As a teacher, I have a number of tasks which recur every week or fortnight – my school uses a two-week timetable – for example preparing lessons, setting and marking homework for each of my classes and completing the electronic register for each lesson.

When there's a school holiday, I need to add a specified number of weeks to the defer and due dates of these recurring events.

## The problem

OmniFocus 2 for Mac allows you to add 1 week at a time to the defer or due date of one or more tasks, but as you can see below it doesn't preserve the time that task is due. Instead it sets the time for each task to the default. This is a problem for me as my tasks' times usually relate to the timing of each lesson.

![]({{ "/assets/add-weeks-defer-due-date-omnifocus/omnifocus-add-1-week.gif" | absolute_url }})

## The solution

OmniFocus Pro for Mac can be controlled using [AppleScript](https://inside.omnifocus.com/applescript), so I've written the script below to provide an alternative method of adding a specified number of weeks to the defer/due dates of your tasks. You can add it to OmniFocus as follows:

1. Open the 'Script Editor' application.
2. Copy the code below into a new script and save it on your Desktop. (I called mine 'Add n weeks'.)
3. In OmniFocus, click 'Help' > 'Open Scripts Folder'.
4. Drag the script from your Desktop into this folder.
5. Right-click the toolbar and select 'Customize Toolbar…', then drag the script icon to where you'd like it.

To use the script once it's been added, select one or more tasks, click the script's icon and enter a number of weeks. If the selected tasks have defer/due dates, the specified number of weeks is added to these dates for each task. There is a short delay while the action is performed and once it's finished, a 'Done' window will appear.

``` applescript
tell application "OmniFocus"
	tell front window
		
		if (count of selected trees of content) < 1 then
			display dialog "You need to select at least one task" buttons {"OK"}
			error number -128
			
		else
			set dialog_return to (display dialog "Add how many weeks to the defer and due dates of the selected items?" default answer "" buttons {"OK", "Cancel"} default button 1)
			if button returned of dialog_return is "Cancel" then
				error number -128
			end if
			
			try
				set num_weeks to text returned of dialog_return as integer
			on error number -1700
				display dialog "You need to enter a number of weeks as a number" buttons {"OK"}
				error number -128
			end try
			
			repeat with this_item in selected trees of content
				set this_task to value of this_item
				if defer date of this_task is not missing value then
					set (defer date of this_task) to (defer date of this_task) + (num_weeks * weeks)
				end if
				if due date of this_task is not missing value then
					set (due date of this_task) to (due date of this_task) + (num_weeks * weeks)
				end if
			end repeat
			
			display dialog "Done" buttons {"OK"} default button 1
			
		end if
		
	end tell
end tell
```

If you have any questions or suggestions, why not get in touch with me on [Twitter](https://twitter.com/gregrs_uk)?
