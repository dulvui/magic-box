+++
title = "Godot - Automated screenshots"
description = "Automatically take screenshots of you Godot game for all possible devices with a tiny script"
date = 2023-01-22
updated = 2023-01-22
aliases = ["gas"]
[extra]
mastodon_link = "https://mastodon.social/@dulvui/110391932466547214"
hackernews_link = "https://news.ycombinator.com/item?id=36466438"
+++

To publish a game to Apple App Store or Google Play Store, screenshots of the game are needed.
For Android you need screenshots of mobile devices and tablets and for iOS you need a screenshot of each currently supported device.  
With a little coding this boring and repetitive task can be automated.
Of course, it depends a lot on the type of game and its mechanics, if its even possible to automate this task.
But if it's a simple game like [Ball2Box](@/games/ball2box/index.md) it will be easy. 


## How does it work
First, the resolutions of the devices are defined.
The iPad screenshots can be reused as 7 inch and 10 inch tablet screenshots for Android.
```gd
const resolutions = {
	 "Android" :Vector2(1080,1920),
	 "iPhone5.5" :Vector2(1242, 2208),
	 "iPhone6.5" :Vector2(1284, 2778),
	 "iPhone6.7": Vector2(1290, 2796),
	 "iPad12.9" : Vector2(2048, 2732)
}
```

Then the paths of the scenes, that should be used for the screenshots, are defined.
```gd
const scenes = [
	"res://src/levels/Level1.tscn",
	"res://src/levels/Level10.tscn",
	"res://src/levels/Level11.tscn",
	"res://src/levels/Level13.tscn",
	"res://src/levels/Level24.tscn",
	"res://src/levels/Level70.tscn",
	"res://src/levels/Level99.tscn",
	"res://src/levels/Level105.tscn",
]
```

The following function changes the scene to the ones we defined in the last step and hides the menu.
```gd
func _change_scene(scene_path):
	if scene != null:
		remove_child(scene)
		scene.queue_free()

	var next_scene = load(scene_path)
	scene = next_scene.instance()
	add_child(scene)
	
	# hide menu
	scene.get_node("UI/Menu").hide()
```

With the following OS function call, we can change the resolution of the screen while the game is executed.
```gd
OS.set_window_size(resolutions.get(resolution))
```

Then we get the texture data of the whole screen with the get_texture() Viewport function call.
Since the image would be upside down, we flip it first and then we can save it on the file system.
```gd
var image = get_viewport().get_texture().get_data().get_rect(fullscreen)
image.flip_y()
image.save_png("../screenshots/" + resolution + "-" + str(scene_counter) + ".png")
```

## The whole code
Here, the whole code that iterates over the resolutions and scenes and takes the screenshots.
It can be executed with the "Play custom scene" button on the right top corner of the Godot editor.
```gd
extends Spatial

const resolutions = {
	 "Android" :Vector2(1080,1920),
	 "iPhone5.5" :Vector2(1242, 2208),
	 "iPhone6.5" :Vector2(1284, 2778),
	 "iPhone6.7": Vector2(1290, 2796),
	 "iPad12.9" : Vector2(2048, 2732)
}
	
const scenes = [
	"res://src/levels/Level1.tscn",
	"res://src/levels/Level10.tscn",
	"res://src/levels/Level11.tscn",
	"res://src/levels/Level13.tscn",
	"res://src/levels/Level24.tscn",
	"res://src/levels/Level70.tscn",
	"res://src/levels/Level99.tscn",
	"res://src/levels/Level105.tscn",
]
	
var scene

func _ready():
	
	var scene_counter = 1
	
	for scene in scenes:
		_change_scene(scene)
		for resolution in resolutions.keys():
			OS.set_window_size(resolutions.get(resolution))

			yield(get_tree().create_timer(2), "timeout")
			
			# calculate x/y offset for bigger screens like iPads to center the screenshot
			var x = (resolutions.get(resolution).x - get_viewport().get_texture().get_size().x) / 2
			var y = (resolutions.get(resolution).y - get_viewport().get_texture().get_size().y) / 2
			var fullscreen = Rect2(0 - x, 0 - y, resolutions.get(resolution).x, resolutions.get(resolution).y)
			
			var image = get_viewport().get_texture().get_data().get_rect(fullscreen)
			image.flip_y()
			image.save_png("../screenshots/" + resolution + "-" + str(scene_counter) + ".png")
		scene_counter += 1
	get_tree().quit()
	
	
func _change_scene(scene_path):
	if scene != null:
		remove_child(scene)
		scene.queue_free()

	var next_scene = load(scene_path)
	scene = next_scene.instance()
	add_child(scene)
	
	# hide menu
	scene.get_node("UI/Menu").hide()

```

This script could be used to create a tool for the Godot Asset Store, so it could be simply installed from there, but for my use case this is enough for now.    
You can find this code and the whole game also on [Github](https://github.com/dulvui/ball2box/blob/main/game/src/screenshot-taker/ScreenshotTaker.gd).

