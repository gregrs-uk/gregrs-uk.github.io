---
layout: post
title: "Setting multiple macOS Calendar events to repeat every n weeks"
date: 2018-11-04
---

The macOS Calendar application allows you to specify that an event should repeat – for example every week or two weeks – and when these repeats should stop. This is useful for me as a teacher because a large number of my calendar events (lessons, meetings, rehearsals, etc.) happen at the same time every week or two weeks. (We have a two-week academic timetable.)

Unfortunately using the Calendar app you can't select multiple events and set them to all have the same recurrence settings. This means quite a lot of clicking is required if you have lots of events to repeat. However, I've written the following AppleScript – which can be run using the 'Script Editor' application – to automate this task.

You first set some variables at the top of the script: the start date, end date and calendars for the events you want to repeat, the number of weeks in between repeats and the date on which you want the repeats to stop (e.g. the end of term). You then run the script and it modifies your calendar events to recur regularly. You can re-run the script with different parameters, for example if you want to repeat events from your lessons calendar every two weeks but events from your rehearsals calendar at the same time every week.

```applescript
# which dates are the existing events on?
set startDate to date ("5 Nov 2018")
set endDate to date ("18 Nov 2018") -- last date to include

# which calendars contain the events?
set specifiedCalendars to {"Teaching", "Meetings"}

# how many weeks in between repeats?
set interval to 2

# date to stop repeats (YYYYMMDD)
set repeatStopDate to "20181212"

tell application "Calendar"
	repeat with thisCalendar in specifiedCalendars
		tell calendar thisCalendar
			set theEvents to (every event where its start date is greater than or equal to startDate and its end date is less than endDate + (1 * days))

			repeat with thisEvent in theEvents
				# calculate weekday part of recurrence string
				set thisEventDate to start date of thisEvent
				if weekday of thisEventDate is Monday then
					set wd to "MO"
				else if weekday of thisEventDate is Tuesday then
					set wd to "TU"
				else if weekday of thisEventDate is Wednesday then
					set wd to "WE"
				else if weekday of thisEventDate is Thursday then
					set wd to "TH"
				else if weekday of thisEventDate is Friday then
					set wd to "FR"
				else if weekday of thisEventDate is Saturday then
					set wd to "SA"
				else if weekday of thisEventDate is Sunday then
					set wd to "SU"
				end if

				# set the recurrence of this event
				set recurrence of thisEvent to "FREQ=WEEKLY;INTERVAL=" & interval & ";UNTIL=" & repeatStopDate & "T235959Z;BYDAY=" & wd & ";WKST=SU"
			end repeat
		end tell
	end repeat
end tell
```

If you have any questions or suggestions, why not get in touch with me on [Twitter](https://twitter.com/gregrs_uk)?
