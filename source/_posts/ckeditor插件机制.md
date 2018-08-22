---
"title": "ckeditor插件机制"
"date": "2016/6/16 20:46:25"
categories:
- js
- ckeditor

tags:
- js
- ckeditor
---


### 插件加载过程

ckeditor base code 加载完成之后，开始执行用户代码，调用editor构造函数，生成editor实例。
Editor构造函数中通过setTimout将initConfig加入浏览器的任务队列

```javascript
CKEDITOR.tools.setTimeout( function() {
   			if ( this.status !== 'destroyed' ) {
   				initConfig( this, instanceConfig );
   			} else {
   				CKEDITOR.warn( 'editor-incorrect-destroy' );
   			}
   		}, 0, this );

```

由于setTimeout延时为0,即主线程一空，会立刻执行initConfig(this, instanceConfig)，该函数会加载自定义配置文件，
并调用onConfigLoaded( editor )->initComponents( editor )->loadSkin( editor )->loadLang( editor )->preloadStylesSet( editor )->loadPlugins( editor )加载插件
在loadPlugins中完成了以下几件事情：

1. 根据配置确定最终加载插件
2. 调用CKEDITOR.plugins.load( plugins.split( ‘,’ ), function( plugins )），CKEDITOR.plugins中的代码如下

```javascript
CKEDITOR.plugins = new CKEDITOR.resourceManager( 'plugins/', 'plugin' );
   // PACKAGER_RENAME( CKEDITOR.plugins )
   CKEDITOR.plugins.load = CKEDITOR.tools.override( CKEDITOR.plugins.load, function( originalLoad ) {...})
```

resourceManager是ckeditor资源管理器，负责将注册插件，并调用CKEDITOR.scriptLoader.load加载插件文件。CKEDITOR.plugins.load是插件实现和核心函数，该函数会在插件加载成功后，回调插件的loaded等函数。


```javascript
// Initialize all plugins that have the "beforeInit" and "init" methods defined.
      				var methods = [ 'beforeInit', 'init', 'afterInit' ];
      				for ( var m = 0; m < methods.length; m++ ) {
      					for ( var i = 0; i < pluginsArray.length; i++ ) {
      						var plugin = pluginsArray[ i ];
      						// Uses the first loop to update the language entries also.
      						if ( m === 0 && languageCodes[ i ] && plugin.lang && plugin.langEntries )
      							editor.lang[ plugin.name ] = plugin.langEntries[ languageCodes[ i ] ];
      						// Call the plugin method (beforeInit and init).
      						if ( plugin[ methods[ m ] ] )
      							plugin[ methods[ m ] ]( editor );
      					}
      				}
```


1. 加载插件语言包
2. 触发loaded事件，调用preloadStylesSet，加载预定义格式，调用UI,轮询插件loaded事件队列。
3. 触发instanceLoaded事件

```javascript

CKEDITOR.on( 'instanceLoaded', function( evt ) {
		var editor = evt.editor;
		// and flag that the element was locked by our code so it'll be editable by the editor functions (#6046).
		editor.on( 'insertElement', function( evt ) {
			var element = evt.data;
			if ( element.type == CKEDITOR.NODE_ELEMENT && ( element.is( 'input' ) || element.is( 'textarea' ) ) ) {
				// // The element is still not inserted yet, force attribute-based check.
				if ( element.getAttribute( 'contentEditable' ) != 'false' )
					element.data( 'cke-editable', element.hasAttribute( 'contenteditable' ) ? 'true' : '1' );
				element.setAttribute( 'contentEditable', false );
			}
		} );
		editor.on( 'selectionChange', function( evt ) {
			if ( editor.readOnly )
				return;
			// Auto fixing on some document structure weakness to enhance usabilities. (#3190 and #3189)
			var sel = editor.getSelection();
			// Do it only when selection is not locked. (#8222)
			if ( sel && !sel.isLocked ) {
				var isDirty = editor.checkDirty();
				// Lock undoM before touching DOM to prevent
				// recording these changes as separate snapshot.
				editor.fire( 'lockSnapshot' );
				fixDom( evt );
				editor.fire( 'unlockSnapshot' );
				!isDirty && editor.resetDirty();
			}
		} );
	} );

```

提示：instanceLoaded事件触发之后，程序流程会脱离editor，接着触发其他事件，直到程序初始化完成。有需要的可以自行调试，跟踪流程。

### ckeditor插件系统

```
 _________           ______________               __________
|editor.js|-------->| plugins.js   | <------------|plugin.js|
|_________|         |______________|              |_________|
                           |
                           |
                           ▽
                   ____________________
                   | resourceManager.js|
                   |___________________|
                            |
                            |
                            ▽
                     _______________
                    |scriptLoader.js|
                    |_______________|
```

1. plugin.js定义插件，插件具体开发规范可查看官方文档。
2. scriptLoader.js 通过js动态插入scirpt标签加载插件。
3. resourceManager.js 将配置转解析成对应的路径，并调用scriptLoader加载。
4. plugins.js 插件机制的核心文件，原理是回调用户实现的接口，来使插件生效。

```javascript
for ( var pluginName in plugins ) {
						var plugin = plugins[ pluginName ],
							requires = plugin && plugin.requires;
						if ( !initialized[ pluginName ] ) {
							// Register all icons eventually defined by this plugin.
							if ( plugin.icons ) {
								var icons = plugin.icons.split( ',' );
								for ( var ic = icons.length; ic--; ) {
									CKEDITOR.skin.addIcon( icons[ ic ],
										plugin.path +
										'icons/' +
										( CKEDITOR.env.hidpi && plugin.hidpi ? 'hidpi/' : '' ) +
										icons[ ic ] +
										'.png' );
								}
							}
							initialized[ pluginName ] = 1;
						}
						if ( requires ) {
							// Trasnform it into an array, if it's not one.
							if ( requires.split )
								requires = requires.split( ',' );
							for ( var i = 0; i < requires.length; i++ ) {
								if ( !allPlugins[ requires[ i ] ] )
									requiredPlugins.push( requires[ i ] );
							}
						}
					}
```

5. editor.js 编辑器实例，为插件提供基础的运行环境和API.