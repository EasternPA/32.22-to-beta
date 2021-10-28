## Background

While waiting for Tesla to roll out Full Self Driving (FSD) Beta to my car, I was constantly refreshing web pages from websites like TeslaScope that showed how many cars were on each firmware revision and the prior version that the current version replaced. TeslaScope is also kind enough to show fairly detailed configurations of the vehicles receiving the updated firmware.

### 32.22 to Beta

With a little investigation, I saw that most vehicles that received Beta came from 2021.32.22 and 2021.32.25. All of the vehicles on 2021.32.25 came from 2021.32.22 but most of the vehicles that received Beta were directly upgraded from 2021.32.22. For my specific situation, I needed to watch the number of vehicles still running 2021.32.22 and I wanted to know when a vehicle basically identical to mine (Model 3 Long Range Dual Motor aka AWD) was upgraded from 2021.32.22 to FSD Beta 2021.36.5.3 aka 10.3.1.

### Node-RED

Node-RED is a visual programming tool that greatly simplifies the act of assembling small, simple scripts. Node-RED also includes numerous modules (called Nodes) and integrations with external tools and sites. Node-RED enables rapid prototyping of functional code by dragging and dropping nodes onto the design space and allows for customization of the code using a variety of programming or scripting languages. A collection of Nodes that interoperate with one another on a single tab in the editor is called a flow. I have exported the flow that I built and published it here as json code.

### How to use it

Clone the repository, fire up Node-RED, click the 3-bar "hamburger" menu in the upper right corner, and click on Import. Navigate to the location where you cloned the repo and import the file. Open the debug window, click on the red Deploy button in the upper right corner, and you will see at least the first of the two example lines shown below in the Output section. You will see the number of vehicles running 2021.32.22 and any Model 3 LR AWDs that have recently been upgraded from 2021.32.22 to 2021.36.5.3.

### How it works

There are two primary purposes of the flow. First is to count the number of vehicles reported by the TeslaScope website as currently running 2021.32.22 -- the primary version that is ultimately upgraded to FSD Beta. Second, I wanted to receive a notification whenever a vehicle that is effectively identical to mine is upgraded from 2021.32.22 to 2021.36.5.3.

#### Vehicles Running 2021.32.22

The first function requires ingesting TeslaScope's detail page for the 2021.32.22 firmware, found here: https://teslascope.com/teslapedia/software/2021.32.22

With the page ingested, I used an html parser node to pull in the html contents of that page. I then used a Change node to only retain item 76 from the fetched page. That contains a sentence listing the number of vehicles currently running this version. I then split that content into an array. Next, as I traverse the array, I convert each entry from a text string into a numeric value. If the resulting numeric value is greater than 0, I convert the value back to a string and append the text " on 32.22" to the end and print it out. Finally, I pass the payload through a filter node (previously called Replace By Exception or RBE). This ensures that the resulting message will be passed only if it has not changed. Then the payload is passed to the debug node, which dumps the payload to the debug window.

#### Any Model 3 LR AWD moving from 2021.32.22 to 2021.36.5.3

It is important to note that the number of vehicles running 2021.32.22 is not enough to tell the whole story. That could be on any model Tesla. The second function in this flow requires ingesting TeslaScope's detail page for the 2021.36.5.3 firmware, found here https://teslascope.com/teslapedia/software/2021.36.5.3

With the page ingested, I used an html parser node to pull in the html contents. Then, I passed those contents on to a change node that only retained object 174 on the page -- which is the table showing the detail of all vehicles that received that update. With the table captured, I ran it through a split node to break it up into lines, a function node that trims leading and trailing white space from all cells in each row, a join node that converts the trimmed text into an array, a switch node that only retains records where the first field matches 2021.32.22, and another switch node that only retains records where the second field matches "Model 3 AWD LR". Once the records not pertaining to cars like mine being upgraded to FSD Beta are filtered out, I send the remaining records to a different debug node which, again, sends the output to the debug window.

### Output

The two functions in this flow dump very simple output. First, if the number of vehicles running 2021.32.22 has changed since the last time it was reported, the function will dump the new value (as shown in this example)

`1336 on 32.22`

Second, any time a Model 3 AWD LR is upgraded from 2021.32.22 to 2021.36.5.3, the second funtion will dump the details of that vehicle as a printed array, for example

`2021.32.22,Model 3 AWD LR,3.0,United States,1 day ago`

The array fields are Prior Firmware Version, Vehicle Model and Trim Level Description, Autopilot Hardware Version, Country, and time since the update. The output is very basic and is not pretty at all.

### Notes

If your car moves from 2021.32.22 to 2021.32.25 instead of 2021.36.5.3, you may easily modify the first switch node in the second function so that it looks for that firmware version instead of the one my car was running. Also, once 2021.36.5.3 gets replaced by a newer version of FSD Beta, you will want to modify the end of the URL that the second function ingests to match the newest firmware being rolled out.

Looking at the trigger node that kicks off both functions, I configured the "Go" button to repeat at the top of every hour between 4am and 8pm since I knew I would only react to early indications of movement during those hours. There's no point in querying TeslaScope's web page more frequently than that and there's no point in querying their site if you're not planning to take any action as a result of any notification you may receive.

It is important to be respectful of other peoples' resources here.

Finally, I terminated the functions with a print to the debug window, but it is up to you to do something different with the output. You may wish to trigger a pop-up on your phone, send yourself a text message, or post a message to or send a private message via Twitter. The debug window is just a dummy output that is useful for functional test but is not very useful beyond that.

### License

This software is published under the Apache 2.0 license. Do whatever you please with it, but respect TeslaScope's wishes if they ask you to reduce the refresh frequency.
