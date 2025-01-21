---
layout: post
title:  "Writing A Visual Studio Code Extension"
date:   2024-10-22 00:19:13 +0100
categories: typescript vscode
---

# Introduction
This week I decided to try writing an extension for [Visual Studio Code](https://code.visualstudio.com). My motivation for this was that I have a lot of experience with [Emacs](https://www.gnu.org/software/emacs/), which is highly extensible, and I wanted to get some more power to extend VSCode when I need to use it.

I chose to develop a simple timer extension, which can be found [here](https://github.com/Fhoughton/VSChrono).

# Step 1: Setup
To set up a VSCode extension you run the following npm commands
```bash
npm install -g yo generator-code
yo code
```

This will start a wizard that will let you configure a new extension. You can then open the folder in the VSCode and press F5 to test it.

# Step 2: Creating A Window
As VSCode is Electron based it's effectively just a very complex browser application. With this we can design our extensions using HTML and Javascript (Typescript in our case).

We can use the WebviewPanel to display a html webpage:
```javascript
export function activate(context: vscode.ExtensionContext) {
    let disposable = vscode.commands.registerCommand('vschrono.display', () => {
        const panel = vscode.window.createWebviewPanel(
            'vschronowindow',
            'VSChrono Tracker',
            vscode.ViewColumn.One,
            {}
        );

        // HTML content for the webview
        panel.webview.html = getWebviewContent();
    });
```

And ```getWebviewContent``` just returns the HTML needed for the window:

```html
function getWebviewContent() {
    return `<!DOCTYPE html>
    <html lang="en">
    <head>
        <title>Time Tracker</title>
    </head>
    <body>
        <h1>Time Tracker</h1>
        <div class="timer" id="timer">00:00:00</div>
        <button onclick="startTimer()">Start</button>
        <button onclick="stopTimer()">Stop</button>
        <button onclick="resetTimer()">Reset</button>
    </body>
    </html>`;
}
```

# Step 3: Creating A Basic Timer
Using some simple JS we can create basic timer functions for the buttons:
```javascript
function startTimer() {
    if (!timerInterval) {
        timerInterval = setInterval(() => {
            seconds++;
            document.getElementById('timer').textContent = new Date(seconds * 1000).toISOString().substr(11, 8);
        }, 1000);
    }
}

function stopTimer() {
    clearInterval(timerInterval);
    timerInterval = null;
}

function resetTimer() {
    stopTimer();
    seconds = 0;
    document.getElementById('timer').textContent = '00:00:00';
}
```

But there's an issue!

***To save memory VSCode destroys Webview's when the tab is switched from!***

# Step 4: Saving State
To save state we have to access the VSCode API, saving it on closure:
```javascript
const vscode = acquireVsCodeApi(); // Acquire VSCode api for handling state

function saveState() {
    vscode.setState({
        seconds: seconds,
        isRunning: isRunning
    });
}
```

And restoring it when the tab is reopened:
```javascript
// Restore the state if available as VSCode destroys webviews on tab out
const previousState = vscode.getState();
if (previousState) {
    seconds = previousState.seconds || 0;
    isRunning = previousState.isRunning || false;
    updateTimerDisplay();

    if (isRunning) {
        startTimer(true); // Automatically resume the timer if it was running
    }
}
```

But we still have one final issue as when we close the tab and come back the timer is still where it was when we left! To fix it we need to track when we left and came back.

# Step 5: Saving the DateTime
We simply have to change the code to use a DateTime, and save it and restore it with the state, calculating the time change to update the timer when it's not focused:

```javascript
// Restore the state if available as VSCode destroys webviews on tab out
const previousState = vscode.getState();
if (previousState) {
    startTime = previousState.startTime ? new Date(previousState.startTime) : null;
    elapsedTime = previousState.elapsedTime || 0;
    isRunning = previousState.isRunning || false;
    updateTimerDisplay();

    if (isRunning) {
        startTimer(true); // Automatically resume the timer if it was running
    }
}

function saveState() {
    vscode.setState({
        startTime: startTime ? startTime.toISOString() : null,
        elapsedTime: elapsedTime,
        isRunning: isRunning
    });
}
```

# Conclusion
I found writing a VSCode extension to be an interesting experience. I found working with the VSCode API to save state slightly clunky, and would've preferred having a simple flag to decide if the tab should be destroyed on change (perhaps this is possible and I missed it). The extensions definitely feel less powerful than Emacs packages, and not having an easy way to write code changes like with .emacs.d is a shame, but I appreciate that there is some extensibility anyway.

In the future I think I'd like to write my packages in PureScript instead of Typescript if possible, as it has a nicer syntax, leads to cleaner code and can express ideas better, for example the saveState function above might be:
```haskell
saveState :: IO ()
saveState = vscode.setState(
    startTime: startTime ? startTime.toISOString() : null
    elapsedTime: elapsedTime
    isRunning: isRunning
)
```