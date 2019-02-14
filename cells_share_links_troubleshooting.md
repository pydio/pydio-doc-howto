### Share content using public links

We are going to see how you can share with a public link and micro-manage it's usage for the users.

### Creating a share

To create a share just right click on the file that you want to share and select the **share** option.

[:image-popup:/cells/create_share.png]

and then enable your share by clicking on the public share slider.

### Customize your share

Now that the share is created you can customize it to limit for instance the number of downloads available and so on.

We will go through each category(tabs Link/Access/Visibility) and explain for what each field stands for.

[:image-popup:/cells/create_share.png]


*Note : When you add an option to your public link and even the first time at it's creation do not forget to click on the save button*

[:image-popup:/cells/create_share_link.png]

- **Link** : in the link table you manage everything about the link
    - **Enable public link** : pretty straightforward.
    - **Public link** : the public link that you have to share, you can see below it that you have buttons, the 1st one allows you to copy it in on click, the 2nd one is to share it to Cells Users and/or via mail and lastly the 3 enables you to modify the link itself meaning for instance that you can have (`http://<your pydio>/public/<you can change this part>`).
    - **Labels and permissions** : what users are able to do with the content of your share.


[:image-popup:/cells/create_share_access.png]

- **Access** : you can manage the access to your link and set limitations such as download, expiration date.
- **Password** : add a password to access the content of this link (when saved you will be able to change to password to another on the fly).
- **Will expire on** : you can set an expiration date for the link and therefore limitate for instance it's access to 1 day or couple.
- **Allowed downloads** : you can limit the amount of download of it's content for the users using the link.
- **Layout** : you have to options either you display a preview of the content or there will be only a download button for the end user.

[:image-popup:/cells/create_share_visibility.png]

- **Visibility** : you have the ability to select which user/group has the ability to see and/or modify your public link inside Cells, you have the default options You & Root Group or you can add a specific user. (it will be displayed inside the My shares menu)

You can find the My shares menu right here :

[:image-popup:/cells/menu_ui.png]
[:image-popup:/cells/my_shares_menu_ui.png]
[:image-popup:/cells/my_shares_menu_ui_public_links.png]
make sure that you have selected the public links category.


### Troubleshoot public links 

If users are getting a blank page or only the cells background it could be that the external role has not the right permissions.

Go to **Identity management > Roles** and edit External Users role.
You will have menu, then choose the **Application Pages** category and Deny access to Home & Settings as seen in the screenshot below :

[:image-popup:/cells/external_users_role.png]




