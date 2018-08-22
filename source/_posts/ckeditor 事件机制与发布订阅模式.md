---
"title": "ckeditor 事件机制与发布订阅模式"
"date": "2016/6/12 20:46:25"
categories:
- js
- ckeditor

tags:
- js
- ckeditor
---


### 前言

js是事件驱动的，ckeditor也不列外，在了解了ckeditor的引导程序之后，一起学习一下ckeditor事件机制的实现。

ckedtor最核心的事件机制实现的文件在event.js中，是基于发布订阅模式的。发布订阅模式又叫观察者模式，它定义的关系，让多个观察对象同时监听某一个主题对象，这个主题发生变化时会通知所有的观察者对象，使得他们能够自动更新。使用观察者模式的好处

1. 支持简单的广播通信，自动通知所有已经订阅过的对象。
2. 页面载入后目标对象很容易与观察者存在一种动态关联，增加了灵活性。
3. 目标对象与观察者之间的抽象耦合关系能够单独扩展以及重用。

### 正文

我们设想一下，还是沿用我们的老方式，情报局要收集整个敌军部队的消息，那肯定不可能是一个人完成的。我们一起用发布订阅模式解决一下。假设敌军有A集团军，B集团军,C级团军。那情报局怎么架构呢？我们派三个特工分别潜入A,B,C集团军,然后告诉T1,T2,T3。你们拿到情报后，直接发给情报局T0。然和T0转交给L1,L2,L3处理。那可能有人要问了，干嘛这么多事，直接让T1发给A1,T2发给A2，不接可以了么,T0可以去掉。那现在问题来了，假设情报在发给L1,L2,L3的同时，假设最近战况比较紧张，情报局老大T0也要看一下，那么总不能L0发个电报告诉A1,A2,A3，你在发给Tn的时候，也发我一份，那么特工发这么多人，是非常容易暴露的，而且每次改变情报收听者，还要告诉情报发布者。所以我们需要建立一个收发处T0,它负责维护情报发送者与监听者的对应关系。好勒，我们一共需要三类角色，情报发布者，情报监听者，发布与监听者对应关系维护者。

现在我们先来看下情报发布者，特工有的通信方式是可以复用的，有的是不可以的，用完一次后就得销毁，防止被人端掉上级

