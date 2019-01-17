# Slack Full Black Theme

Fork of [Nockiro/slack-black-theme](https://github.com/Nockiro/slack-black-theme) with full black sidebar and fixed white text.  
You can use the original CSS url if you want; all the changes are in the snippet below.

# Preview

![Screenshot](https://i.postimg.cc/43PNJZP0/2019-01-09-17-15-09.png)

# Installing into Slack

Find your Slack's application directory.

* Windows: `%homepath%\AppData\Local\slack`
* Mac: `/Applications/Slack.app/Contents/`
* Linux: `/usr/lib/slack/` (Debian-based)
* Linux (Flatpak): `/var/lib/flatpak/app/com.slack.Slack/current/active/files/extra/lib/slack/resources/app.asar.unpacked/src/static/ssb-interop.js`

Open up the most recent version (e.g. `app-2.5.1`) then open
`resources\app.asar.unpacked\src\static\index.js`

For versions after and including `3.0.0` the same code must be added to the following file
`resources\app.asar.unpacked\src\static\ssb-interop.js`

At the very bottom, add

```js
// First make sure the wrapper app is loaded
document.addEventListener("DOMContentLoaded", function() {

   // Then get its webviews
   let webviews = document.querySelectorAll(".TeamView webview");

   // Fetch our CSS in parallel ahead of time
   const cssPath = 'https://raw.githubusercontent.com/paualberto/slack-full-black-theme/master/custom.css';
   let cssPromise = fetch(cssPath).then(response => response.text());

   let customCustomCSS = `
   :root {
      /* Modify these to change your theme colors: */
      --primary: #09F;
      --text: #CCC;
      --text-bright: #FFF;
      --background: #080808;
      --background-elevated: #080808;
   } 

    `

   // Insert a style tag into the wrapper view
   cssPromise.then(css => {
      let s = document.createElement('style');
      s.type = 'text/css';
      s.innerHTML = css + customCustomCSS;
      document.head.appendChild(s);
   });

   // Wait for each webview to load
   webviews.forEach(webview => {
      webview.addEventListener('ipc-message', message => {
         if (message.channel == 'didFinishLoading')
            // Finally add the CSS into the webview
            cssPromise.then(css => {
               let script = `
                     let s = document.createElement('style');
                     s.type = 'text/css';
                     s.id = 'slack-custom-css';
                     s.innerHTML = \`${css + customCustomCSS}\`;
                     document.head.appendChild(s);
                     `
               webview.executeJavaScript(script);
            })
      });
   });
});
```

Notice that you can edit any of the theme colors using the custom CSS (for
the already-custom theme.) Also, you can put any CSS URL you want here,
so you don't necessarily need to create an entire fork to change some small styles.

That's it! Restart Slack and see how well it works.

NB: You'll have to do this every time Slack updates.

## Coloring people/channel/conversations

One of the most frustrating things about Slack is the lack of visual emphasis on key conversations beyond a long list of alphabetically ordered favorites. If you want to color a conversation that is easy to do. Using the browser developer tools accessible inside Slack (see section right below) you can find out what CSS to target, but the result is that you basically target the "aria-label" attributes which happen to contain your people and channel conversation names. 

So if you have a channel called "*apj*-sa" then you target and style it like this with additional CSS:

```
   a[aria-label^="apj-sa"]
   {
        background: #4d0000  !important;
        text-transform: uppercase  !important;
        letter-spacing: 2px !important;
        text-shadow: 1px 1px white;
    }

```

You can also target multiple people or conversations at once, i.e. to target conversations with takuya, Rubs and Yuan Li use this css:

```
   a[aria-label^="takuya"],
   a[aria-label^="Yuan Li"],
   a[aria-label^="Rubs"]
   {
        background: #4d0000  !important;
        text-transform: uppercase  !important;
        letter-spacing: 2px !important;
        text-shadow: 1px 1px white;

    }

```

![Screenshot of people/channels/conversations colored](https://user-images.githubusercontent.com/1035157/45020404-39f64000-b072-11e8-981b-e02b5a582faa.png)



# Development

`git clone` the project and `cd` into it.

Change the CSS URL to `const cssPath = 'http://localhost:8080/custom.css';`

Run a static webserver of some sort on port 8080:

```bash
npm install -g static
static .
```

In addition to running the required modifications, you will likely want to add auto-reloading:

```js
const cssPath = 'http://localhost:8080/custom.css';
const localCssPath = '/Users/bryankeller/Code/slack-black-theme/custom.css';

window.reloadCss = function() {
   const webviews = document.querySelectorAll(".TeamView webview");
   fetch(cssPath + '?zz=' + Date.now(), {cache: "no-store"}) // qs hack to prevent cache
      .then(response => response.text())
      .then(css => {
         console.log(css.slice(0,50));
         webviews.forEach(webview =>
            webview.executeJavaScript(`
               (function() {
                  let styleElement = document.querySelector('style#slack-custom-css');
                  styleElement.innerHTML = \`${css}\`;
               })();
            `)
         )
      });
};

fs.watchFile(localCssPath, reloadCss);
```

Instead of launching Slack normally, you'll need to enable developer mode to be able to inspect things.

* Mac: `export SLACK_DEVELOPER_MENU=true; open -a /Applications/Slack.app`

* Linux: `export SLACK_DEVELOPER_MENU=true && /usr/bin/slack`

* Windows (PowerShell):

```powershell
# Set environment variable
[System.Environment]::SetEnvironmentVariable('SLACK_DEVELOPER_MENU', 'true', 'Process')

# Launch Slack (replace x.y.z with the latest version)
& $env:LOCALAPPDATA\slack\app-x.y.z\slack.exe

# Open developer console by pressing: Ctrl + Alt + I
```

# License

Apache 2.0
