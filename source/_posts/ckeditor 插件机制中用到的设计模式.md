---
"title": "ckeditor 插件机制中用到的设计模式"
"date": "2016/6/10 20:46:25"
categories:
- js
- ckeditor

tags:
- js
- ckeditor
---


### 代理模式

代理模式（Proxy），为其他对象提供一种代理以控制对这个对象的访问。

代理模式使得代理对象控制具体对象的引用。代理几乎可以是任何对象：文件，资源，内存中的对象，或者是一些难以复制的东西。

```javascript
/**
	 * Adds a command definition to the editor instance. Commands added with
	 * this function can be executed later with the <code>{@link #execCommand}</code> method.
	 *
	 * 		editorInstance.addCommand( 'sample', {
	 * 			exec: function( editor ) {
	 * 				alert( 'Executing a command for the editor name "' + editor.name + '"!' );
	 * 			}
	 * 		} );
	 *
	 * @param {String} commandName The indentifier name of the command.
	 * @param {CKEDITOR.commandDefinition} commandDefinition The command definition.
	 */
	addCommand: function( commandName, commandDefinition ) {
		commandDefinition.name = commandName.toLowerCase();
		var cmd = new CKEDITOR.command( this, commandDefinition );
		// Update command when mode is set.
		// This guarantees that commands added before first editor#mode
		// aren't immediately updated, but waits for editor#mode and that
		// commands added later are immediately refreshed, even when added
		// before instanceReady. #10103, #10249
		if ( this.mode )
			updateCommand( this, cmd );
		return this.commands[ commandName ] = cmd;
	},
```

代理模式一般适用于如下场合：

1. 远程代理，也就是为了一个对象在不同的地址空间提供局部代表，这样可以隐藏一个对象存在于不同地址空间的事实，就像web service里的代理类一样。
2. 虚拟代理，根据需要创建开销很大的对象，通过它来存放实例化需要很长时间的真实对象，比如浏览器的渲染的时候先显示问题，而图片可以慢慢显示（就是通过虚拟代理代替了真实的图片，此时虚拟代理保存了真实图片的路径和尺寸。
3. 安全代理，用来控制真实对象访问时的权限，一般用于对象应该有不同的访问权限。
4. 智能指引，只当调用真实的对象时，代理处理另外一些事情。例如C#里的垃圾回收，使用对象的时候会有引用次数，如果对象没有引用了，GC就可以回收它了。

### 命令模式

命令模式(Command Pattern)：将一个请求封装为一个对象，从而使我们可用不同的请求对客户进行参数化；对请求排队或者记录请求日志，以及支持可撤销的操作。命令模式是一种对象行为型模式，其别名为动作(Action)模式或事务(Transaction)模式。
抽象命令类中声明了用于执行请求的execute()等方法，通过这些方法可以调用请求接收者的相关操作；

```javascript

CKEDITOR.command = function( editor, commandDefinition ) {
	/**
	 * Lists UI items that are associated to this command. This list can be
	 * used to interact with the UI on command execution (by the execution code
	 * itself, for example).
	 *
	 *		alert( 'Number of UI items associated to this command: ' + command.uiItems.length );
	 */
	this.uiItems = [];
	/**
	 * Executes the command.
	 *
	 *		command.exec(); // The command gets executed.
	 *
	 * **Note:** You should use the {@link CKEDITOR.editor#execCommand} method instead of calling
	 * `command.exec()` directly.
	 *
	 * @param {Object} [data] Any data to pass to the command. Depends on the
	 * command implementation and requirements.
	 * @returns {Boolean} A boolean indicating that the command has been successfully executed.
	 */
	this.exec = function( data ) {
		if ( this.state == CKEDITOR.TRISTATE_DISABLED || !this.checkAllowed() )
			return false;
		if ( this.editorFocus ) // Give editor focus if necessary (#4355).
			editor.focus();
		if ( this.fire( 'exec' ) === false )
			return true;
		return ( commandDefinition.exec.call( this, editor, data ) !== false );
	};
}

```

具体命令类是抽象命令类的子类，实现了在抽象命令类中声明的方法，它对应具体的接收者对象，将接收者对象的动作绑定其中；

```javascript

var redoCommand = editor.addCommand( 'redo', {
				exec: function() {
					if ( undoManager.redo() ) {
						editor.selectionChange();
						this.fire( 'afterRedo' );
					}
				},
				startDisabled: true,
				canUndo: false
			} );

```

调用者即请求的发送者，又称为请求者，它通过命令对象来执行请求；


```javascript

CKEDITOR.tools.callFunction(103,this);
```

接收者执行与请求相关的操作，它具体实现对请求的业务处理。

```javascript
this.exec = function( data ) {
		if ( this.state == CKEDITOR.TRISTATE_DISABLED || !this.checkAllowed() )
			return false;
		if ( this.editorFocus ) // Give editor focus if necessary (#4355).
			editor.focus();
		if ( this.fire( 'exec' ) === false )
			return true;
		return ( commandDefinition.exec.call( this, editor, data ) !== false );
	};
```

在命令模式中，将一个请求封装为一个对象，从而使我们可用不同的请求对客户进行参数化；对请求排队或者记录请求日志，以及支持可撤销的操作。命令模式是一种对象行为型模式，其别名为动作模式或事务模式。
命令模式包含四个角色：抽象命令类中声明了用于执行请求的execute()等方法，通过这些方法可以调用请求接收者的相关操作；具体命令类是抽象命令类的子类，实现了在抽象命令类中声明的方法，它对应具体的接收者对象，将接收者对象的动作绑定其中；调用者即请求的发送者，又称为请求者，它通过命令对象来执行请求；接收者执行与请求相关的操作，它具体实现对请求的业务处理。
命令模式的本质是对命令进行封装，将发出命令的责任和执行命令的责任分割开。命令模式使请求本身成为一个对象，这个对象和其他对象一样可以被存储和传递。
命令模式的主要优点在于降低系统的耦合度，增加新的命令很方便，而且可以比较容易地设计一个命令队列和宏命令，并方便地实现对请求的撤销和恢复；其主要缺点在于可能会导致某些系统有过多的具体命令类。
命令模式适用情况包括：需要将请求调用者和请求接收者解耦，使得调用者和接收者不直接交互；需要在不同的时间指定请求、将请求排队和执行请求；需要支持命令的撤销操作和恢复操作，需要将一组操作组合在一起，即支持宏命令。

