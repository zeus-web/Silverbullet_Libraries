---
tags: meta
description: Displays a Calendar for Journal Files
date: 2025-03-03
---

```space-lua

function generate_calendar(year, month)
    -- use actual date, if nothing given as parameter
    if year == nil or month == nil then
       year = tonumber(os.date("%Y"))
       month = tonumber(os.date("%m"))
    end
  
    local days_in_month = {31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31}
    local month_names = {"Januar", "Februar", "März", "April", "Mai", "Juni", "Juli", "August", "September", "Oktober", "November", "Dezember"}
    local day_names = {"Mo", "Di", "Mi", "Do", "Fr", "Sa", "So"}
    
    -- Check for leap year
    if (year % 4 == 0 and year % 100 ~= 0) or (year % 400 == 0) then
        days_in_month[2] = 29
    end
    
    local days = days_in_month[month]
    
    -- Get the first day of the month (1 = Sunday, 2 = Monday, ..., 7 = Saturday)
    local first_day = tonumber(os.date("%w", os.time{year=year, month=month, day=1}))+1
    
    -- Adjust first day to start from Monday (1 = Monday, ..., 7 = Sunday)
    first_day = first_day - 1
    if first_day == 0 then
        first_day = 7
    end
    
    local html = "<table border='1'>\n"
    html = html .. "<tr><th colspan='7'>" .. month_names[month] .. " " .. year .. "</th></tr>\n"
    html = html .. "<tr>"
    for _, day_name in ipairs(day_names) do
        html = html .. "<th>" .. day_name .. "</th>"
    end
    html = html .. "</tr>\n"
    
    local day = 1
    local started = false
    local journalfilename =""
    local cellcontent =""
    for week = 1, 6 do
        html = html .. "<tr>"
        for weekday = 1, 7 do
            if not started and weekday == first_day then
                started = true
            end
            if started and day <= days then

              -- define Jounral directory and Filenames
              journalfilename = "Journal/" .. year .. "-" .. leadingzero(month) .. "-" .. leadingzero(day) .. "_" .. day_names[weekday]

              if space.fileExists(journalfilename .. ".md") then
                 cellcontent = "<b><a href=\"" .. journalfilename .. "\" class=\"mark\"  data-ref=\"" .. journalfilename .. "\">" ..  day .. "</a></b>"
              else
                 cellcontent = "<a href=\"" .. journalfilename .. "\"  data-ref=\"" .. journalfilename .. "\">" ..  day .. "</a>"
              end
              
              -- mark today
              if day == tonumber(os.date("%d")) and month == tonumber(os.date("%m"))  and year == tonumber(os.date("%Y")) then
                html = html .. "<td class=\"mark\">(" .. cellcontent .. ")</td>"
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
      html=html;
      display="block";
      cssClasses={"calendartable"}
    }
end

local function leadingzero(number)
   return (number < 10 and "0" .. number) or tostring(number)
end

```


```space-style
.calendartable {
  border-collapse: collapse; 
  width: 400px;
}

.calendartable table {
  border-color: var(--panel-border-color);
}

.calendartable th {
  width: 10%;
  padding: 3px; 
  text-align: left;  
  background-color: var(--panel-background-color);
}


.calendartable td {
  width: 10%;
  padding: 3px; 
  text-align: left;
}

.calendartable a {
  text-decoration-line: none;
  color: var(--root-color);
}

.calendartable a.mark {
  text-decoration-line: none;
  color: var(--ui-accent-text-color);
}
```
