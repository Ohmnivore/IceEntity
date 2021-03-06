IceEntity
=========

![IceEntity Demo](http://i.imgur.com/9tQt9rh.png)

A simple framework for managing gameobjects and components in haxeflixel

**Changes:**
----------
  
  **[NEW v0.10.0]**
  
  Added syntax for exposing and requesting classes inside of scripts

  **[NEW v0.9.0]**
  
  Added simple FSM system (sorry no write-up yet)

  **[v0.8.0]**
  
  Added "noreload" option for script elements
  
  Made scripts remove after calling init, unless they have update or destroy logic

  **[v0.7.0]**
  Allow access to instance via a property

  **[v0.6.1]**
  Made live-reloading fully automatic

  **[v0.6.0]**
  Added live-reloading of scripts

  **[v0.5.0]**
  Added hscript integration

  **Earlier:**
  
  [See Changelog](https://github.com/NicoM1/IceEntity/blob/dev/CHANGELOG.md)
  
**Installation**
----------

  **[1]** Run ```haxelib git iceentity https://github.com/NicoM1/IceEntity``` in a terminal with access to git
  
  **[2]** Add ```<haxelib name="iceentity"/>``` to your ```Project.xml``` file, directly under ```<haxelib name="flixel"/>```
  
  **[3]** Get annoyed with missing features, give up;)

**Usage:**
----------

  Call ```add(EntityManager.instance);``` in the create function of your playstate.
  
  Use ```EntityManager.instance.AddEntity();``` to add objects or groups to the manager.
  
  Use ```AddComponent();``` on a class extending entity to add a new component.
  
  ```init()``` will run on the first update cycle, useful for getting and stashing references to other entities.
  
 **Entity Parser:**
----------
As of v0.3.0, IceEntity includes an xml parser, which can build entities from simple xml files. Do note, this is a new feature, and may have bugs (not like the rest of IceEntity doesn't;)). In order to use this system, you must:

**[1]** Create an xml file with this structure:

    <?xml version="1.0" encoding="utf-8" ?>
    <data>
	    <entity tag="myTag" x="0" y="0">
		    <art width="32" height="32" path="assets/images/myimage.png">
			    <animation name="idle" frames="0,1,2,3-6,6-0" framerate="10" looped="true" autorun="true"/>
				//ANIMATIONS: all parameters must be supplied, except "autorun" which mearly starts the animation as soon as it is loaded
				//FRAMES: as of v0.3.1, "frames" can now use hyphens, allowing you to do things such as: "0-10", or "10-0", note that commas are still allowed, and this is valid: "0-10,10-0,0,1,2,3,4,5,6,7,8,9,10"
		    </art>
		    <component type="com.me.MyComponent">
			    <param name="speed" type="int" value="10"/>
		    </component>
			//CHECK NEXT SECTION FOR SCRIPTING INFORMATION (yes, you can write code here!!)
	    </entity>
    </data>
	
**Important information:** 

Parameters for components **must** be in the same order as specified in their constructor, **passing the entity's GID is not required**, it will be auto set. 

Param "name" attributes are not required, but are strongly recommended for organization.

Allowed types for parameters are: **"int" "float" and "bool" anything else will be treated as a plain string, including a lack of the "type" attribute**. Capitalization on param types does not matter. 

**MOST IMPORTANTLY: any component you wish to add in a xml file MUST be referenced somewhere in your basecode, even adding an ```import com.me.MyComponent;``` to your playstate will work. xml parsing will not work without this, as the component will not be compiled.**

**[2]** Call ```EntityManager.instance.BuildFromXML("assets/data/MyEntities.xml");``` in your playstates create function (or wherever really).

**HScript Integration:**
----------

As of v0.5.0, IceEntity has integrated the hscript scripting engine into its entity parser, and entity classes. Note this is a brand new feature, relativly untested, and may not be as efficient as standard components. If you find an issue, or are confused, tweet to me at: [@nico_m__](https://twitter.com/nico_m__), or email me: nico(dot)may99(at)gmail(dot)com. This allows you, the developer, to program without recompiling, or do many other cool things. I'll let you think of the possibilities:). It also means, that with little or no work, modding can be integrated into your game! Here are the steps to getting this to work in you game:

**[1]** In you entity xml file (described above), there are two places you can put scripts. One being inside of an entity declaration, like so:

    <?xml version="1.0" encoding="utf-8" ?>
    <data>
	    <entity tag="myTag" x="0" y="0">
		    <script/>
	    </entity>
    </data>
	
The other is outside of an entity declaration, like so:
	
	<?xml version="1.0" encoding="utf-8" ?>
    <data>
		<script/>
    </data>
	
Before I describe how to actually write a script, let me make sure you know the difference between the two places. A script placed inside an entity declaration has two features that orphan (outside) scripts do not. First: a "owner" variable is automatically created, allowing you to access the entity that the script resides in, and second: they scripts "destroy" function will automatically be called in the event of its parent entity's destruction.

**[2]** In order to write your script, this is the format you must use:

	<script path="" id="-1"> 
	<!--if a path is specified, that is loaded instead of the <text> node-->
	<!--id can be left at -1, to auto-assign, but I recommend setting it to any int greater than 0, to allow referencing this script-->
		<expose name="FlxG" path="flixel.FlxG"/> <!--Gets a static class-->
		<request name="Player"/>	<!--requests a class instance from a global pool-->
		<text>
			//As of v0.10.0, you may now use this syntax instead of the above if you wish
			expose flixel.FlxG;
			request Player;
			
			@init
			{
				i = 10;
			}| <!--close these "functions" with }|-->
			@reload
			{
				<!--described in next section-->
			}|
			@update
			{
				trace(Player.x); 
				trace(owner.x); <!--if this script was inside an entity declaration, you can reference that entity with "owner"-->
				trace(FlxG.camera.x);
				trace(i);
			}|
			@destroy
			{
				Player = null;
			}|
		</text>
	</script>
	
As you can see, the "expose" tag allows the script to gain access to a static class, and reference it as whatever is in the "name" attribute. The "request" tag allows the script to get access to a class instance (or, truthfully, a static class will also work), from the ```ScriptHandler```s global pool. You can add to the pool in your code with ```ScriptHandler.AddModule(name, value);``` Note that this must be done BEFORE you parse the entity file, or you will get a nasty error message.

**[3]** You can think of the lines with "@" as functions, although their variables are **global scope**. Currently, the above 4 are the only possible "functions", you can think of them as: "at(@) (function) do whatever is in these brackets." **Do not add comments between the function name and first curly-bracket**, elsewhere, comments are great. Close these "functions" with the "}" character followed immediately by the "|" character, like so: "}|".

**[4]** The scripting system relies on hscript, which is basically interpreted haxe. Unfortunately, I do not know enough about hscript yet to explain what you can and can't do, but two things to note are: do not use the "var" keyword, just pretend the variables already exist, and do not specify variable types. **EDIT:** Do not use properties in scripts, you must instead just use methods (as far as I can tell...). If you are more experienced in hscript, please submit a pull request with a fuller description:)

**[5]** As a developer, you may not want scripts, specificity mods, to have access to sensitive areas of your game. There are two ways to achieve this. The broad stroke way is to completely disallow access to the expose tag, ensuring scripts have no access to anything unless you specificity add it to the ```ScriptHandler```s modules list. This can be done with: ```ScriptHandler.allowExpose = false;```. The second, more specific way is to "blacklist" classes with ```ScriptHandler.Blacklist("path.to.Class");```, this will warn the user they can not access this package.

**Important:**

Neko, while a nice target for quick testing, does have some quirks. If you are having strange issues with scripts, you may wish to try moving to a CPP build. I'm sorry if this is an annoyance, but the main point of the live-scripting system is to eliminate constant rebuilds, so the extra build time should not be too bad. **Note:** I fixed the main issue I had found with neko, so it appears to work fine now, but I'm leaving the above advice, as other situations may still be affected.

**Live Scripting**
----------

![Live-Scripting Demo](http://i.imgur.com/CkyiKeF.gif)
As of v0.6.0, IceEntity makes use of Openfl 2.0's new live asset reloading system, to allow you to literally code your game **while it is running**. This currently only works for CPP and Neko builds, and only for external script files (ie scripts created inside of your entities.xml file will not be editable at runtime). Here is what you need to do to make use of this new feature:

**[1]** Any scripts you want to be able to edit at runtime must be taken out of your xml file, placed in their own file, and then declared in your script element's "path" attribute.

**[2]** As of v0.10.0, you can expose and request classes inside your scripts, similar to how imports are handled in regular haxe:

    expose flixel.FlxG; //this is identical to the expose tag in xml, except you do not control the name of the class, the last section is used, in this case: "FlxG".
	request MyClass; //this is identical using a request tag in your xml
	
	@init
	{
	
	}|

This also means the live-reloader will update your imports, so you can add things as you need them. This, however, may slow the reloader, if you find that to be the case, you can add:

    <haxedef name="ICE_NO_RELOAD_IMPORTS"/>

to your project.xml file.

**[3]** To help with testing, a new scripting function has been added:

    @reload
	{
		//This runs every time the script is updated, you can use it to edit variables for testing.
		//Note that it is ONLY for testing, you should not use this for any real game code, just for playing with values for speed and so on. THIS NEVER RUNS IN A FINAL BUILD
	}|
	
**[4]** The files you edit while making use of live reloading **are not your main files**. The files you want to edit are in: yourProject\export\windows\ (neko or cpp)\bin\assets\data. **If you wish to use the logic you've created, copy these files back into your main folder when you are done**.

**[5]** Reloading can be costly; you may not wish to reload every script when you only want to tweak a couple. To stop certain scripts from reloading, a "noreload" attribute has been added for script elements:

    <script noreload="true">
	
Also, you may wish to do setup on an entity after the game begins, but not run any actual update logic. This means a script object is created, and updated every frame for no reason. To combat this, automatic script "cleaning" has been added, meaning that if only an "init" function is specified, the script will be deleted after running said "init" function. It is possible that during live-scripting, this behaviour may be annoying, if you wish to add other functions while running. In that case, you can specify:

    <script noclean="true">
	
Note that cleaning **only happens** if the script only had an "init" function at startup, the noclean option is only meant to be used if you really have a need for it, in normal use you should **never** specify this option.

**[6]** As of v0.6.1, there are now two ways of initiating live-reloading, for easier (and more helpful) usage, I recommend the first option:

**[Option 1]** Fully Automatic Live-Scripting:

This is the easier, and more useful method of live-scripting, **however**, it is likely much more taxing on your system. All you need to do to use this system is to add:
```<haxedef name="ICE_LIVE_RELOAD"/>``` to your project.xml file. Thats it!!!!
If you want control over the exact time between reloads, you can add a ```reloaddelay``` attribute to the root element of your entities.xml file.

**[Option 2]** (only sort of) Live-Scripting:

Here is the less user friendly version: Once you've built and opened your game, open a console window and navigate to your main project folder. Now, whenever you wish to see the effect of a change you've made, simply type "lime update [neko/windows]" (where [neko/windows] is whatever target your running your project in). Hint: after you've done this once, you can just press the [up arrow] and then enter to update it again.

**[6] Important Notes**: 

**This doesn't work for scripts specified in the "text" elements of your xml file**

**If using full-auto scripting, make sure to remove ```<haxedef name="ICE_LIVE_RELOAD"/>``` before releasing your game, it is a performance waster, and has no use in a final build.**

For more responsive use I recommend adding ```<haxedef name="FLX_NO_FOCUS_LOST_SCREEN"/>``` to your project.xml, and setting ```FlxG.autoPause``` to ```false```. This way, you do not need to give the game focus to see your changes in action.

As a rule of thumb, any script that you wish to use the "@update" function in should be contained in its own file, not in your xml file. This does two things: keeps your xml file readable, and combats the annoyance of live-reloading not working for internal scripts.
  
**Message System:**
----------

IceEntity now includes a simple message broadcasting system. It is a useful way of quickly sending information or data between objects, without needing to store a reference. Simply call ```SendMessage()``` on an entity, with any info you need to send (explained in the method details). To receive messages, simply override the ```ReceiveMessage()``` function on an entity, and use that as a simple way to do whatever you want with the messages data, no need to manage a complicated event listener setup:)
  
  **Contact/Extra Info:**
  ----------
  
  It should be fairly self explanatory, but if not, you can get in touch with me on twitter: [@nico_m__](https://twitter.com/nico_m__).
  
  **Please note, I've had issues with code completion in flashdevelop, possibly due to the singleton pattern, so it may be easier to just read the source if you need function names + parameters. (if anyone knows how to fix this let me know)**
