# RCJ Rescue Line - Digital Scoring Sheet
The aim of this project is to replace the paper scoring sheets at RCJ Rescue Line with digital ones on mobile devices.

It's planned to use this software at RCJ Berlin (29.02.-01.03.2020) if the project is finished until then.

The implementation is according to rules of 2019. An implementation for 2020-rules will come later.

## Guide for Referees/Users
Note: During development it is being tested with Firefox and at greater intervals even with Chrome.
If you face issues with other browsers (e.g. Safari), feedback is highly appreciated (of course also if problems occur with Firefox/Chrome).

**You will find the guide [here](./docs/guide.md).**

## Documentation

### Requirements
This project must meet certain demands so that it can be used during competition:
* all requirements deduced from the rules must be implemented and work correctly
* functionality has to be as good as with paper scoring sheets (if not better)
* referees should use their own smartphones &rarr; many different devices should be supported (various operating systems, browsers, ...)
* requirements for user interface:
  * should work for various screen sizes (size, aspect ratio)
  * UI elements have to be big enough so that every text can be read and every button clicked easily
  * all steps during scoring run should go fast and without scrolling
  * resistant against incorrect handling (closing app / page reload, clicking wrong button, ...)
* energy consumption of devices should not be increased significantly
* information for all runs have to be collected from multiple devices to create standings/rankings
* the internet connection at competition could be slow &rarr; traffic should be reduced to a minimum
* problems which might occur but which must not affect the competition
  * short internet outages
  * devices/systems could crash/freeze

### What's planned?
* web page (optimized for smartphones)
* Single-Page-Application
* storing all data locally and send everything to an evaluation system where all runs are brought together
* UI consisting of 8 different parts:
  1. Referee login; specify competition (Line / Line Entry), field, round
  2. Select teamname (from list of all teams); select kind of evacuation point (low/high) in Rescue Line
  3. Measure time for calibration and setting checkpoints; team showed up
  4. Main UI, enter all information during scoring run: start/pause time; section completed, Lack of Progess, skipped section; award points for gaps, obstacles, speed bumps, ramps and intersections; see current status (time, section, try, number of scoring elements, ...); reached last checkpoint; undo steps
  5. Enter number of tiles per section; enter number of rescued victims; left evacuation zone successfully
  6. Review everything together with team captain and correct things if needed; text input for complaints; signature of team captain
  7. Send whole sheet to central evaluation system; get confirmation in case of successful transmission
  8. View all locally saved data (including status whether transmitted successfully or not), possibility to send it again or to export data (&rarr; other medium for transmission can be used); it should be possible to change name/password for all (past) runs
* navigation between these parts (mostly only back/next)
* step 1 will be used when competition starts / referee switches field / next round starts
* normally you will go through step 2 till 7 sequentially for each team
* step 8 will only be used if data transmission fails
* integration of timetable is not planned yet but may be added later

### Architecture / Technical Planning
* to store data for a longer period of time LocalStorage is used; what's stored:
  * current page (SPA, for reloading)
  * current run (JSON with sections, tries, scoring elements, time, ...)
  * history of runs (with all details needed for later evaluation)
* to implement Single-Page-Application: multiple `div`'s which can be hidden through JavaScript
* Main UI should work without scrolling, everything should fit on screen: everything is in a 3:2-box (as large as possible)
* Undo-function should be implemented by logging all user input (in main UI)

Example object for a run:
```
run = {
    referee: {
        name: "NL",
        auth: "wbSwFU6tY1c-uczbe"
    },
    competition: "line",
    arena: "A",
    round: "2",
    teamname: "pi++",
    evacuationPoint: "high",
    time: {
        timeOffset: 57.82108,
        timeStartedTimestamp: 1555975800,
        timestampRunStart: 1554854400,
        timestampRunEnd: null
    },
    teamStarted: true,
    sections: [
        {
            sectionId: 1,
            completedSection: true,
            skippedSection: false,
            lops: 5,
            isAfterLastCheckpoint: false,
            gaps: 7,
            obstacles: 8,
            speedbumps: 2,
            ramps: 1,
            intersections: 0,
            tiles: 8
        },
        ...
    ],
    victims: {
        deadVictimsBeforeAllLivingVictims: 3,
        livingVictims: 2,
        deadVictimsAfterAllLivingVictims: 0
    },
    leftEvacuationZone: false,
    comments: "",
    confirmedByTeamCaptain: true,
    complaints: "",
    logs: [
        { time: 31, log: "ADD GAP" },
        { time: 41, log: "LACK OF PROGRESS" },
        { time: 59, log: "ADD GAP" },
        { time: 265, "SECTION COMPLETE" },
        ...
    ],
    logsUndone: [
        { time: 36, log: "ADD GAP", timeUndone: 39 },
        ...
    ],
    originalValues: {
        teamname: "m++",
        section1: {
            gaps: 3,
            ramps: 5
        },
        section2: {
            tiles: 8,
            ...
        },
        ...
    }
}
```

