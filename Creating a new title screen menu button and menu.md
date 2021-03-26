# Creating your own title screen menu
### Breakdown of the process
All the buttons for creating a title screen menu button are created internally on `sc.TitleScreenButtonGui` by `_createButton(buttonLabel, yPosition, index, onPressCallback, buttonName)`

- `buttonLabel` such that `ig.lang.get("sc.gui.title-screen." + name)`.
- `yPosition` is the y position relative to the bottom of the screen as to position the button.
- `index` is the internal index of the button in `sc.TitleScreenButtonGui.buttonGroup`.
- `onPressCallback` is the method called when the menu button is pressed (Optional).
- `buttonName` used internally for some of the other buttons (Optional).

### Creating your own menu
```javascript

// Create the actual menu, needs to extend sc.BaseMenu
sc.myMenu = sc.BaseMenu.extend({
	buttonGroup: null,
	init() {
		this.parent();
		this.hook.size.x = ig.system.width;
		this.hook.size.y = ig.system.height;

		this.buttonGroup = new sc.ButtonGroup;

		this.doStateTransition("DEFAULT", true);
	},

	// Called to show the menu
	showMenu() {
		this.addObservers();
		// At least one button group is required to exist and be pushed onto the stack otherwise the back button errors for lack of a button group.
		// The button group doesn't necessarily need to be created here if a child creates one.
		sc.menu.buttonInteract.pushButtonGroup(this.buttonGroup); 
		sc.menu.pushBackCallback(this.onBackButtonPress.bind(this));
		sc.menu.moveLeaSprite(0, 0, sc.MENU_LEA_STATE.HIDDEN);
	},

	hideMenu() {
		this.removeObservers();
		sc.menu.moveLeaSprite(0, 0, sc.MENU_LEA_STATE.LARGE);
		sc.menu.buttonInteract.removeButtonGroup(this.buttonGroup);
	},

	// Add/remove the observers from your child UI or call to their respective add/remove observer methods in the following methods

	// Called by showMenu
	addObservers() {
		sc.Model.addObserver(sc.menu, this);
	},

	// Called by hideMenu
	removeObservers() {
		sc.Model.removeObserver(sc.menu, this);
	},

	onBackButtonPress() {
		sc.menu.popBackCallback();
		sc.menu.popMenu();
	},

	// Although empty, is required.
	modelChanged(sender, event, data) {

	},
});

// Create a new sub-menu type by getting the max index of the current list of sub-menus and then adding 1.
// Don't try to set it directly or it might conflict with other mods.
sc.MENU_SUBMENU["MY_MENU"] = Math.max(...Object.values(sc.MENU_SUBMENU)) + 1;

sc.SUB_MENU_INFO[sc.MENU_SUBMENU.MY_MENU] = {
	Clazz: sc.myMenu,
	name: "myMenu" // Identifies the menu and is also used as the text for the top left within the menu
}

// Inject our stuff
sc.TitleScreenButtonGui.inject({
	init() {
		this.parent();

		// Get the last button that was instanced in the parent init, which is also the button with the highest Y position. 
		let highestButton = this.buttons[this.buttons.length - 1];

		this.myMenuButton = this._createButton(
			"myMenuName",
			highestButton.hook.pos.y + highestButton.hook.size.y + 4, // Four for padding
			6, // Or whatever is the highest unused index (Without any other mods it should be 6)
			function() {
				this._enterMyMenu(); // Other things may go here if needed, hence the bind.
			}.bind(this),
			"myMenuName" // 
		);

		this.doStateTransition("DEFAULT", true);
	},

	_enterMyMenu() {
		sc.menu.setDirectMode(true, sc.MENU_SUBMENU.MY_MENU);
		sc.model.enterMenu(true);
	}
});
```

### Adding localisation
The naming for your menu should be as followed:
`assets/data/lang/sc/gui.en_US.json.patch`
```json
{
	"labels": {
		"title-screen": {
			"myMenu": "My Menu"
		},
		"menu": {
			"menu-titles": {
				"myMenu": "My Menu"
			}
		}
	}
}
```

### A more hard-coded method to make a button
Another method to make the button that doesn't depend on the use of `_createButton`. Whether you want to do this is up to you, but it allows for more freedom of control over the style and position of the button, most of the functionality is taken from `_createButton` itself.

In the body of `sc.TitleScreenButtonGui.init`:
```javascript
this.parent();

let highestButton = this.buttons[this.buttons.length - 1];

let myButton = new sc.ButtonGui(
	ig.lang.get("sc.gui.title-screen.mods"), // Localisation text path
	sc.BUTTON_DEFAULT_WIDTH, // Button width
);

myButton.setPos(highestButton.hook.pos.x, highestButton.hook.size.y + highestButton.hook.size.y + 4);
myButton.hook.transitions = highestButton.transitions; // Copy the transitions for consistency's sake
myButton.onButtonPress = this._enterMyMenu;
myButton.doStateTransition("DEFAULT", true);
myButton.setAlign(ig.GUI_ALIGN_X.LEFT, ig.GUI_ALIGN_Y.BOTTOM);
this.buttonGroup.addFocusGui(myButton, 0, 6);
this.addChildGui(myButton);
this.namedButtons["myMenu"] = myButton;
this.buttons.push(myButton);

this.doStateTransition("DEFAULT", true);
```