```javascript

/**
		 * Fires an specific event in the object. All registered listeners are
		 * called at this point.
		 *
		 *		someObject.on( 'someEvent', function() { ... } );
		 *		someObject.on( 'someEvent', function() { ... } );
		 *		someObject.fire( 'someEvent' );				// Both listeners are called.
		 *
		 *		someObject.on( 'someEvent', function( event ) {
		 *			alert( event.data );					// 'Example'
		 *		} );
		 *		someObject.fire( 'someEvent', 'Example' );
		 *
		 * @method
		 * @param {String} eventName The event name to fire.
		 * @param {Object} [data] Data to be sent as the
		 * {@link CKEDITOR.eventInfo#data} when calling the listeners.
		 * @param {CKEDITOR.editor} [editor] The editor instance to send as the
		 * {@link CKEDITOR.eventInfo#editor} when calling the listener.
		 * @returns {Boolean/Object} A boolean indicating that the event is to be
		 * canceled, or data returned by one of the listeners.
		 */
		fire: ( function() {
			// Create the function that marks the event as stopped.
			var stopped = 0;
			var stopEvent = function() {
					stopped = 1;
				};
			// Create the function that marks the event as canceled.
			var canceled = 0;
			var cancelEvent = function() {
					canceled = 1;
				};
			return function( eventName, data, editor ) {
				// Get the event entry.
				var event = getPrivate( this )[ eventName ];
				// Save the previous stopped and cancelled states. We may
				// be nesting fire() calls.
				var previousStopped = stopped,
					previousCancelled = canceled;
				// Reset the stopped and canceled flags.
				stopped = canceled = 0;
				if ( event ) {
					var listeners = event.listeners;
					if ( listeners.length ) {
						// As some listeners may remove themselves from the
						// event, the original array length is dinamic. So,
						// let's make a copy of all listeners, so we are
						// sure we'll call all of them.
						listeners = listeners.slice( 0 );
						var retData;
						// Loop through all listeners.
						for ( var i = 0; i < listeners.length; i++ ) {
							// Call the listener, passing the event data.
							if ( event.errorProof ) {
								try {
									retData = listeners[ i ].call( this, editor, data, stopEvent, cancelEvent );
								} catch ( er ) {}
							} else {
								retData = listeners[ i ].call( this, editor, data, stopEvent, cancelEvent );
							}
							if ( retData === false )
								canceled = 1;
							else if ( typeof retData != 'undefined' )
								data = retData;
							// No further calls is stopped or canceled.
							if ( stopped || canceled )
								break;
						}
					}
				}
				var ret = canceled ? false : ( typeof data == 'undefined' ? true : data );
				// Restore the previous stopped and canceled states.
				stopped = previousStopped;
				canceled = previousCancelled;
				return ret;
			};
		} )(),
		/**
		 * Fires an specific event in the object, releasing all listeners
		 * registered to that event. The same listeners are not called again on
		 * successive calls of it or of {@link #fire}.
		 *
		 *		someObject.on( 'someEvent', function() { ... } );
		 *		someObject.fire( 'someEvent' );			// Above listener called.
		 *		someObject.fireOnce( 'someEvent' );		// Above listener called.
		 *		someObject.fire( 'someEvent' );			// No listeners called.
		 *
		 * @param {String} eventName The event name to fire.
		 * @param {Object} [data] Data to be sent as the
		 * {@link CKEDITOR.eventInfo#data} when calling the listeners.
		 * @param {CKEDITOR.editor} [editor] The editor instance to send as the
		 * {@link CKEDITOR.eventInfo#editor} when calling the listener.
		 * @returns {Boolean/Object} A booloan indicating that the event is to be
		 * canceled, or data returned by one of the listeners.
		 */
		fireOnce: function( eventName, data, editor ) {
			var ret = this.fire( eventName, data, editor );
			delete getPrivate( this )[ eventName ];
			return ret;
		},
		/**
		 * Unregisters a listener function from being called at the specified
		 * event. No errors are thrown if the listener has not been registered previously.
		 *
		 *		var myListener = function() { ... };
		 *		someObject.on( 'someEvent', myListener );
		 *		someObject.fire( 'someEvent' );					// myListener called.
		 *		someObject.removeListener( 'someEvent', myListener );
		 *		someObject.fire( 'someEvent' );					// myListener not called.
		 *
		 * @param {String} eventName The event name.
		 * @param {Function} listenerFunction The listener function to unregister.
		 */

```

接下来一起看一下，监听与发布者对应关系维护者，那么他需要干什么呢，看一下这份情报有哪些订阅者，退订处理，新建订阅

```javascript

/**
		 * Predefine some intrinsic properties on a specific event name.
		 *
		 * @param {String} name The event name
		 * @param meta
		 * @param [meta.errorProof=false] Whether the event firing should catch error thrown from a per listener call.
		 */
		define: function( name, meta ) {
			var entry = getEntry.call( this, name );
			CKEDITOR.tools.extend( entry, meta, true );
		},
/**
		 * Unregisters a listener function from being called at the specified
		 * event. No errors are thrown if the listener has not been registered previously.
		 *
		 *		var myListener = function() { ... };
		 *		someObject.on( 'someEvent', myListener );
		 *		someObject.fire( 'someEvent' );					// myListener called.
		 *		someObject.removeListener( 'someEvent', myListener );
		 *		someObject.fire( 'someEvent' );					// myListener not called.
		 *
		 * @param {String} eventName The event name.
		 * @param {Function} listenerFunction The listener function to unregister.
		 */
		removeListener: function( eventName, listenerFunction ) {
			// Get the event entry.
			var event = getPrivate( this )[ eventName ];
			if ( event ) {
				var index = event.getListenerIndex( listenerFunction );
				if ( index >= 0 )
					event.listeners.splice( index, 1 );
			}
		},
		/**
		 * Remove all existing listeners on this object, for cleanup purpose.
		 */
		removeAllListeners: function() {
			var events = getPrivate( this );
			for ( var i in events )
				delete events[ i ];
		},
		/**
		 * Checks if there is any listener registered to a given event.
		 *
		 *		var myListener = function() { ... };
		 *		someObject.on( 'someEvent', myListener );
		 *		alert( someObject.hasListeners( 'someEvent' ) );	// true
		 *		alert( someObject.hasListeners( 'noEvent' ) );		// false
		 *
		 * @param {String} eventName The event name.
		 * @returns {Boolean}
		 */
		hasListeners: function( eventName ) {
			var event = getPrivate( this )[ eventName ];
			return ( event && event.listeners.length > 0 );

```


