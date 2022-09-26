---
title: 'Dynamic Menu in MAUI'
date: 2022-09-26T10:33:05+10:00
draft: false
tags: ['dotnet', 'c#', 'MAUI', 'Universal App', 'Menu Component']
---

## Context

The requirement here is to be able to add dynamic menu items to a MAUI app. The use case chosen here is of selection of a font from a list of fonts installed on the machine. The list of fonts is shows under a menu item in the menu bar.

More on menu bar from the [Microsoft documentation](https://learn.microsoft.com/en-us/dotnet/maui/user-interface/menubar)

## Approach

Step 1: Mark up

> The component to be used here is the MenuBarItem. This component support binding context of the content page. As part of the page markup, a menu item is added to the page and the "Choose Language" menu item is left blank, so that the menu items can be added after fetching it.

Step 2: Code Behind

> A view model for the page is created with a property for the command that is of type "ICommand". This property is initialized with an action/delegate that takes a single parameter and the logic to handle the menu click. In the page constructor, after the components are initialized, the installed fonts are fetched and looped though to be added to the placeholder menu item. While the menu item is added for each language, the command and commmandparameter attribute is populated, the command attribute is assigned to command property of the view model and language name as commandparameter.

## Source code

Sample code for the Menu Item creation markup. The markup adds a menu bar item and a place holder menu item to render all the language menu items.

```html
<ContentPage.MenuBarItems>
  <MenuBarItem Text="Options">
    <MenuFlyoutSubItem
      x:Name="mnuLanguages"
      Text="Choose Language"
    ></MenuFlyoutSubItem>
  </MenuBarItem>
</ContentPage.MenuBarItems>
```

View model class that is used to bind all the view related data. The below class only shows the command property.

```c#
public class LanguageTypeViewModel
{
    public ICommand ChangeLanguageCommand { get; private set; }

    public LanguageTypeViewModel()
    {
        ChangeLanguageCommand = new Command(async (languageName) => {
            // Logic to handle based on language name
        });
    }
}
```

The below code shows the modified constructor of the content page. The added code creates new Menu item for each installed language and adds it to the language placeholder menu item. It also initializes the command and commandparameter and points it to the command in the view model.

```c#
   public LanguageType()
    {
        InitializeComponent();

        var fonts = new InstalledFontCollection();
        var installedFonts = fonts.Families.ToList();
        var vm = new LanguageTypeViewModel();
        BindingContext = vm;

       installedFonts.ForEach(q =>
        {
            var langMenu = new MenuFlyoutItem
            {
                Text = q.Name,
                Command = vm.ChangeLanguageCommand,
                CommandParameter = q.Name
            };

            mnuLanguages.Add(langMenu);
        });
    }

```
