# website

Go to [thomas.simmer.free.fr](http://thomas.simmer.free.fr) to see this project.

3D model sources :
 - [Shuttle](https://skfb.ly/6R6Iu)
 - [Asteroids](https://skfb.ly/COwN)
 - [Earth texture](http://planetpixelemporium.com/planets.html)

Change in GLTFLoader.js l.1911:
```
		// Use an ImageBitmapLoader if imageBitmaps are supported. Moves much of the
		// expensive work of uploading a texture to the GPU off the main thread.
		// if ( typeof createImageBitmap !== 'undefined' && /Firefox/.test( navigator.userAgent ) === false ) {

		// 	this.textureLoader = new ImageBitmapLoader( this.options.manager );

		// } else {

			this.textureLoader = new TextureLoader( this.options.manager );

		// }
```
ImageBitmapLoader does not allow file:// in url in cordova-android.

ImageBitmapLoader does not allow shuttle loading (but allows asteroid loading) in cordova-ios.


## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).

### To serve cordova on Android
```
cd cordova
cordova platform add android
cordova run android
```

### To serve cordova on ios
```
cd cordova
cordova platform add ios
cordova run ios
```
