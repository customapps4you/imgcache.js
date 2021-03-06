imgcache.js
===========

The purpose of this JS library is to provide a nice interface for locally storing images for offline apps using
PhoneGap/Cordova or [browsers supporting the new html5 File API](http://caniuse.com/filesystem)
(e.g. Chrome).

This library is especially useful for mobile web applications using Phonegap/Cordova where the normal browser cache cannot be relied upon and where offline navigation is quite common.

Used with [imagesloaded](http://desandro.github.com/imagesloaded/) as shown in `examples/example2.html`, you can see that it can automatically:
* store images in cache
* replace images with cached version if they fail to load (offline / busy server..)

This is the best solution I have found so far to provide easy caching of images within a phonegap web app.

This library works with Phonegap/Cordova (v >= 1.7) so the supported platforms should be:
* Android [TESTED]
* iOS [TESTED]
* BlackBerry WebWorks (OS 5.0 and higher)
* Windows Phone 7 ( Mango )

All methods are asynchronous : use callbacks if required.

Using imgcache.js
=================

Requirements
------------
* jQuery (any version from 1.6 should do) or Zepto
* Phonegap/Cordova *optional* (v >= v1.7)
* [imagesloaded] (http://desandro.github.com/imagesloaded/) *optional*

Installation
------------
To use `imgcache.js`, you need to copy `js/imgcache.js` and add it to your
application's Javascript. You can then load it like so:

```html
<script src="imgcache.js" type="application/javascript"></script>
```
    
Using with PhoneGap/Cordova:
* Requires the File API permission in `config.xml`:
```
<feature name="http://api.phonegap.com/1.0/file"/>
```
* Remember to allow access to remote files by adding your domain in config.xml - or all domains using a wildcard: `<access origin="*" />`

Using with PhoneGap/Cordova (version >= 3.0):
* You may need additional permissions and settings. Check [Issue #15](https://github.com/chrisben/imgcache.js/issues/15) for more information
    
Using with Chrome or other browsers that support the [html5 filesystem API]:
* Beware of cross domain ajax issue! retrieve image from the same domain or set CORS solutions with the server...
* If the page is opened locally (file:// ..), Chrome needs to be loaded with the following flags: `--allow-file-access-from-files --allow-file-access` otherwise the local filesystem will not be accessible (security error)
* To navigate through the local filesystem open a new tab with filesystem:http://*yourSiteDomain*/persistent/ or filesystem:http://*yourSiteDomain*/temporary/

Setup your cache
----------------
Before initializing the cache, you must specify any default options you wish to configure:

```javascript
// write log to console
ImgCache.options.debug = true;

// increase allocated space on Chrome to 50MB, default was 10MB
ImgCache.options.chromeQuota = 50*1024*1024;
```

See `ImgCache.options` at the top of the source file for more settings.

After setting any custom configuration, initialize the cache:

```javascript
ImgCache.init(function(){
  alert('cache created successfully!');
}, function(){
  alert('check the log for errors');
});
````

If the cache successfully initializes, `ImgCache.ready` will be set to `true`.

If you're using imgcache.js with PhoneGap/Cordova, `ImgCache.init()` must be called in `onDeviceReady`, not before!

Note that in Chrome, the user will be prompted to give permission to the page for accessing the local filesystem (which will run the error callback if they decline).

Storing images
--------------
Images are stored into the local folder specified by `ImgCache.options.localCacheFolder`. To add a file to the cache:

```javascript
ImgCache.cacheFile('http://my-cdn.com/users/2/profile.jpg');
```

TODO: add documentation for `ImgCache.cacheBackground`

Using cached images
-------------------
Once an image is stored in the cache, you can replace the file displayed by the image tag:

```javascript
target = $('img#profile');
ImgCache.cacheFile(target.attr('src'), function(){
  ImgCache.useCachedFile(target, function(){
    alert('now using local copy');
  }, function(){
    alert('could not load from cache');
  })
});
```
    
To check if a file is stored locally:

```javascript
ImgCache.isCached(target.attr('src'), function(path, success){
  if(success){
    // already cached
    ImgCache.useCachedFile(target);
  } else {
    // not there, need to cache the image
    ImgCache.cacheFile(target.attr('src'), function(){
      ImgCache.useCachedFile(target);
    });
  }
});
```
    
When you no longer want to use the locally cached file:

```javascript
target = $('img#profile');
ImgCache.useOnlineFile(target);
```

Clearing the cache
------------------
To remove all cached files, clear the local cache folder:

```javascript
ImgCache.clearCache(function(){
  // continue cleanup...
}, function(){
  // something went wrong
});
```
    
There is currently no way to invalidate single images from the cache.

High level API
--------------
* ImgCache.init() *initialises the local cache*
* ImgCache.cacheFile() *writes a copy of a file into the local cache*
* ImgCache.isCached() *checks if a the given image exists in the cache - does not check if the latest version of that file is cached*
* ImgCache.useCachedFile() *replaces the img src with the cached version*
* ImgCache.useCachedFileWithSource() *similar to useCachedFile but with the image source url as extra parameter*
* ImgCache.useOnlineFile() *replaces back the img src with the original (online) version*
* ImgCache.clearCache() *clears the local cache folder*
* ImgCache.cacheBackground() *caches the background image of an element*
* ImgCache.useCachedBackground() *replaces the background image source of the given element with the cached version*

Options
-------
See ImgCache.options at the top of the source file for the list of options.
Options can be overriden from your own script, no need to modify the library!

Code samples
------------
See html files in the `examples/` folder.

Release Notes
-------------
See `CHANGELOG.md` for recent updates.

License
-------
Copyright 2012-2013 (c) Christophe BENOIT - [Wobis](http://www.wobis.fr)

Apache License - see LICENSE.md

Code from http://code.google.com/p/tiny-sha1/ is being used which is under the MIT License.
The copyright for this part belongs to the creator of this work.

Todo
----
* Find a solution for cache invalidation in case an image changes
* When Chrome finally supports canvas.toBlob(), possibly replace download method with new one that draws an Image into a canvas and then retrieves its content using the toBlob() method -- or use [canvas-toBlob.js] (https://github.com/eligrey/canvas-toBlob.js)
