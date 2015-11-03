### Disabling actions on a per-user/per-group basis
The simplest way to disable an action in the interface is to use the “role”-based action management in the Settings Panel. For either a USER, a GROUP or a ROLE, you will be able  to restrict some actions. At the end of the day, a user will see the “sum” of all its groups, roles, and personal settings parameters.

Go to the “Actions” tab the object you are editing, and there you’ll have to first choose a plugin, then an action provided by this plugin, and then one or more workspaces to which this setting will apply. Then click on “Add action” to be sure that it appears on the list below, which is the list of the disabled actions.

[:image-popup:miscellaneous/Fine-tune_the_actions_in_the_gui/screenshot-2013-06-03-at-15-14-59.png]

### Going further: inside the plugins
If you want to customize the action at a finer level, like deciding in which part of the GUI an action button will appear, you can manually edit the XML file of the according plugin. Actions are generally described inside the manifest.xml file, but in some case, for intensive action providers, a dedicated xxActions.xml file can be found.

NOTE : The XML files are cached by the application, thus your changed will not appear until you “empty the plugins cache”, ie. remove the files data/cache/plugins_*.ser. If you are testing incrementaly, you can disable this cache by setting AJXP_SKIP_CACHE to true inside the conf/bootstrap_context.php file.

Anatomy of an action


    <action name="download" fileDefault="false">
    <gui text="88" title="88" src="download_manager.png" iconClass="icon-download-alt" accessKey="download_access_key" hasAccessKey="true">
    <context selection="true" dir="" recycle="false"
    actionBar="true" contextMenu="true" infoPanel="true"
    actionBarGroup="get,inline">
    </context>
    <selectionContext dir="true" file="true" recycle="false" unique="false"/></gui>
    <rightsContext noUser="true" userLogged="only" read="true" write="false" adminOnly=""/>
    <processing>

    .......

The XML elements are described below

+ gui: main parameters of the action display , icon & text label (refering to the i18n files)
+ context: in which global context this action appear. Here you can select if it’s in a right-click (contextMenu), in the top bar (actionBar), or in any action bar group.
+ selectionContext : if context had the “selection” attribute set to true, decide how the action will appeared based on the current selection nature (is it a file or a folder, is it a multiple selection, etc…)
+ rightsContext : decide how the action appears depending on the current user state and access rights.