# Sequence Maker

## In one sentence

Command line Python script that 1) takes geotagged 360 images, 2) reads latitude, longitude and altitude, 3) automatically calculates connections between photos, and 4) embeds connections as JSON object in image metadata.

## Why we built this

We shoot 360 tours of the natural world (paths, rivers, bike trails...).

As we shoot photos in a timelapse, each photo in the timelapse has a relationship (e.g. photo 1 is connected to photo 2, photo 2 ...).

This information is not embedded into the images by the camera.

When uploading images to Google Street View and others, there is the option to explicitly define connection information at upload (vs. calculation on Google servers).

Sometimes we also shoot at a high frame rate (or get stopped on route). This means many photos get taken very close to each other, which we would otherwise have to discard manually.

To ensure the correct connections are made between photos we use Sequence Maker to define these connections from a series of timelapse photos. 

## How it works

1. You define the timelapse series of photos, desired photo spacing (by distance or capture time), and how they should be connected
2. IF distance selected, the script calculates the distance between photos
3. The script orders the photos in specified order (either capture time or distance)
4. The script discards images that don't match the specified spacing condition
5. The script calculates the distance, elevation change, time difference, and heading between remaining photos
6. The script writes a JSON object into the remaining photos `-xmp:Notes` tag with this information
 
## Requirements

### OS Requirements

Works on Windows, Linux and MacOS.

### Software Requirements

* Python version 3.6+
* [exiftool](https://exiftool.org/)

### Image Requirements

All images must be geotagged with the following values:

* `GPSLatitude`
* `GPSLongitude`
* `GPSAltitude`
* `GPTimeStamp`
* `GPDateStamp`
* `GPDateTime`

{"MAPDeviceMake": "GoPro", "MAPOrientation": 1, "MAPAltitude": 180.129, "MAPSettingsUserKey": "pG6L5dj57PAuxaEUUgj9hA", "MAPPhotoUUID": "ff6e6658-ab5b-4fcf-aa4d-d433b4325a4e", "MAPDeviceModel": "GoPro Fusion FS1.04.02.00.00", "MAPCompassHeading": {"TrueHeading": 0.0, "MagneticHeading": 0.0}, "MAPMetaTags": {"strings": [{"key": "mapillary_tools_version", "value": "0.5.0"}, {"key": "mapillary_uploader_version", "value": "1.2.3"}]}, "MAPSettingsUsername": "trekviewhq", "MAPCaptureTime": "2019_10_13_17_47_13_000", "MAPLatitude": 51.24514444444444, "MAPSequenceUUID": "4e46f539-88ce-4bf2-8c58-9d6eec9e4024", "MAPLongitude": -0.8202777777777778}

If a photo does not contain this information, you will be presented with a warning, and asked whether you wish continue (discard the photos missing this information from the process).

This software will work with most image formats. Whilst it is designed for 360 photos, it will work with a sequence of traditional flat (Cartesian) images too.

## Quick start guide

### Arguments

* frame_rate: maximum frames per second (value between 0.05 and 5)

_A note on frame_rate spacing:  designed to discard photos when unnecessary high frame rate. the script will start on first photo and count to the nearest photo by value specified. All photos in between will be discarded. The script will then start from 0 and count to the next photo using the value specified. All photos in between will be discarded._

* horizontal_distance_min: minimum spacing between photos in meters (between 0.5 and 20) e.g. photos cannot be closer than this value.

_A note on distance_min spacing: designed to discard photos when lots taken in same place. if the value you define is less than the distance between two photo values, the script will still make a connection (e.g min_spacing = 10m and actual disantce = 20m, photos will still be joined. The script will start on first photo and count to the nearest photo by value specified. All photos in between will be discarded. The script will then start from 0 and count to the next photo using the value specified. All photos in between will be discarded._

_Note, if two or more of these arguments are used (frame_rate, distance_max, distance_min, or vertical_distance_max arguments), the script will process in the order: 1) frame_rate, 2) distance_max, 3) distance_min, and 4) vertical_distance_max._

* join mode:
	- time (ascending e.g. 00:01 - 00:10); OR
	- filename (ascending e.g A.jpg > Z.jpg)

_A note on join modes: generally you should join by time unless you have a specific usecase. time will join the photo to the next photo using the photos `GPDateTime` value. Filename will join the photo to the next photo in ascending alphabetical order._

![Sequence maker joins](/sequence-maker-diagram.png)

### Format

```
python sequence-maker.py -[SPACING] [SPACING VALUE] -[JOIN MODE] [INPUT PHOTO DIRECTORY] [OUTPUT PHOTO DIRECTORY]
```

### Examples

**Connect photos with a minimum interval of 1 seconds and minimum distance between photos of 3 meters in ascending time order (recommended)**

```
python sequence-maker.py -frame_rate 1 horizontal_distance_min 3 -time my_input_photos/ my_output_photos/
````

**Connect photos within 10m of each other in ascending time order (recommended)**

### Output

Sequence maker will generate new photo files with JSON objects with destination connections printed under the [XMP] `Notes` tag:

```
{
	connections: {
		[FILENAME_1]: {
			distance_mtrs: # horizontal distance (haversine) to destination file
			elevation_mtrs: # vertical distance to destination file
			heading_deg: # between 0 and 360
			time_sec: # time in seconds before destination file captured (can be negative, if source photo taken after destination photo -- for example, when moving backwards)
		},
		[FILENAME_n]: {
			distance_mtrs:
			elevation_mtrs:
			heading_deg:
			time_sec: 
	}
	create_date: 2020-05-30:00:00:00
	software: sequence-maker
}

```

You can view the the value of tags assigned:

```
exiftool -G -a  exiftool PHOTO_FILE > PHOTO_FILE_metadata.txt
```

## Support 

We offer community support for all our software on our Campfire forum. [Ask a question or make a suggestion here](https://campfire.trekview.org/c/support/8).

## License

Sequence Maker is licensed under a [GNU AGPLv3 License](https://github.com/trek-view/sequence-maker/blob/master/LICENSE.txt).