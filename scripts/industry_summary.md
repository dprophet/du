{
  "slots": {
    "0": {
      "name": "slot1",
      "type": {
        "events": [],
        "methods": []
      }
    },
    "1": {
      "name": "slot2",
      "type": {
        "events": [],
        "methods": []
      }
    },
    "2": {
      "name": "slot3",
      "type": {
        "events": [],
        "methods": []
      }
    },
    "3": {
      "name": "slot4",
      "type": {
        "events": [],
        "methods": []
      }
    },
    "4": {
      "name": "slot5",
      "type": {
        "events": [],
        "methods": []
      }
    },
    "5": {
      "name": "slot6",
      "type": {
        "events": [],
        "methods": []
      }
    },
    "6": {
      "name": "slot7",
      "type": {
        "events": [],
        "methods": []
      }
    },
    "7": {
      "name": "slot8",
      "type": {
        "events": [],
        "methods": []
      }
    },
    "8": {
      "name": "slot9",
      "type": {
        "events": [],
        "methods": []
      }
    },
    "9": {
      "name": "slot10",
      "type": {
        "events": [],
        "methods": []
      }
    },
    "-1": {
      "name": "unit",
      "type": {
        "events": [],
        "methods": []
      }
    },
    "-2": {
      "name": "system",
      "type": {
        "events": [],
        "methods": []
      }
    },
    "-3": {
      "name": "library",
      "type": {
        "events": [],
        "methods": []
      }
    }
  },
  "handlers": [
    {
      "code": "version = \"1.5\"\n\nenableRefinerMonitoring = true --export: enable or disable monitoring of refiners\nenableAssemblyMonitoring = true --export: enable or disable monitoring of assembly lines\nenableSmelterMonitoring = true --export: enable or disable monitoring of smelters\nenableChemicalMonitoring = true --export: enable or disable monitoring of chemical industries\nenableElectronicsMonitoring = true --export: enable or disable monitoring of electronics industries\nenableGlassMonitoring = true --export: enable or disable monitoring of glass furnaces\nenableHoneycombMonitoring = true --export: enable or disable monitoring of honeycomb refineries\nenableRecyclerMonitoring = true --export: enable or disable monitoring of recyclers\nenableMetalworkMonitoring = true --export: enable or disable monitoring of metalwork industries\nenable3DPrinterMonitoring = true --export: enable or disable monitoring of 3D Printers\nenableTransferMonitoring = true --export: enable or disable monitoring of transfer units\nenableSorting = true --export: when enabled the screen is sorted by industry type and elements cycle by id\noutputFullIsError = false --export: whether to treat the output full error as an error or normal\ndeactivateScreens = true --export: enable or disable deactivating the screens on script shutdown\nscreenRefreshRate = 0 --export: how often the screens refresh in seconds\nindustryRefreshRate = 5 --export: how often the core is queried for industry elements in seconds\nlandscapeMode = true --export: whether the screen is mounted in landscape or portrait orientation\nheaderColor = \"white\" --export: color to use for headers\ntextColor = \"gray\" --export: color to use for default text color\nstoppedColor = \"sienna\" --export: color to use for industry types with stopped machines\nerrorColor = \"darkred\" --export: color to use for industry types with errored machines\nbackgroundColor = \"black\" --export: color to use for the background\nborderColor = \"gray\" --export: color to use for borders\n\ncore = false\nscreen = false\nscreens = {}\ntotals = {}\nstickers = {}\nscreenRow = 0\nmouseState = 0\nmouseClicked = false\nclickedColumn = false\nclickedRow = false\ntotalWidth = 11\ntotalPadding = 1\nerrorWidth = 14\nerrorPadding = 1\nstoppedWidth = 16\nstoppedPadding = 1\ntypePadding = 1\nheaderHeight = 11.48\nrowHeight = 6.7\ncoreWorldOffset = 16\nmachineIndex = -1\ncolumnStops = {\n\t[\"stopped\"] = {\n        [\"max\"] = 100, \n        [\"min\"] = 100 - stoppedWidth\n\t},\n\t[\"errors\"] = {\n        [\"max\"] = 100 - stoppedWidth, \n        [\"min\"] = 100 - stoppedWidth - errorWidth\n\t},\n\t[\"total\"] = {\n        [\"max\"] = 100 - stoppedWidth - errorWidth, \n        [\"min\"] = 100 - stoppedWidth - errorWidth - totalWidth\n\t},\n\t[\"type\"] = {\n        [\"max\"] = 100 - stoppedWidth  - errorWidth - totalWidth, \n        [\"min\"] = 0\n\t},\n}\nvec3 = require('cpml.vec3')\n\ntypes = {\n\t[\"assembly line\"] = enableAssemblyMonitoring,\n\t[\"glass furnace\"] = enableGlassMonitoring,\n\t[\"3d printer\"] = enable3DPrinterMonitoring,\n\t[\"smelter\"] = enableSmelterMonitoring,\n\t[\"recycler\"] = enableRecyclerMonitoring,\n\t[\"refinery\"] = enableHoneycombMonitoring,\n\t[\"refiner\"] = enableRefinerMonitoring,\n\t[\"transfer unit\"] = enableTransferMonitoring,\n\t[\"chemical industry\"] = enableChemicalMonitoring,\n\t[\"electronics industry\"] = enableElectronicsMonitoring,\n\t[\"metalwork industry\"] = enableMetalworkMonitoring\n}\n\nfunction initializeSlots()\n\tfor slot_name, slot in pairs(unit) do\n\t\tif type(slot) == \"table\" and type(slot.export) == \"table\" and slot.getElementClass then\n\t\t\tlocal elementClass = slot.getElementClass():lower()\n\t\t\tif elementClass:find(\"coreunit\") then\n\t\t\t\tcore = slot\n\t\t\telseif elementClass == \"screenunit\" then\n\t\t\t\tscreens[#screens + 1] = {\n\t\t\t\t\telement = slot,\n\t\t\t\t\tid = slot.getId(),\n\t\t\t\t\tslotName = slot_name\n\t\t\t\t}\n\t\t\tend\n\t\tend\n\tend\nend\n\nfunction getScreen()\n\tfor i, screen in pairs(screens) do\n        return screen\n\tend\n\n\treturn false\nend\n\nfunction normalizeType(typeName)\n\tlocal type = typeName:lower()\n\treturn type:gsub('basic ', ''):gsub('uncommon ', ''):gsub('advanced ', ''):gsub('rare ', ''):gsub('exotic ', '')\nend\n\nfunction isIndustryEnabled(typeName)\n\tif types[typeName] then return true else return false end\nend\n\nfunction compareTypes(l, r)\n\treturn l.actualType:lower() < r.actualType:lower()\nend\n\nfunction updateElements()\n\ttotals = {}\n\tlocal totalKeys = {}\n\tlocal elementIds = core.getElementIdList()\n\n\tfor _, id in ipairs(elementIds) do\n\t\tlocal actualType = core.getElementTypeById(id)\n\t\tif actualType == nil then actualType = \"\" end\n\n\t\tlocal type = normalizeType(actualType)\n\t\tif isIndustryEnabled(type) then\n\t\t\tif totalKeys[actualType] == nil then\n\t\t\t\ttotalKeys[actualType] = #totals + 1\n\t\t\t\ttotals[totalKeys[actualType]] = {\n\t\t\t\t\t[\"actualType\"] = actualType,\n\t\t\t\t\t[\"total\"] = 0,\n\t\t\t\t\t[\"errors\"] = 0,\n\t\t\t\t\t[\"stopped\"] = 0,\n\t\t\t\t\t[\"elements\"] = {}\n\t\t\t\t}\n\t\t\tend\n\t\t\ttotals[totalKeys[actualType]].total = totals[totalKeys[actualType]].total + 1\n\n\t\t\tlocal eJson = core.getElementIndustryStatus(id)\n\t\t\tif eJson == nil then eJson = \"\" end\n\n\t\t\tlocal state = eJson:match([[\"state\":\"([^\"]*)\"]]):gsub([[_]],\" \")\n\t\t\tif state == nil then state = \"\" end\n\n\t\t\tif state == \"PENDING\" then goto continue end\n\t\t\tif state == \"RUNNING\" then goto continue end\n\t\t\tif not outputFullIsError and state:match(\"JAMMED OUTPUT FULL\") then goto continue end\n\n\t\t\tif state:match('STOPPED') then\n\t\t\t\ttotals[totalKeys[actualType]].stopped = totals[totalKeys[actualType]].stopped + 1\n\t\t\tend\n\n\t\t\tif state:match('JAMMED') then\n\t\t\t\ttotals[totalKeys[actualType]].errors = totals[totalKeys[actualType]].errors + 1\n\t\t\tend\n\n\t\t\ttable.insert(totals[totalKeys[actualType]].elements, {\n\t\t\t\t[\"id\"] = id,\n\t\t\t\t[\"status\"] = state,\n\t\t\t\t[\"actualType\"] = actualType\n\t\t\t})\n\t\t\t::continue::\n\t\tend\n\tend\n\n\tif enableSorting then\n\t\ttable.sort(totals, compareTypes)\n\tend\nend\n\nfunction getContainerOrientation()\n\tlocal html = \"\"\n\tif not landscapeMode then\n\t\thtml = [[\n.containerLayout{\n\twidth:100vh;\n\theight:100vw;\n\ttransform:rotate(-90deg);\n\ttransform-origin:0% 0%;\n\tmargin-top:100vh;\n\tbackground:]] .. backgroundColor .. [[;\n}]]\n\telse\n\t\thtml = [[\n.containerLayout{\n\twidth:100vw;\n\theight:100vh;\n\tbackground:]] .. backgroundColor .. [[;\n}]]\n\tend\n\n\treturn html\nend\n\nfunction updateSummary()\n\tlocal html = \"<style>\" .. getContainerOrientation()\n\thtml = html .. [[\n.headerColor{\n\tcolor:]] .. headerColor .. [[;\n}\n.headerLayout{\n\tfont-family=\"Arial\";\n\tfont-size: 6vh;\n\ttext-align:center;\n}\n.tableLayout {\n\twidth:100vw;\n\ttable-layout: fixed;\n}\n.rowLayout {\n\tfont-family=\"Arial\";\n\tfont-size: 5vh;\n\tcolor: ]] .. textColor .. [[s\n\tborder-bottom-width: 1px;\n\tborder-bottom-style: solid;\n\tborder-bottom-color: ]] .. borderColor .. [[;\n}\n.totalColumn {\n    width: ]] .. totalWidth ..[[vw;\n    min-width: ]] .. totalWidth ..[[vw;\n    padding-right: ]] .. totalPadding ..[[vw;\n    text-align: right;\n}\n.errorColumn {\n    width: ]] .. errorWidth ..[[vw;\n    min-width: ]] .. errorWidth ..[[vw;\n    padding-right: ]] .. errorPadding ..[[vw;\n    text-align: right;\n}\n.stoppedColumn {\n    width: ]] .. stoppedWidth ..[[vw;\n    min-width: ]] .. stoppedWidth ..[[vw;\n    padding-right: ]] .. stoppedPadding ..[[vw;\n    text-align: right;\n}\n.typeColumn {\n\tpadding-left: ]] .. typePadding ..[[vw;\n}\n.columnLayout {\n    overflow: hidden;\n    text-overflow: ellipsis;\n    white-space: nowrap;\n}\n</style>\n<div class=\"containerLayout\">\n<div class=\"headerLayout headerColor\">Industry Summary (v]] .. version .. [[)</div>\n<table class=\"tableLayout\">\n<thead>\n\t<tr class=\"rowLayout headerColor\">\n\t\t<th class=\"columnLayout typeColumn\">Industry Type</th>\n\t\t<th class=\"columnLayout totalColumn\">Total</th>\n\t\t<th class=\"columnLayout errorColumn\">Errors</th>\n\t\t<th class=\"columnLayout stoppedColumn\">Stopped</th>\n\t</tr>\n</thead>\n<tbody>]]\n\n\tfor machine = screenRow + 1, #totals do\n\t\tlocal rowColor = backgroundColor;\n\t\tif totals[machine].errors > 0 then\n\t\t\trowColor = errorColor\n\t\telseif totals[machine].stopped > 0 then\n\t\t\trowColor = stoppedColor\n\t\tend\n\t\thtml = html .. [[<tr class=\"rowLayout\" style=\"background-color: ]] .. rowColor .. [[;\">]]\n\t\thtml = html .. [[<td class=\"columnLayout typeColumn\">]] .. totals[machine].actualType .. [[</td>]]\n\t\thtml = html .. [[<td class=\"columnLayout totalColumn\">]] .. totals[machine].total .. [[</td>]]\n\t\thtml = html .. [[<td class=\"columnLayout errorColumn\">]] .. totals[machine].errors .. [[</td>]]\n\t\thtml = html .. [[<td class=\"columnLayout stoppedColumn\">]] .. totals[machine].stopped .. [[</td>]]\n\t\thtml = html .. [[</tr>]]\n     end\n\n\thtml = html .. [[</tbody></table></div>]]\n\tscreen.element.setHTML(html)\nend\n\nfunction UpdateStickers(column, row, index)\n\tif column ~= false then\n\t\tlocal ids = GetIds(column, row)\n\t\tif #ids == 0 then return end\n\n\t\tlocal idIndex = ((index + 1) % (#ids)) + 1\n\t\tlocal position = core.getElementPositionById(ids[idIndex])\n\t\tlocal vector = AdjustForCore(vec3(position))\n\t\tlocal rotation = core.getElementRotationById(ids[idIndex])\n\n\t\tif #stickers > 0 then\n\t\t\tMoveStickers(vector)\n\t\telse\n\t\t\tCreateStickers(vector)\n\t\tend\n\telse\n\t\tDeleteAllStickers()\n\tend\nend\n\nfunction DeleteAllStickers()\n\tfor i, sticker in ipairs(stickers) do\n\t\tcore.deleteSticker(stickers[i])\n\tend\n\tstickers = {}\nend\n\nfunction CreateStickers(vector)\n\tlocal zOffset = 1.5\n\n\ttable.insert(stickers, core.spawnArrowSticker(vector.x, vector.y, vector.z, \"down\"))\n\ttable.insert(stickers, core.spawnArrowSticker(vector.x, vector.y, vector.z, \"down\"))\n\tcore.rotateSticker(stickers[2], 0, 0, 90)\n\n\ttable.insert(stickers, core.spawnArrowSticker(vector.x, vector.y, vector.z + zOffset, \"north\"))\n\ttable.insert(stickers, core.spawnArrowSticker(vector.x, vector.y, vector.z + zOffset, \"north\"))\n\tcore.rotateSticker(stickers[4], 90, 90, 0)\n\n\ttable.insert(stickers, core.spawnArrowSticker(vector.x, vector.y, vector.z + zOffset, \"south\"))\n\ttable.insert(stickers, core.spawnArrowSticker(vector.x, vector.y, vector.z + zOffset, \"south\"))\n\tcore.rotateSticker(stickers[6], 90, -90, 0)\n\n\ttable.insert(stickers, core.spawnArrowSticker(vector.x, vector.y, vector.z + zOffset, \"east\"))\n\ttable.insert(stickers, core.spawnArrowSticker(vector.x, vector.y, vector.z + zOffset, \"east\"))\n\tcore.rotateSticker(stickers[8], 90, 0, 90)\n\n\ttable.insert(stickers, core.spawnArrowSticker(vector.x, vector.y, vector.z + zOffset, \"west\"))\n\ttable.insert(stickers, core.spawnArrowSticker(vector.x, vector.y, vector.z + zOffset, \"west\"))\n\tcore.rotateSticker(stickers[10], -90, 0, 90)\nend\n\nfunction AdjustForCore(vector)\n\tvector.x = vector.x - coreWorldOffset\n\tvector.y = vector.y - coreWorldOffset\n\tvector.z = vector.z - coreWorldOffset\n\treturn vector\nend\n\nfunction MoveStickers(vector)\n\tlocal zOffset = 1.5\n\tlocal nsOffset = 1.0\n\tlocal ewOffset = 2.0\n\n\tcore.moveSticker(stickers[1], vector.x, vector.y, vector.z + zOffset + 1)\n\tcore.moveSticker(stickers[2], vector.x, vector.y, vector.z + zOffset + 1)\n\n\tcore.moveSticker(stickers[3], vector.x + nsOffset, vector.y, vector.z + zOffset)\n\tcore.moveSticker(stickers[4], vector.x + nsOffset, vector.y, vector.z + zOffset)\n\n\tcore.moveSticker(stickers[5], vector.x - nsOffset, vector.y, vector.z + zOffset)\n\tcore.moveSticker(stickers[6], vector.x - nsOffset, vector.y, vector.z + zOffset)\n\n\tcore.moveSticker(stickers[7], vector.x, vector.y - ewOffset, vector.z + zOffset)\n\tcore.moveSticker(stickers[8], vector.x, vector.y - ewOffset, vector.z + zOffset)\n\n\tcore.moveSticker(stickers[9], vector.x, vector.y + ewOffset, vector.z + zOffset)\n\tcore.moveSticker(stickers[10], vector.x, vector.y + ewOffset, vector.z + zOffset)\nend\n\nfunction GetIds(column, row)\n\tlocal ids = {}\n\tlocal status = totals[row];\n\tfor key, machine in pairs(status.elements) do\n\t\tif (column == \"stopped\" and machine.status == \"STOPPED\") or (column == \"errors\" and machine.status:match('JAMMED')) then\n\t\t\tids[#ids+1] = machine.id\n\t\tend\n\tend\n\tif enableSorting then\n\t\ttable.sort(ids)\n\tend\n\treturn ids\nend\n\ninitializeSlots()\nscreen = getScreen()\n\nif core ~= false and screen ~= false then\n\tlocal coreHP = core.getMaxHitPoints()\n\tif coreHP > 10000 then\n\t\tcoreWorldOffset = 128\n\telseif coreHP > 1000 then\n\t\tcoreWorldOffset = 64\n\telseif coreHP > 150 then\n\t\tcoreWorldOffset = 32\n\telse\n\t\tcoreWorldOffset = 16\n\tend\n\n\tupdateElements()\n\tupdateSummary()\n\n\tscreen.element.activate()\n\n\tunit.setTimer(\"ReadMouse\")\n\n\tif screenRefreshRate > 0 then\n\t\tunit.setTimer(\"UpdateScreen\", screenRefreshRate)\n\t\tunit.setTimer(\"UpdateStickers\")\n\telse\n\t\tunit.setTimer(\"UpdateScreen\", screenRefreshRate)\n\t\tunit.setTimer(\"UpdateStickers\")\n\tend\n\tif industryRefreshRate > 0 then\n\t\tunit.setTimer(\"UpdateElements\", industryRefreshRate)\n\telse\n\t\tunit.setTimer(\"UpdateElements\")\n\tend\nend",
      "filter": {
        "args": [],
        "signature": "start()",
        "slotKey": "-1"
      },
      "key": "0"
    },
    {
      "code": "if screen ~= false then\n\t-- Detect Interaction and Update Backend\n\tlocal currentY = 100 * screen.element.getMouseY()\n\tlocal currentX = 100 * screen.element.getMouseX()\n\n\t-- Browser Scroll\n\tif (currentY > headerHeight) then\n\t\tlocal wheel = system.getMouseWheel()\n\t\tif wheel ~= 0 then\n\t\t\tlocal adjusted = math.floor(math.max(math.min(wheel, 1), -1) + 0.5)\n\t\t\tlocal rawIndex = screenRow - (adjusted * 4)\n\t\t\tscreenRow = math.max(math.min(rawIndex, #totals - 1), 0)\n\t\tend\n\n\t\tlocal newState = screen.element.getMouseState()\n\t\tif mouseState == 1 and newState == 0 then\n\t\t\tlocal newRow = math.floor((currentY - headerHeight) / rowHeight)\n\t\t\tlocal newColumn = false\n\t\t\tfor key, value in pairs(columnStops) do\n\t\t\t\tif currentX <= value.max and currentX >= value.min then\n\t\t\t\t\tnewColumn = key\n\t\t\t\tend\n\t\t\tend\n\t\t\tif clickedColumn == newColumn and clickedRow == newRow then\n\t\t\t\tmachineIndex = machineIndex + 1\n\t\t\telse\n\t\t\t\tmachineIndex = -1\n\t\t\tend\n\t\t\tclickedColumn = newColumn\n\t\t\tclickedRow = newRow\n\t\t\tmouseClicked = true\n\t\tend\n\t\tmouseState = newState\n\tend\nend",
      "filter": {
        "args": [
          {
            "value": "ReadMouse"
          }
        ],
        "signature": "tick(timerId)",
        "slotKey": "-1"
      },
      "key": "1"
    },
    {
      "code": "if deactivateScreens then\n    if screen ~= false then screen.element.deactivate() end\nend\n",
      "filter": {
        "args": [],
        "signature": "stop()",
        "slotKey": "-1"
      },
      "key": "2"
    },
    {
      "code": "if screen ~= false then\n\tupdateSummary()\nend",
      "filter": {
        "args": [
          {
            "value": "UpdateScreen"
          }
        ],
        "signature": "tick(timerId)",
        "slotKey": "-1"
      },
      "key": "3"
    },
    {
      "code": "updateElements()",
      "filter": {
        "args": [
          {
            "value": "UpdateElements"
          }
        ],
        "signature": "tick(timerId)",
        "slotKey": "-1"
      },
      "key": "4"
    },
    {
      "code": "if mouseClicked then\n    UpdateStickers(clickedColumn, screenRow + clickedRow + 1, machineIndex)\n    mouseClicked = false\nend",
      "filter": {
        "args": [
          {
            "value": "UpdateStickers"
          }
        ],
        "signature": "tick(timerId)",
        "slotKey": "-1"
      },
      "key": "5"
    }
  ],
  "methods": [],
  "events": []
}