Additional notes:
* time
  * kept by saving timestamp when time started
  * when paused: set timestamp to null and add current time to offset
  * currentTime = currentTimestamp - savedTimestamp + timeOffset
  * &rarr; no problems with page reloads (even if time is running)
* last checkpoint
  * everything after last checkpoint is saved as a section which will not be marked as completed
  * scoring elements are still counted and also LoPs (which will be deduced from points for victims)
* signature/confirmation of team captain
  * no signatures, just selecting a radiobutton to confirm
* logs / history - list of possible logs:
  * SECTION COMPLETE
  * LACK OF PROGRESS
  * SKIP SECTION
  * ADD GAP / OBSTACLE / SPEED BUMP / RAMP / INTERSECTION
  * DEL GAP / ...
  * LAST CHECKPOINT
  * a redo-function is not planned
* corrections (in review)
  * edit number of LoPs, number of scoring elements
  * adding sections or deleting sections is not planned
  * old values will be overwritten directly, nevertheless they are stored in an additional object (&rarr; best performance when accessing number of scoring elements)
  * even the teamname can be changed (not yet sure whether text-input or select-input)
* the URL query parameter `cred` can be used to inject credentials for referees
  * the value must be Base64-encoded `username:password` (e.g. `Ym9iYnk6ZnNt`)
  * this can be used to give referees their credentials in form of links or QR-codes

### Evaluation
* runs are submitted to a central system to evaluate all runs and create standings (check out https://github.com/rcjberlin/rcj-server)
* the run-object is modified slightly before submitting so that it can be evaluated easily
* all runs are saved in a local variable so that runs can be re-submitted if something failed

Example object for a submitted run / run to submit (most parts are similiar to run-object above):
```
runSubmit = {
    referee: {
        name: "NL",
        auth: "wbSwFU6tY1c-uczbe"
    },
    competition: "2020-berlin-line",
    arena: "A",
    round: "2",
    teamname: "pi++",
    time_duration: 57,
    time_start: 1554854400,
    time_end: 1558665000,
    scoring: {
        teamStarted: true,
        evacuationPoint: "high",
        sections: [ ... ],
        victims: {
            deadVictimsBeforeAllLivingVictims: 3,
            livingVictims: 2,
            deadVictimsAfterAllLivingVictims: 0
        },
        leftEvacuationZone: false,
        score: 314
    },
    comments: "",
    confirmed: true,
    complaints: "",
    logs: [ ... ],
    logsUndone: [ ... ],
    originalValues: { ... },
    submits: [
        { time: 1143239400, submitStatus: "FAILED" },
        { time: 1517949900, submitStatus: "SUCCESSFUL" }
    ]
}
```

The `referee`-object is not used for authentication.
But the request headers are set to authorize.
Check https://github.com/rcjberlin/rcj-server for more information.

### Issues
If you want to test it locally you may need to make sure that CORS requests are working.
When using Firefox you can set *privacy.file_unique_origin* in about:config to false (see https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS/Errors/CORSRequestNotHttp for more details).

### Next Steps
* replace array with undone logs with tree-structure to offer redo
* styling, css, nicer appearance
* refactoring: better structure, maybe split into multiple files, more code-reuse?
* runHistory: modal with information about each run, possibility to change certain fields and re-submit just this run
* multi-language?
* interfaces for multiple backends, create runSubmit object just before submitting and store a general run object in runHistory
