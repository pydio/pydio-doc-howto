In this HowTo, we take a closer look at the visibility parameters of the shared resources:

- what they are
- how to enable them
- and a few use cases

### Cells ownership

[:image-popup:miscellaneous/visibility/visibility_cells.png]

With Pydio Cells, you can define the **Visibility (2)** of your resource (eg. a cell in an instance) that enables transfering ownership of a resource, using users, groups or even roles:  
A user who has been assigned or has corresponding group or role, can act as if it was her own resource. She therefore has same rights as if she has created the resource herself and can thus modify the cell definition, add users, etc...

This **Visibility (2)** setting is often mistaken for the **Shared with (1)** setting. For the record, the **Shared with (1)** setting defines with whom you share a resource, and what they can do with it (read, download or modify, among others).

### Share links ownership

In the case of a workspace that can be accessed by multiple users, you can choose to display your link to some of them and/or allow them to edit the link settings.

Below is an example of a common workspace (common-files) that can be accessed by two different users, `admin` and `jess`.

In the first couple of screenshots, the setting is disabled (screenshot **1**): user `jess` is not selected. She therefore does not see the share link on the file (Desert 6.png) (screenshot **2**).

_Screenshot #1_

[:image-popup:miscellaneous/visibility/visibility_share_1.png]

_Screenshot #2_

[:image-popup:miscellaneous/visibility/visibility_share_1_disabled.png]

We then add user `jess` to the list of users and give her following privileges (see screenshot **3**):

- **view**: she sees the link in the workspace
- **edit**: she can modify the share settings as if it was her own

You can now notice on screenshot **4** that she has the link listed on her view. She can also **right-click >> edit share**.

_Screenshot #3_

[:image-popup:miscellaneous/visibility/visibility_share_2.png]

_Screenshot #4_

[:image-popup:miscellaneous/visibility/visibility_share_2_enabled.png]

### Use cases

- You have a user that is going on vacation and owns important resources about a given client. Before she leaves, she should give the visibility to another of her co-workers that is then able to access this resource.
