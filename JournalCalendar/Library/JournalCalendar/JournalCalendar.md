---
tags: meta
description: Displays a Calendar for Journal Files
version: 0.6.0
date: 2025-03-05
---

# Use the “widget” on your page
Place the command in expression syntax somewhere on your desired page.
You can set the year and month as parameter (e.g. `(2025,3)`)
Leave the parameters empty for current month/year.
`${generate_calendar(2025,3)}`

You are able to place additional markers, too. Just add a third parameter with a LUA table with "x" plus the day's number and a text to display like that:

Example with day nr. 10 is marked "Holiday", day nr. 19 is marked with "Blocked"

`{x10="Holiday", x19="Blocked"}`

_(Hint: you can define how it looks like by changing the CSS Style ".calendartable span.extramarker")_

## Example:
${generate_calendar(2025,3,{x10="Abc", x19="XYZ"} )}

# Settings
For changing the month and day names and other configs, you can place a `space-config` section somewhere in your space. 
**Do not** use it *at this page*, if you install it automatically, as it could get overwritten.
You could safely use it inside your SETTINGS page for example.

Possible options are:
- month names
- day names
- week should not start on monday (but sunday)
- path of your journal files and thier file names
  
```space-config_example
JournalCalendar:
  month_names: ["Januar", "Februar", "März", "April", "Mai", "Juni", "Juli", "August", "September", "Oktober", "November", "Dezember"]
  day_names: ["Mo", "Di", "Mi", "Do", "Fr", "Sa", "So"]
  week_start_on_monday: true
  journal_path_pattern: "Journal/%year%-%month%-%day%_%weekday%"
```

# Script 

```space-lua
-- Configuration: Customize these settings or use a space-config with section JournalCalendar
local week_start_on_monday = true -- Set to false for Sunday start
local journal_path_pattern = "Journal/%year%/%month%/%year%-%month%-%day%_%weekday%"
local month_names =  {"January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"}
local day_names = {"Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"} -- Used when the week starts on Monday

-- overwrite settings, if there is a configuration in space-config
local JC_config = system.getSpaceConfig("JournalCalendar")
if (JC_config) then
  if (JC_config.journal_path_pattern != null) then
    journal_path_pattern = JC_config.journal_path_pattern
  end

  if (JC_config.month_names != null) then
    month_names = JC_config.month_names
  end
  
  if (JC_config.day_names != null) then
    day_names = JC_config.day_names
  end
  
  if (JC_config.week_start_on_monday != null) then
    week_start_on_monday = JC_config.week_start_on_monday
  end
end
  

function generate_calendar(year, month, markers)
    -- Use current date if no parameters are provided
    if year == nil or month == nil then
        year = tonumber(os.date("%Y"))
        month = tonumber(os.date("%m"))
    end

    -- Determine day names based on week start setting
    local selected_day_names = week_start_on_monday and day_names or {day_names[7], day_names[1], day_names[2], day_names[3], day_names[4], day_names[5], day_names[6]}

    local days = get_days_in_month(year, month)

    -- Get the first day of the month (1 = Sunday, 2 = Monday, ..., 7 = Saturday)
    local first_day = tonumber(os.date("%w", os.time{year=year, month=month, day=1})) + 1

    -- Adjust the first day based on the week start setting
    if week_start_on_monday then
        first_day = first_day - 1
        if first_day == 0 then
            first_day = 7
        end
    end

    local html = "<table border='1'>\n"
    html = html .. "<tr><th colspan='7'>" .. month_names[month] .. " " .. year .. "</th></tr>\n"
    html = html .. "<tr>"
    for _, day_name in ipairs(selected_day_names) do
        html = html .. "<th>" .. day_name .. "</th>"
    end
    html = html .. "</tr>\n"

    local day = 1
    local started = false
    for week = 1, 6 do
        html = html .. "<tr>"
        for weekday = 1, 7 do
            if not started and weekday == first_day then
                started = true
            end
            if started and day <= days then
                -- Generate journal file path using the defined pattern
                local journalfilename = generate_journal_path(year, month, day, weekday, journal_path_pattern)

                local cellcontent = ""
                if space.fileExists(journalfilename .. ".md") then
                    cellcontent = "<b><a href=\"" .. journalfilename .. "\" class=\"mark\" data-ref=\"" .. journalfilename .. "\">" .. day .. "</a></b>"
                else
                  if (markers != nil and markers["x" .. day] != nil ) then -- add special dates/markings
                    cellcontent = "<a href=\"" .. journalfilename .. "\" data-ref=\"" .. journalfilename .. "\">" .. day .. " <span class=\"extramarker\">(" .. markers["x" .. day] .. ")</div></a>"
                  else
                    cellcontent = "<a href=\"" .. journalfilename .. "\" data-ref=\"" .. journalfilename .. "\">" .. day .. "</a>"
                  end
                end

                -- Mark today's date
                if day == tonumber(os.date("%d")) and month == tonumber(os.date("%m")) and year == tonumber(os.date("%Y")) then
                    html = html .. "<td class=\"today\">(" .. cellcontent .. ")</td>"
                else
                    html = html .. "<td>" .. cellcontent .. "</td>"
                end

                day = day + 1
            else
                html = html .. "<td></td>"
            end
        end
        html = html .. "</tr>\n"
        if day > days then
            break
        end
    end

    html = html .. "</table>"

    return {
        html = html,
        display = "block",
        cssClasses = {"calendartable"}
    }
end

-- Local function to generate journal file path using the defined pattern
local function generate_journal_path(year, month, day, weekday, pattern)    
    -- Ensure we use leading zero format where necessary
    local replacements = {
        ["%%year%%"] = tostring(year),
        ["%%month%%"] = leadingzero(month),
        ["%%day%%"] = leadingzero(day),
        ["%%weekday%%"] = day_names[weekday]
    }

    -- Replace placeholders in the pattern
    for key, value in pairs(replacements) do
        pattern = string.gsub(pattern, key, value)
    end

    return pattern
end
-- Local function to ensure two-digit formatting
local function leadingzero(number)
    return (number < 10 and "0" .. number) or tostring(number)
end

-- Local function to get the correct number of days in a month (handles leap years)
local function get_days_in_month(year, month)
    local days_in_month = {31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31}
    
    -- Check for leap year and adjust February
    if month == 2 and ((year % 4 == 0 and year % 100 ~= 0) or (year % 400 == 0)) then
        return 29
    end
    
    return days_in_month[month]
end    


```

# Space Style

*Hint*: For changes, copy this to another page, as it might get overwritten by updates, if you use automatic library updates.

```space-style
.calendartable {
  border-collapse: collapse; 
  width: 500px;
}

.calendartable table {
  border-color: var(--panel-border-color);
}

.calendartable th {
  width: 14%;
  padding: 3px; 
  text-align: left;  
  background-color: var(--panel-background-color);
}


.calendartable td {
  width: 14%;
  padding: 3px; 
  text-align: left;
}

.calendartable td.today {
  background-color: var(--ui-accent-color);
}

.calendartable a {
  text-decoration-line: none;
  color: var(--root-color);
}

.calendartable a.mark {
  text-decoration-line: none;
  color: var(--ui-accent-text-color);
}

.calendartable span.extramarker {
  background-color: yellow;
}

```
