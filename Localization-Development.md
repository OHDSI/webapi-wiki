# I18n Resources

Localization files are stored in the resources directory: https://github.com/OHDSI/WebAPI/tree/master/src/main/resources/i18n. 

Localization files are JSON with naming convention `messages_{lang}.json`, where `{lang}` is two-letters (ISO 639-1) language code. Translated lines are organized as a tree, so each localized message has a unique path. The client-side application requests a localization file from server (language is user's locale by default, or manually choosen by user from the dropdown on the page). If the server finds requested language in the resources, it sends it to the browser, and if not - sends the default English. The client-side application unwraps the localization file to an object and uses it to render text lines. See for example https://github.com/OHDSI/Atlas/blob/master/js/pages/home/home.html. Calls like `ko.i18n('home.welcome', 'Welcome to ATLAS.')` are i18n calls with a path to a message as first parameter and the default value as second parameter.

# Additional I18n 

Let's assume we want to add a japanese i18n to Atlas/WebAPI.

First, we need a localization file. The easiest way is to create a copy of `messages_en.json`, rename it to `messages_jp.json` - and translate every message in it. 

Place `messages_jp.json` in `src/main/resources/i18n` among other localization files, and edit `locales.json` in the same directory: add `{"code": "jp", "name": "日本語"}`. After that, you can build WebAPI with Maven and run the resuting WebAPI.war in Tomcat (see https://github.com/OHDSI/WebAPI/wiki/WebAPI-Installation-Guide). And if you build and run client-side Altas (see https://github.com/OHDSI/Atlas/wiki/Atlas-Setup-Guide), there should be 日本語 in the upper dropdown menu.

# Contribute Your Localization

To contribute your localization please create a pull-request to WebAPI master branch. PR should contain two aforementioned files:
1. `messages_{lang-code}.json` - new file;
2. `locales.json` - with new localization section `{"code": "{lang-code}", "name": "{lang-name}"}`

