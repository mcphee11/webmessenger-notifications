# webmessenger-notifications
This repo is an example of how you can enable desktop notifications on new messages with Genesys Cloud WebMessaging widget. This is not a setup guide for WebMessaging and assumes you already have all of this setup in Genesys Cloud. As at the time of wrtiing this if the client has either the "Launcher" closed or is in focus on another page or application there is not "Notification" or "Alert" to let the client know that there is a new message.

This additional code snippet allows for a notification to be pushed from the browser to the OS notification as long as the user has "Allowed" notifications from teh browser.

Depending on the Operating System used the notification will visually render differently here is an example of it loading on my Linux OS

![](/docs/images/popup.png?raw=true)

I have blured out the domain that I have this page running on. If the computer or device also has audio this will also play an audio OS notification tone.

## Step 1

You can either deploy this additonal JavaScript code below the default WebMessenger config snippet on the HTML page inside of a

    <script>...</script>

Tag or load is as a different JS file. The main thing to consider is that it needs to load AFTER the default snippet as it does call back to the WebMessenger SDK via the "Genesys" funciton name. In this example I will load it all on the one simple HTML file.

## Step 2

On line 10 of the "index.html" file you will need to change the section

    ENTER_YOUR_ID

With your actual WebMessenger DeploymentId. If you are NOT in the Australian region for your Genesys Cloud ORG you will also need to change the 

    environment: apse2 

To your region, you should also change the domain of the genesys bootstrap to your region as im in Australia i have mine as .com.au yours may differ. The other option is to replace this row entirly with your default WebMessenger snippet from Genesys Cloud

## Step 3

Once you have loaded the index.html file on your webserver or localhost the page will request access to allow notifications

![](/docs/images/allow.png?raw=true)

Now when you recieve a message from the server side eg agent or bot you will get a notification if the launcher is either not open or the page is not in focus. This will then also open the launcher for the client.

## Explaining the code

The magic happens in this snippet If you would preffer to just add this to an existing page you have running.

```
//Notification Start----->
    Notification.requestPermission().then(function (permission) {
        // If the user accepts, let's create a notification
        if (permission === "granted") {
            console.log("notification permission granted")
        }
    });

    Genesys("subscribe", "MessagingService.messagesReceived", function (o) {
        console.log(o)
        try {
            if (sessionStorage.getItem('gc_widget') === 'false' && o.data.messages[0].direction === 'Outbound') {
                var notification = new Notification("New Message", {tag: "genesys", body: o.data.messages[0].text, icon: "https://dhqbrvplips7x.cloudfront.net/contact-center/5000-5000/img/favicon.ico" });
                Genesys("command", "Messenger.open")
                return
            }
            if(document.hasFocus() === false){
                var notification = new Notification("New Message", {tag: "genesys",  body: o.data.messages[0].text, icon: "https://dhqbrvplips7x.cloudfront.net/contact-center/5000-5000/img/favicon.ico" });
                Genesys("command", "Messenger.open")
            }
        } catch (err) {
            console.error(err)
        }
    });
    Genesys("subscribe", "Conversations.closed", function (o) { sessionStorage.setItem('gc_widget', 'false') });
    Genesys("subscribe", "Conversations.opened", function (o) { sessionStorage.setItem('gc_widget', 'true') });
    //Notification End----->
```

The first part requests permission to allow the browser to leverage the notificaiton service.

The second part uses the WebMessenger SDK to "subscribe" to the messagesReceived and if the messenger is closed and the message is outbound from the server it will trigger a notification as well as open the messenger view. Otherwise if the document does NOT have focus it will trigger a notification and open the messenger.

To track if the messenger is open or closed its tracked via the subscriptions and setting the state in a sessionStorage() object.

You can configure the notificaitons to be different based on the use cases you need to cover for inside the try statements. Its also worth noting that as I have set the "tag: genesys" to be the same everytime this means that when a new notification comes in it writes over the top of the old notification. That way if you recieve 3x messages you dont get 3x notifications shown in the OS notificaiton trya but only the latest one. IF you want them all to show as seprate simply remove this option in the JSON paylod of the obejct. Personally I find it annoying when my OS Notificaiton tray gets filled with the same Application hence I ahve set it this way by default.