最后看下订阅者

```javascript

/**
		 * Registers a listener to a specific event in the current object.
		 *
		 *		someObject.on( 'someEvent', function() {
		 *			alert( this == someObject );		// true
		 *		} );
		 *
		 *		someObject.on( 'someEvent', function() {
		 *			alert( this == anotherObject );		// true
		 *		}, anotherObject );
		 *
		 *		someObject.on( 'someEvent', function( event ) {
		 *			alert( event.listenerData );		// 'Example'
		 *		}, null, 'Example' );
		 *
		 *		someObject.on( 'someEvent', function() { ... } );						// 2nd called
		 *		someObject.on( 'someEvent', function() { ... }, null, null, 100 );		// 3rd called
		 *		someObject.on( 'someEvent', function() { ... }, null, null, 1 );		// 1st called
		 *
		 * @param {String} eventName The event name to which listen.
		 * @param {Function} listenerFunction The function listening to the
		 * event. A single {@link CKEDITOR.eventInfo} object instanced
		 * is passed to this function containing all the event data.
		 * @param {Object} [scopeObj] The object used to scope the listener
		 * call (the `this` object). If omitted, the current object is used.
		 * @param {Object} [listenerData] Data to be sent as the
		 * {@link CKEDITOR.eventInfo#listenerData} when calling the
		 * listener.
		 * @param {Number} [priority=10] The listener priority. Lower priority
		 * listeners are called first. Listeners with the same priority
		 * value are called in registration order.
		 * @returns {Object} An object containing the `removeListener`
		 * function, which can be used to remove the listener at any time.
		 */
		on: function( eventName, listenerFunction, scopeObj, listenerData, priority ) {
			// Create the function to be fired for this listener.
			function listenerFirer( editor, publisherData, stopFn, cancelFn ) {
				var ev = {
					name: eventName,
					sender: this,
					editor: editor,
					data: publisherData,
					listenerData: listenerData,
					stop: stopFn,
					cancel: cancelFn,
					removeListener: removeListener
				};
				var ret = listenerFunction.call( scopeObj, ev );
				return ret === false ? false : ev.data;
			}
			function removeListener() {
				me.removeListener( eventName, listenerFunction );
			}
			var event = getEntry.call( this, eventName );
			if ( event.getListenerIndex( listenerFunction ) < 0 ) {
				// Get the listeners.
				var listeners = event.listeners;
				// Fill the scope.
				if ( !scopeObj )
					scopeObj = this;
				// Default the priority, if needed.
				if ( isNaN( priority ) )
					priority = 10;
				var me = this;
				listenerFirer.fn = listenerFunction;
				listenerFirer.priority = priority;
				// Search for the right position for this new listener, based on its
				// priority.
				for ( var i = listeners.length - 1; i >= 0; i-- ) {
					// Find the item which should be before the new one.
					if ( listeners[ i ].priority <= priority ) {
						// Insert the listener in the array.
						listeners.splice( i + 1, 0, listenerFirer );
						return { removeListener: removeListener };
					}
				}
				// If no position has been found (or zero length), put it in
				// the front of list.
				listeners.unshift( listenerFirer );
			}
			return { removeListener: removeListener };
		},

```

那么现在情报局boss，突然说我要查看下021编号的这份情报，所以关系维护者还要实现，这个功能，

```javascript
/**
		 * @static
		 * @property {Boolean} useCapture
		 * @todo
		 */
		/**
		 * Register event handler under the capturing stage on supported target.
		 */
		capture: function() {
			CKEDITOR.event.useCapture = 1;
			var retval = this.on.apply( this, arguments );
			CKEDITOR.event.useCapture = 0;
			return retval;
		},
```

好勒，那现在警察局也需要这个机制，其实很多地方都需要这个机制，那怎么办呢

```javascript
/**
 * Implements the {@link CKEDITOR.event} features in an object.
 *
 *		var myObject = { message: 'Example' };
 *		CKEDITOR.event.implementOn( myObject );
 *
 *		myObject.on( 'testEvent', function() {
 *			alert( this.message );
 *		} );
 *		myObject.fire( 'testEvent' ); // 'Example'
 *
 * @static
 * @param {Object} targetObject The object into which implement the features.
 */
event.implementOn = function( targetObject ) {
	var eventProto = CKEDITOR.event.prototype;
	for ( var prop in eventProto ) {
		if ( targetObject[ prop ] == null )
			targetObject[ prop ] = eventProto[ prop ];
	}
};
```


