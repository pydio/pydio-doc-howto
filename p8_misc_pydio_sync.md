<div style="background-color: #fbe9b7;font-size: 14px;">
<span style="background-color: #fae4a6;padding: 10px;">WARNING</span>
<span style="padding: 10px;display: inline-block;">This article is for Pydio 8 (PHP). Time to move to <a href="https://pydio.com/en/docs/administration-guides">Pydio Cells</a>!</span>
</div>

### How to use the pydio Sync client

#### Setting & Using the pydio sync client

#### 1. FOR WINDOWS, MAC & LINUX(graphical environnement)
Now launch the app click on getting started and you should have the following screen.

[:image-popup:/sync/sync_connection_pydio_SYNC.png]

+ **HTTP/HTTPS** : You should always choose HTTPS but if you didnt configure security you can use HTTP.
+ **URL** : Its your Pydio's server URL.
+ **Login** : Your Pydio Login.
+ **Password** : Your Pydio's User Password.

[:image-popup:/sync/sync_step1_pydio_SYNC.png]

Then choose the workspace that you want to be Synced.(You can also choose if you only want a subfolder of this workspace).

[:image-popup:/sync/sync_step2_pydio_SYNC.png]

After that you will have to choose a destination, meaning the folder on your local device that you want to be synced with Pydio.

[:image-popup:/sync/sync_step3_pydio_SYNC.png]

This screen shows you a global view of your syncronization, and also offers you more **advanced parameters** ( see below ).

[:image-popup:/sync/sync_adv_pydio_SYNC.png]


> The settings are the same with **Windows** and **Linux**

#### Troubleshooting

You can find the files and therefore the log files of the pydio sync client here:

Win: `C:\Users\<USER>\AppData\Roaming\Pydio`

Mac: `~/Library/Application\ Support/Pydio`
Full path : `/Users/username/Library/Application Support/Pydio`
