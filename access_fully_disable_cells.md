Although counter-intuitive, as user-managed collaboration spaces provided by Cells and are at the core of the product, it may be sometime desireable to strictly restrict access to a "read-only" or "upload-only" interface for users. Find out how in this short article.  

First log in as administrator on your Cells Console and toggle `Advanced Parameters` on (top-left menu).

## Disable sharing

- Go to `Application parameters >> Sharing Features`
- Disable all 4 check boxes (you might keep the first enable if necessary)
- Save changes

## Disable Cells and their Creation

### Merge Workspaces and Cells

- Go to `Application parameters >> Main Settings`
- Check the `Merge Workspace and Cells` box
- Save

### Customize parameters via Roles

- Go to `Identity Management >> Roles`
- Edit the Root Group
- Go to the Application Parameters
- Choose `Add for... >> All Workspaces`
- Search for `cells`
- Choose `Let User create new Cells`
- toogle this off
- Add another rule by clicking on the `+` button that is left to the `All Workspaces` Header
- Search for `action.share`, choose `My Shares` and turn it off
- Add another, search for `action.share`, choose `Shares` and turn it off
- Save

You should be good to go (after a hard refresh of the interface)!