好勒，我们的发布订阅者模式之事件机制已经完工了。我们看一下完整的代码(我是不会告诉你，我抄了ckeditor的事件代码的，哈哈。。。)

```javascript
/**
 * @license Copyright (c) 2003-2016, CKSource - Frederico Knabben. All rights reserved.
 * For licensing, see LICENSE.md or http://ckeditor.com/license
 */
/**
 * @fileOverview Defines the {@link CKEDITOR.event} class, which serves as the
 *		base for classes and objects that require event handling features.
 */
if ( !CKEDITOR.event ) {
	/**
	 * Creates an event class instance. This constructor is rarely used, being
	 * the {@link #implementOn} function used in class prototypes directly
	 * instead.
	 *
	 * This is a base class for classes and objects that require event
	 * handling features.
	 *
	 * Do not confuse this class with {@link CKEDITOR.dom.event} which is
	 * instead used for DOM events. The CKEDITOR.event class implements the
	 * internal event system used by the CKEditor to fire API related events.
	 *
	 * @class
	 * @constructor Creates an event class instance.
	 */
	CKEDITOR.event = function() {};
	/**
	 * Implements the {@link CKEDITOR.event} features in an object.
	 *
	 *		var myObject = { message: 'Example' };
	 *		CKEDITOR.event.implementOn( myObject );
	 *
	 *		myObject.on( 'testEvent', function() {
	 *			alert( this.message );
	 *		} );
	 *		myObject.fire( 'testEvent' ); // 'Example'
	 *
	 * @static
	 * @param {Object} targetObject The object into which implement the features.
	 */
	CKEDITOR.event.implementOn = function( targetObject ) {
		var eventProto = CKEDITOR.event.prototype;
		for ( var prop in eventProto ) {
			if ( targetObject[ prop ] == null )
				targetObject[ prop ] = eventProto[ prop ];
		}
	};
	CKEDITOR.event.prototype = ( function() {
		// Returns the private events object for a given object.
		var getPrivate = function( obj ) {
				var _ = ( obj.getPrivate && obj.getPrivate() ) || obj._ || ( obj._ = {} );
				return _.events || ( _.events = {} );
			};
		var eventEntry = function( eventName ) {
				this.name = eventName;
				this.listeners = [];
			};
		eventEntry.prototype = {
			// Get the listener index for a specified function.
			// Returns -1 if not found.
			getListenerIndex: function( listenerFunction ) {
				for ( var i = 0, listeners = this.listeners; i < listeners.length; i++ ) {
					if ( listeners[ i ].fn == listenerFunction )
						return i;
				}
				return -1;
			}
		};
		// Retrieve the event entry on the event host (create it if needed).
		function getEntry( name ) {
			// Get the event entry (create it if needed).
			var events = getPrivate( this );
			return events[ name ] || ( events[ name ] = new eventEntry( name ) );
		}
		return {
			/**
			 * Predefine some intrinsic properties on a specific event name.
			 *
			 * @param {String} name The event name
			 * @param meta
			 * @param [meta.errorProof=false] Whether the event firing should catch error thrown from a per listener call.
			 */
			define: function( name, meta ) {
				var entry = getEntry.call( this, name );
				CKEDITOR.tools.extend( entry, meta, true );
			},
			/**
			 * Registers a listener to a specific event in the current object.
			 *
			 *		someObject.on( 'someEvent', function() {
			 *			alert( this == someObject );		// true
			 *		} );
			 *
			 *		someObject.on( 'someEvent', function() {
			 *			alert( this == anotherObject );		// true
			 *		}, anotherObject );
			 *
			 *		someObject.on( 'someEvent', function( event ) {
			 *			alert( event.listenerData );		// 'Example'
			 *		}, null, 'Example' );
			 *
			 *		someObject.on( 'someEvent', function() { ... } );						// 2nd called
			 *		someObject.on( 'someEvent', function() { ... }, null, null, 100 );		// 3rd called
			 *		someObject.on( 'someEvent', function() { ... }, null, null, 1 );		// 1st called
			 *
			 * @param {String} eventName The event name to which listen.
			 * @param {Function} listenerFunction The function listening to the
			 * event. A single {@link CKEDITOR.eventInfo} object instanced
			 * is passed to this function containing all the event data.
			 * @param {Object} [scopeObj] The object used to scope the listener
			 * call (the `this` object). If omitted, the current object is used.
			 * @param {Object} [listenerData] Data to be sent as the
			 * {@link CKEDITOR.eventInfo#listenerData} when calling the
			 * listener.
			 * @param {Number} [priority=10] The listener priority. Lower priority
			 * listeners are called first. Listeners with the same priority
			 * value are called in registration order.
			 * @returns {Object} An object containing the `removeListener`
			 * function, which can be used to remove the listener at any time.
			 */
			on: function( eventName, listenerFunction, scopeObj, listenerData, priority ) {
				// Create the function to be fired for this listener.
				function listenerFirer( editor, publisherData, stopFn, cancelFn ) {
					var ev = {
						name: eventName,
						sender: this,
						editor: editor,
						data: publisherData,
						listenerData: listenerData,
						stop: stopFn,
						cancel: cancelFn,
						removeListener: removeListener
					};
					var ret = listenerFunction.call( scopeObj, ev );
					return ret === false ? false : ev.data;
				}
				function removeListener() {
					me.removeListener( eventName, listenerFunction );
				}
				var event = getEntry.call( this, eventName );
				if ( event.getListenerIndex( listenerFunction ) < 0 ) {
					// Get the listeners.
					var listeners = event.listeners;
					// Fill the scope.
					if ( !scopeObj )
						scopeObj = this;
					// Default the priority, if needed.
					if ( isNaN( priority ) )
						priority = 10;
					var me = this;
					listenerFirer.fn = listenerFunction;
					listenerFirer.priority = priority;
					// Search for the right position for this new listener, based on its
					// priority.
					for ( var i = listeners.length - 1; i >= 0; i-- ) {
						// Find the item which should be before the new one.
						if ( listeners[ i ].priority <= priority ) {
							// Insert the listener in the array.
							listeners.splice( i + 1, 0, listenerFirer );
							return { removeListener: removeListener };
						}
					}
					// If no position has been found (or zero length), put it in
					// the front of list.
					listeners.unshift( listenerFirer );
				}
				return { removeListener: removeListener };
			},
			/**
			 * Similiar with {@link #on} but the listener will be called only once upon the next event firing.
			 *
			 * @see CKEDITOR.event#on
			 */
			once: function() {
				var args = Array.prototype.slice.call( arguments ),
					fn = args[ 1 ];
				args[ 1 ] = function( evt ) {
					evt.removeListener();
					return fn.apply( this, arguments );
				};
				return this.on.apply( this, args );
			},
			/**
			 * @static
			 * @property {Boolean} useCapture
			 * @todo
			 */
			/**
			 * Register event handler under the capturing stage on supported target.
			 */
			capture: function() {
				CKEDITOR.event.useCapture = 1;
				var retval = this.on.apply( this, arguments );
				CKEDITOR.event.useCapture = 0;
				return retval;
			},
			/**
			 * Fires an specific event in the object. All registered listeners are
			 * called at this point.
			 *
			 *		someObject.on( 'someEvent', function() { ... } );
			 *		someObject.on( 'someEvent', function() { ... } );
			 *		someObject.fire( 'someEvent' );				// Both listeners are called.
			 *
			 *		someObject.on( 'someEvent', function( event ) {
			 *			alert( event.data );					// 'Example'
			 *		} );
			 *		someObject.fire( 'someEvent', 'Example' );
			 *
			 * @method
			 * @param {String} eventName The event name to fire.
			 * @param {Object} [data] Data to be sent as the
			 * {@link CKEDITOR.eventInfo#data} when calling the listeners.
			 * @param {CKEDITOR.editor} [editor] The editor instance to send as the
			 * {@link CKEDITOR.eventInfo#editor} when calling the listener.
			 * @returns {Boolean/Object} A boolean indicating that the event is to be
			 * canceled, or data returned by one of the listeners.
			 */
			fire: ( function() {
				// Create the function that marks the event as stopped.
				var stopped = 0;
				var stopEvent = function() {
						stopped = 1;
					};
				// Create the function that marks the event as canceled.
				var canceled = 0;
				var cancelEvent = function() {
						canceled = 1;
					};
				return function( eventName, data, editor ) {
					// Get the event entry.
					var event = getPrivate( this )[ eventName ];
					// Save the previous stopped and cancelled states. We may
					// be nesting fire() calls.
					var previousStopped = stopped,
						previousCancelled = canceled;
					// Reset the stopped and canceled flags.
					stopped = canceled = 0;
					if ( event ) {
						var listeners = event.listeners;
						if ( listeners.length ) {
							// As some listeners may remove themselves from the
							// event, the original array length is dinamic. So,
							// let's make a copy of all listeners, so we are
							// sure we'll call all of them.
							listeners = listeners.slice( 0 );
							var retData;
							// Loop through all listeners.
							for ( var i = 0; i < listeners.length; i++ ) {
								// Call the listener, passing the event data.
								if ( event.errorProof ) {
									try {
										retData = listeners[ i ].call( this, editor, data, stopEvent, cancelEvent );
									} catch ( er ) {}
								} else {
									retData = listeners[ i ].call( this, editor, data, stopEvent, cancelEvent );
								}
								if ( retData === false )
									canceled = 1;
								else if ( typeof retData != 'undefined' )
									data = retData;
								// No further calls is stopped or canceled.
								if ( stopped || canceled )
									break;
							}
						}
					}
					var ret = canceled ? false : ( typeof data == 'undefined' ? true : data );
					// Restore the previous stopped and canceled states.
					stopped = previousStopped;
					canceled = previousCancelled;
					return ret;
				};
			} )(),
			/**
			 * Fires an specific event in the object, releasing all listeners
			 * registered to that event. The same listeners are not called again on
			 * successive calls of it or of {@link #fire}.
			 *
			 *		someObject.on( 'someEvent', function() { ... } );
			 *		someObject.fire( 'someEvent' );			// Above listener called.
			 *		someObject.fireOnce( 'someEvent' );		// Above listener called.
			 *		someObject.fire( 'someEvent' );			// No listeners called.
			 *
			 * @param {String} eventName The event name to fire.
			 * @param {Object} [data] Data to be sent as the
			 * {@link CKEDITOR.eventInfo#data} when calling the listeners.
			 * @param {CKEDITOR.editor} [editor] The editor instance to send as the
			 * {@link CKEDITOR.eventInfo#editor} when calling the listener.
			 * @returns {Boolean/Object} A booloan indicating that the event is to be
			 * canceled, or data returned by one of the listeners.
			 */
			fireOnce: function( eventName, data, editor ) {
				var ret = this.fire( eventName, data, editor );
				delete getPrivate( this )[ eventName ];
				return ret;
			},
			/**
			 * Unregisters a listener function from being called at the specified
			 * event. No errors are thrown if the listener has not been registered previously.
			 *
			 *		var myListener = function() { ... };
			 *		someObject.on( 'someEvent', myListener );
			 *		someObject.fire( 'someEvent' );					// myListener called.
			 *		someObject.removeListener( 'someEvent', myListener );
			 *		someObject.fire( 'someEvent' );					// myListener not called.
			 *
			 * @param {String} eventName The event name.
			 * @param {Function} listenerFunction The listener function to unregister.
			 */
			removeListener: function( eventName, listenerFunction ) {
				// Get the event entry.
				var event = getPrivate( this )[ eventName ];
				if ( event ) {
					var index = event.getListenerIndex( listenerFunction );
					if ( index >= 0 )
						event.listeners.splice( index, 1 );
				}
			},
			/**
			 * Remove all existing listeners on this object, for cleanup purpose.
			 */
			removeAllListeners: function() {
				var events = getPrivate( this );
				for ( var i in events )
					delete events[ i ];
			},
			/**
			 * Checks if there is any listener registered to a given event.
			 *
			 *		var myListener = function() { ... };
			 *		someObject.on( 'someEvent', myListener );
			 *		alert( someObject.hasListeners( 'someEvent' ) );	// true
			 *		alert( someObject.hasListeners( 'noEvent' ) );		// false
			 *
			 * @param {String} eventName The event name.
			 * @returns {Boolean}
			 */
			hasListeners: function( eventName ) {
				var event = getPrivate( this )[ eventName ];
				return ( event && event.listeners.length > 0 );
			}
		};
	} )();
}
```