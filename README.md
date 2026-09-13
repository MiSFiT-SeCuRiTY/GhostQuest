# 😈 GhostQuest

A concise, step-by-step guide showing how to **complete Discord quests without owning the game or watching the required video**, using safe, reproducible methods for testing, research, and automation.

> [!NOTE]
> This does not works in browser for quests which require you to play a game! Use the [desktop app](https://discord.com/download) to complete those.

# 👻 HOW TO USE THE SCRIPT ?
- Allow Inspect Permissions for Github.
- Run %appdata%\discord and Open `settings.json` file in notepad.
- Put a `Comma , ` where code is finishing in the last line and paste
```"DANGEROUS_ENABLE_DEVTOOLS_ONLY_ENABLE_IF_YOU_KNOW_WHAT_YOURE_DOING": true,```
and save the file.
- Accept a quest under the Quests tab.
- Press <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>I</kbd> to open DevTools.
- Go to the `Console` tab.
- Paste the following code and hit enter:

	<summary>😎 COPY THE CODE FROM BELOW 😎</summary>
```
delete window.$;
let wpRequire = webpackChunkdiscord_app.push([[Symbol()], {}, r => r]);
webpackChunkdiscord_app.pop();

let QuestsStore = Object.values(wpRequire.c).find(x => x?.exports?.A?.__proto__?.getQuest)?.exports?.A;
let ChannelStore = Object.values(wpRequire.c).find(x => x?.exports?.A?.__proto__?.getAllThreadsForParent)?.exports?.A;
let GuildChannelStore = Object.values(wpRequire.c).find(x => x?.exports?.Ay?.getSFWDefaultChannel)?.exports?.Ay;
let api = Object.values(wpRequire.c).find(x => x?.exports?.Bo?.get)?.exports?.Bo;

const supportedTasks = ["WATCH_VIDEO", "PLAY_ON_DESKTOP", "STREAM_ON_DESKTOP", "PLAY_ACTIVITY", "WATCH_VIDEO_ON_MOBILE"];
let quests = [...QuestsStore.quests.values()].filter(x => x.userStatus?.enrolledAt && !x.userStatus?.completedAt && new Date(x.config.expiresAt).getTime() > Date.now() && supportedTasks.find(y => Object.keys((x.config.taskConfig ?? x.config.taskConfigV2)?.tasks ?? {}).includes(y)));

if (quests.length === 0) {
    console.log("You don't have any uncompleted quests!");
} else {
    let doJob = async function() {
        const quest = quests.pop();
        if (!quest) return;

        const questName = quest.config.messages?.questName ?? "Discord Quest";
        const taskConfig = quest.config.taskConfig ?? quest.config.taskConfigV2;
        const taskName = supportedTasks.find(x => taskConfig?.tasks?.[x] != null);
        
        if (!taskName) {
            console.log("Unsupported task type, skipping...");
            return doJob();
        }

        const secondsNeeded = taskConfig.tasks[taskName].target;
        let secondsDone = quest.userStatus?.progress?.[taskName]?.value ?? 0;

        console.log(`Starting ${questName} (${taskName})... Target: ${secondsNeeded}s`);

        if (taskName === "WATCH_VIDEO" || taskName === "WATCH_VIDEO_ON_MOBILE") {
            const speed = 7;
            while (secondsDone < secondsNeeded) {
                await new Promise(resolve => setTimeout(resolve, speed * 1000));
                secondsDone = Math.min(secondsNeeded, secondsDone + speed);
                
                try {
                    const res = await api.post({
                        url: `/quests/${quest.id}/video-progress`, 
                        body: { timestamp: Math.min(secondsNeeded, secondsDone + Math.random()) }
                    });
                    console.log(`Video Progress: ${secondsDone}/${secondsNeeded}`);
                    if (res.body?.completed_at) break;
                } catch (e) {
                    console.error("Video heartbeat failed, retrying...", e);
                }
            }
            console.log(`${questName} completed!`);
            doJob();
        } else {
            // Direct API Heartbeat mode for game/stream/activity tasks
            const channelId = ChannelStore?.getSortedPrivateChannels?.()?.[0]?.id ?? 
                              Object.values(GuildChannelStore?.getAllGuilds?.() ?? {}).find(x => x != null && x.VOCAL?.length > 0)?.VOCAL[0]?.channel?.id ?? 
                              "0";
            const streamKey = `call:${channelId}:1`;

            while (secondsDone < secondsNeeded) {
                await new Promise(resolve => setTimeout(resolve, 20 * 1000));
                
                try {
                    const res = await api.post({
                        url: `/quests/${quest.id}/heartbeat`, 
                        body: { stream_key: streamKey, terminal: false }
                    });
                    
                    const progress = res.body?.progress?.[taskName]?.value ?? (secondsDone + 20);
                    secondsDone = Math.min(secondsNeeded, progress);
                    console.log(`Quest Progress: ${secondsDone}/${secondsNeeded}`);

                    if (secondsDone >= secondsNeeded) {
                        await api.post({
                            url: `/quests/${quest.id}/heartbeat`, 
                            body: { stream_key: streamKey, terminal: true }
                        });
                        break;
                    }
                } catch (err) {
                    console.error("Heartbeat error, retrying...", err);
                }
            }
            console.log(`${questName} completed!`);
            doJob();
        }
    };
    doJob();
}
```
- (If you're unable to paste into the console, you might have to type `allow pasting` and hit enter)
- Follow the printed instructions depending on what type of quest you have.
     - If your quest says to "play" the game or watch a video, you can just wait and do nothing
     - If your quest says to "stream" the game, join a vc with a friend or alt and stream any window 
- Wait a bit for it to complete the quest
- You can now claim the reward!

You can track the progress by looking at the `Quest progress:` prints in the Console tab, or by looking at the progress bar in the quests tab.

## FAQ

**Q: Running the script does nothing besides printing "undefined", and makes chat messages not go through**

A: This is a random bug with opening devtools, where all http requests break for a few minutes. It's not the script's fault. Either wait and try again, or restart discord and try again.

**Q: Can I get banned for using this?**

A: There is always a risk, though so far nobody has been banned for this or other similar things like client mods.


**Q: Ctrl + Shift + I doesn't work**

A: Either download the [ptb client](https://discord.com/api/downloads/distributions/app/installers/latest?channel=ptb&platform=win&arch=x64), or use [this](https://www.reddit.com/r/discordapp/comments/sc61n3/comment/hu4fw5x/) to enable DevTools on stable.


**Q: Ctrl + Shift + I takes a screenshot**

A: Disable the keybind in your AMD Radeon app.


**Q: I get a syntax error/unexpected token error**

A: Make sure your browser isn't auto-translating this website before copying the script. Turn off any translator extensions and try again.


**Q: I'm on Vesktop but it tells me that I'm using a browser**

A: Vesktop is not a true desktop client, it's a fancy browser wrapper. Download the actual desktop app instead.


**Q: I get a different error**

A: Make sure you're copy/pasting the script correctly and that you've have done all the steps.


**Q: Can I complete expired quests with this?**

A: No, there is no way to do that.


**Q: Can you make the script auto accept the quest/reward?**

A: No. Both of those actions may show a captcha, so automating them is not a good idea. Just do the two clicks yourself.


**Q: Can you make this a Vencord plugin?**

A: No. The script sometimes requires immediate updates for Discord's changes, and Vencord's update cycle and code review would be too slow for that. There are some Vencord forks which have implemented this script or their own quest completers if you really want one.


**Q: Can you upload the standalone script to a repo and make this gist's code a one line fetch()?**

A: No. Doing that would put you at risk because I (or someone in my account) could change the underlying code to be malicious at any time, then forcepush it away later, and you'd never know.


## HOW TO ALLOW PASTING IN DISCORD DEVTOOLS ? 
<img width="800" height="749" alt="Image" src="https://github.com/user-attachments/assets/c7181725-0d6b-40b3-9abc-596ff8dbee38" />

## 🎥 VIDEO TUTORIAL 📽️
https://github.com/user-attachments/assets/00d8e153-44c4-4cce-99db-fd2c96ede881

## THANKS FOR VISTING THIS PAGE :)
