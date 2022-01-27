![](https://img.shields.io/badge/Foundry-v9-informational)
![Forge Installs](https://img.shields.io/badge/dynamic/json?label=Forge%20Installs&query=package.installs&suffix=%25&url=https%3A%2F%2Fforge-vtt.com%2Fapi%2Fbazaar%2Fpackage%2Fmonarchk&colorB=4aa94a)
![Latest Release Download Count](https://img.shields.io/github/downloads/zeel01/monarch/latest/monarch.zip) 

# 🦋 Monarch - A card UI fit for royalty.

**Monarch is currently in beta. There are some rough edges and incomplete features. Feel free to open [issues](https://github.com/zeel01/monarch/issues) if you find problems.**

Monarch is an enhanced card UI for Foundry VTT v9+ which provides replacements for card, hand, deck, and pile sheets.

![A "light" deck, with two open hands and two open individual cards.](examples/monarch.png)

## Features

Monarch includes apps for hands, decks, cards, and piles. It also remembers the positions of open card sheets, and restores them when the application is reloaded. This allows players and GMs to keep their hands, and any important card sheets, open without them being closed acceidentally.

Monarch supports drag and drop, including moving cards between piles, decks, and hands as well as re-ordering cards in these containers.

### Settings

**Card Height** - You can configure the visual hieght of cards in Monarch applications. This value is in pixels, the default is 200. Adjusting this will change how large cards appear in decks, piles, and hands. *Note: Other clients may need to refresh to see this change*

## 🦋 My Hand

The "My Hand" sheet displays your hand similarly to how you might see it in real life. All of the cards are layed out in a compact "fan" format. You can see more details about each card by hovering your mouse over it. You can also play a card, flip it over, or view its individual sheet.

![Animation showing the hand application.](examples/hand.gif)

## 🦋 Card

The card sheet is a compact view of a single card which shows off the artwork. When you hover your mouse over the card, you will reveal additional information and can edit its properties. There is a button for editing the faces and back of the card as well. There is also a magnifying glass button to pop open the card artwork for a full view.

![Animation showing three single card sheets.](examples/card.gif)

## 🦋 Deck

The deck sheet is a slightly restyled version of the standard deck sheet, which shows the card artwork more clearly, and arranges the cards in a grid rather than rows.

![A deck of standard playing cards using the "light" theme.](examples/deck.png)

## 🦋 Pile 

The pile sheet is very similar to the deck sheet, but with slightly different controls.

![A discard pile app.](examples/pile.png)

## Using Monarch

To use Monarch, open one of the supported documents (card, deck, pile, hand) and click the "⚙️ Sheet" button in the title bar. This will open the sheet config window. You can then choose to set either "This Sheet" for this specific document, or "Default Sheet" for all documents of this type to use the corresponding 🦋 Monarch sheet.

![A standard hand app, with the sheet config window open.](examples/sheet-config.png)

## 🦋 Monarch API

Monarch is designed to be flexible for the various needs of systems and games. It provides an API for adding custom information badges, and custom interactive buttons to the various sheets.

### Technical Details

This section contains technical details about the Monarch API. The next section will give examples and instructions on how to use it.

#### Hooks

Monarch adds the following hooks:

##### `getMonarch{Application}Controls`

This hook fires just before any Monarch sheet renders, the name will depend on which sheet is rendering: `getMonarchHandControls`, `getMonarchDeckControls`, `getMonarchPileControls`, or `getMonarchCardControls`. This hook is used to gather data for constructing information badges and interactive controls on the application.

###### Parameters
| Parameter | Type | Description |
| --------- | ---- | ----------- |
| `monarch` | `FormApplication` | The Monarch application object. |
| `controls` | `Array<CardControl>` | An array of controls to add to the application. These are added to each card individually for piles. |
| `badges` | `Array<CardBadge>` | An array of information badges to add to the application. These are added to each card individually for piles. |
| ~~`appControls`~~ | ~~`Array<AppControl>`~~ | *Not Yet Implmented* ~~An array of controls to add to the application. These are added to the application for interacting with the pile itself.~~ |

#### Data Objects

These objects are used to define certain kinds of dynamic components.

##### `CardBadge`

Data to define an information badge that will display on a card.

| Property | Type | Description |
| -------- | ---- | ----------- |
| `tooltip` | `string` or `Function<string>` | Used as the HTML `title` attribute providing a tooltip describing/labeling this badge. May be a function that returns a string. |
| `text` | `string` or `Function<string>` | The text to display on the badge, may contain HTML. May be a function that returns a string. |
| `class` | `string` or `Function<string>` | The CSS class to apply to the badge. May be a function that returns a string. |
| `hide` | `boolean` of `Function<boolean>` | *Optional*. Whether or not to hide the badge, not including it in the HTML. May be a function that returns a boolean, useful for badges that only display under certain conditions. `false` be default. |

##### `CardControl`

Data to define an interactive contrl that will display on a card.

###### Properties
| Property | Type | Description |
| -------- | ---- | ----------- |
| `tooltip` | `string` or `Function<string>` | Used as the HTML `title` attribute providing a tooltip describing/labeling this control. May be a function that returns a string. |
| `aria` | `string` or `Function<string>` | The ARIA label for the control, used by screen readers to identify it. When omitted, the `tooltip` will be used instead. May be a function that returns a string. |
| `icon` | `string` or `Function<string>` | The classes used to display a font awesome icon (ex: `"fas fa-caret-up"`). May be a function that returns a string. |
| `class` | `string` | A unique CSS class to apply to the control. **IMPORTANT**: This property is required. Monarch used this clas to identify the control in order to attach event listeners. |
| `disabled` | `boolean` or `Function<boolean>` | *Optional*. Whether or not to disable the control. The control will appear grayed out. May be a function that returns a boolean. `false` by default. |
| `onclick` | `Function<void>` | A function that will run when the control is clicked. See blow for parameter information. |
| `controls` | `Array<CardControl>` | *Optional*. An array of controls to be added as a control group. When included, you may omit all other properties except `class`. Instead, each item in this `controls` array should be a complete `CardControl` object. Does not support nesting. |

###### `onclick` Parameters
| Parameter | Type | Description |
| --------- | ---- | ----------- |
| `event` | `MouseEvent` | The mouse event that triggered the click. |
| `card` | `Card` | The card that the control is on. |
| `container` | `Cards` | The Cards object that the card is a member of. |

### API Guide

This section contains a series of brief examples explaining how to use certain sections of the Monarch API.
#### Creating a Custom Badge

As an example we will create an info badge on the hand sheet indicating the number of the currently displayed face.

```js
Hooks.on("getMonarchHandControls", (monarch, controls, badges) => {
	badges.push({
		tooltip: "Current Face",
		text: (card) => `Face: ${card.data.face}`,
		class: "card-face-num"
	});
});
```
We use the `getMonarchHandControls` hook to respond when the hand is being displayed. This hook provieds a number of parameters, the first is the application instance itself, the second is data pertaining to controls (more on those later), and third is the data peraining to badges.

This data is an array of objects, each of which describes a badge. Check the technical details above for more information on the badge object.

In this case, we just use `Array.push` to add a new badge to the `badges` array. This will add a new badge after all the existing ones.

Aside from the `class` property, the other properties of a badge shown above may be either a string or a function. A string is used in this tooltip, because we don't need it to be different for each card in the hand. However, the `text` needs to be different for each card, so we give it a function instead that will return a string later. When this function is called, it will be passed parameters. The first parameter is the `Card` object for the card this text will be displayed on. This function will be called for *each* card in the hand.

Here, we simply have the function return a string created from a template literal. It will always look like "Face: #" where # is the current face shown of the card.

![A hand sheet with a badge showing the current face.](examples/guide-custom-badge.png)

#### Creating a Custom Control

Custom controls are created very similarly to badges, but they have a bit more data assocaiated with them. In this example, we will re-implement the "discard" button available in Monarch.

```js
Hooks.on("getMonarchHandControls", (monarch, controls, badges) => {
	controls.push({
		tooltip: "monarch.label.discard",
		icon: "fas fa-trash",
		class: "discard-card",
		onclick: (event, card) => card.pass(game.cards.getName("Discard"))
	});
});
```
Of note, the `tooltip` here is being given a localization key rather than a plain string. Monarch will always attempt to localize the provided tooltip allowing you to use localization easily.

Controls also have an `icon` property. Like tooltip this can optionally be a function that returns a string. This string should identify a font-awesome icon to use for this control.

The most important thing that sets a control apart from a badge, is the `onclick` property. This must be a function, and it will be called when the button is clicked on the sheet. It will recieve a number of parameters (see technical details above) including the click event and the card that the control was used on.

IMPORTANT: Along with `onclick` you *must* include a unique `class` property for each control. This class is what's used by Monarch to attach the correct event listener to the contol element.

Here, we look up a pile in `game.cards` called "Discard" and pass the card to it. This is an example, in everyday use we would want to do something to prevent errors in case no such pile exists.

![A hand sheet with a custom discard control.](examples/guide-custom-control.png)

#### Custom Control Groups

Above, you may notice that the "discard" button is in a seperate box from the play button. But the standard Monarch discard button when enabled appears in the same box next to the play button. This is because you can also create *groups* of controls. Each control object may contain an array `controls` of control objects to display together in a group.

The control group created by Monarch is called `basic-controls`, if we search through the controls array for this item we can add our control to the same group:

```js
Hooks.on("getMonarchHandControls", (monarch, controls, badges) => {
	controls.find(c => c.class === "basic-controls").controls.push({
		tooltip: "monarch.label.discard",
		icon: "fas fa-trash",
		class: "discard-card",
		onclick: (event, card) => card.pass(game.cards.getName("Discard"))
	});
});
```
Here we used `Array.find` to locate a control that has the class `basic-controls`. We then pushed our control onto its `controls` array.

![A hand sheet with a custom discard control in a group.](examples/guide-custom-control-in-group.png)

If we wanted to create our own control group, we could do so like this:
```js
Hooks.on("getMonarchHandControls", (monarch, controls, badges) => {
	controls.push({
		class: "my-controls",
		controls: [
			{
				tooltip: "monarch.label.discard",
				icon: "fas fa-trash",
				class: "discard-card",
				onclick: (event, card) => card.pass(game.cards.getName("Discard"))
			},
			{
				tooltip: "CARD.Play",
				icon: "fas fa-play",
				class: "play-card",
				onclick: (event, card, pile) => pile.playDialog(card)
			}
		]
	});
});
```
Here we push a new object into `controls`, and that object contains a `class` and its own `controls` array. That array contains two new controls we want to make.

![A hand sheet with a custom control group.](examples/guide-custom-control-group.png)