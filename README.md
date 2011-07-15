## What it does

**upgrademe** allows any WordPress plugin to register its own url for querying about newly available versions
from outside of official WordPress plugins repository. This plugin intelligently integrates with WordPress without changing its sources,
it uses filters API available to all the plugins.

## Usage

### Step 1: integrate with upgrademe

In your plugin (which is not hosted on the official WordPress plugins directory) declare one function:

	function <plugin_name>_upgrademe()
	{
		return 'http://yoursite.com/latest-version.php';
	}

Here <plugin_name> is the name of your main plugin file (without php extension, of course).
For example, if your main plugin file is *superplug.php*, then function name will be:

	function superplug_upgrademe() { ... }

*btw, upgrademe plugin uses itself for auto-update and declares function upgrademe_upgrademe()*

### Step 2: provide version info from your server

On url provided by your xxx_upgrademe function, return json of this structure:

	{
		"new_version":"0.9",
		"url":"https://github.com/meglio/wp-upgrademe",
		"package":"https://github.com/downloads/meglio/wp-upgrademe/upgrademe-0.1.zip",
		"info":{
			"name":"upgrademe",
			"author":"<a href='https://github.com/meglio'>Meglio</a>",
			"author_profile":"https://github.com/meglio",
			"requires":"3.2",
			"tested":"3.2.1",
			"rating":98,
			"num_ratings":680,
			"downloaded":1950,
			"last_upated":"2011-07-15",
			"added":"2011-07-15",
			"homepage":"https://github.com/meglio/wp-upgrademe",
			"tags":{
				"wordpress":"wordpress",
				"upgrade":"upgrade",
				"update":"update",
				"wp":"wp",
				"repository":"repsitory",
				"plugins":"plugins",
				"hacks":"hacks",
				"trucks":"trucks"
			},
			"sections":{
				"description":"Allows auto-upgrade for plugins outside of official WordPress repository",
				"installation":"As simple as 2 steps: extract in plugins folder, activate in administration panel",
				"screenshots":"",
				"changelog":"",
				"faq":""
			}
		}
	}

* All top-level values in this tree are obligatory, upgrademe will ignore your response if it does not contain all 4 of them,
eg "new_version","url", "package" and "info".

* Details in the "info" section are not obligatory, so you can omit it by specifying empty value: *"info":""*,
however it's recommended to provide all values as in the example.

* HTML is allowed in items under "sections".

### Step 3: Test

WordPress caches API requests to their plugins directory for about 12 hours,
so you will need to run this MYSQL query in order to remove cached data about updates available:

	DELETE FROM `wp_options` WHERE option_name='_site_transient_update_plugins'

As you know, in your main plugin php file you are declaring current version of your plugin used,
here is how it's declared in upgrademe plugin:

	<?php
	/*
	Plugin Name: upgrademe
	Description: Allows auto-upgrade for plugins outside of official WordPress repository
	Company: Copyright 2011 Meglio. All rights reserved
	Version: 0.9
	Author: Meglio (Anton Andriyevskyy)
	Plugin URI: https://github.com/meglio/wp-upgrademe
	*/

So, what WordPress does is just compares version number declared in this file (0.9 in case of this examples)
with version returned in your json (1.0 in json example above): "new_version":"1.0",
and if actual version is outdated it will consider your plugin has update available.