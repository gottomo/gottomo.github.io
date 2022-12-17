---
layout: post
title:  "Adding a custom export lua script without conflicting with other scripts"
date:   2022-12-17 16:52:07 +0100
categories: dcs export lua script memo
---
I have recently been trying to write a lua script to export some basic flight data into a log file, and as a total lua noob without any prior knowledge, I have gone through many trial and errors to make things work. In this post I record, as a reference for future myself and for others, the issues encountered and the solutions to these issues.

Referencing the [hoggit wiki](https://wiki.hoggitworld.com/view/DCS_export), I wrote a lua script that exports flight data successfully when it is the only script exporting data. As a next step I tried to run this script alongside other plugins such as volanta and tacview by moving the written script into "flight_testFDR.lua", and then adding the lines
```
local flight_testFDRlfs = require('lfs'); dofile(flight_testFDRlfs.writedir().."Scripts/flight_testFDR.lua")
```
into the last line of "export.lua". 
 
My custom data exporter worked without any issue, but then I noticed that tacview's data export stopped working i.e. there was a conflict with other scripts. 
 
The solution to the problem was to firstly add lines
```
local FDRLuaExportStart = LuaExportStart
local FDRLuaExportBeforeNextFrame = LuaExportBeforeNextFrame
local FDRLuaExportAfterNextFrame = LuaExportAfterNextFrame
local FDRLuaExportStop = LuaExportStop
local FDRLuaExportActivityNextEvent = LuaExportActivityNextEvent
```
at the beginning of "flight_testFDR.lua". Next step involved adding lines such as
```
if FDRLuaExportXYZ then
	FDRLuaExportXYZ()
end
```
onto the end of each LuaExportXYZ() functions. As an example, modifying
```
function LuaExportStop()
   if log_file then
      log_file:close()
      log_file = nil
   end
end
```
into
```
function LuaExportStop()
   if log_file then
      log_file:close()
      log_file = nil
   end

   if FDRLuaExportStop then
      FDRLuaExportStop()
   end
end
```
It is a little bit tricky when modyfing the LuaExportActivityNextEvent() function. The sript didn't work if I just added the lines like before. The solution is to not forget to give the argument "t" to the FDRLuaExportActivityNextEvent() function at the end, i.e. write like
```
function LuaExportActivityNextEvent(t)
...
   if FDRLuaExportActivityNextEvent then
      FDRLuaExportActivityNextEvent(t)
   end
   return tNext
end
```

After taking these steps I could successfully run my sript alongside other plugins. What I noticed is that this method only works when adding the line
```local flight_testFDRlfs = require('lfs'); dofile(flight_testFDRlfs.writedir().."Scripts/flight_testFDR.lua")```
to the very last line of the "export.lua". When I added this line to the beginning of the "export.lua", the export did not work.