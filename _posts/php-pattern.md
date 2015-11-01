---
layout: page
title:	设计模式
category: blog
description: 
---
# Preface

# Factory Pattern, 工厂模式

	interface IUser {
	  function getName();
	}

	class User implements IUser {
	  public function __construct( $id ) { }

	  public function getName() {
		return "Jack";
	  }

	  public static function Create( $id ) {
		return new User( $id );
	  }
	}

	class UserFactory {
	  public static function Create( $id ) {
		return new User( $id );
	  }
	}

	$uo = UserFactory::Create( 1 );
	//$uo = User::Create( 1 );
	echo( $uo->getName()."\n" );

# Observer Pattern, 观察者模式
观察者模式为您提供了避免组件之间紧密耦合的另一种方法。该模式非常简单：一个对象通过添加一个方法（该方法允许另一个对象，即观察者 注册自己）使本身变得可观察。当可观察的对象更改时，它会将消息发送到已注册的观察者。这些观察者使用该信息执行的操作与可观察的对象无关。结果是对象可以相互对话，而不必了解原因。
一个简单示例是系统中的用户列表。清单 4 中的代码显示一个用户列表，添加用户时，它将发送出一条消息。添加用户时，通过发送消息的日志观察者可以观察此列表。

	class UserList implements IObservable {
	  private $_observers = array();

	  public function addCustomer( $name ) {
		foreach( $this->_observers as $obs )
		  $obs->onChanged( $this, $name );
	  }

	  public function addObserver( $observer ) {
		$this->_observers []= $observer;
	  }
	}

	class UserListLogger implements IObserver {
	  public function onChanged( $sender, $args ) {
		echo( "'$args' added to user list\n" );
	  }
	}

	$ul = new UserList();
	$ul->addObserver( new UserListLogger() );
	$ul->addCustomer( "Jack" );

# Strategy Pattern, 策略模式
下例中，通过使用策略模式，可将`过滤部分`放入另一个类中，而不影响用户列表中的其余代码。

	class FindAfterStrategy implements IStrategy {
	  private $_name;

	  public function __construct( $name ) {
		$this->_name = $name;
	  }

	  public function filter( $record ) {
		return strcmp( $this->_name, $record ) <= 0;
	  }
	}

	class RandomStrategy implements IStrategy {
	  public function filter( $record ) {
		return rand( 0, 1 ) >= 0.5;
	  }
	}

	class UserList {
	  private $_list = array();

	  public function __construct( $names ) {
		if ( $names != null ) {
		  foreach( $names as $name ) {
			$this->_list []= $name;
		  }
		}
	  }

	  public function add( $name ) {
		$this->_list []= $name;
	  }

	  public function find( $filter ) {
		$recs = array();
		foreach( $this->_list as $user ) {
		  if ( $filter->filter( $user ) )
			$recs []= $user;
		}
		return $recs;
	  }
	}

	$ul = new UserList( array( "Andy", "Jack", "Lori", "Megan" ) );
	$f1 = $ul->find( new FindAfterStrategy( "J" ) );
	print_r( $f1 );

	$f2 = $ul->find( new RandomStrategy() );
	print_r( $f2 );

# CommandChain Pattern, 命令链模式
命令链 模式以松散耦合主题为基础，发送消息、命令和请求，或通过一组处理程序发送任意内容。每个处理程序都会自行判断自己能否处理请求。如果可以，该请求被处理，进程停止。您可以为系统添加或移除处理程序，而不影响其他处理程序。清单 5 显示了此模式的一个示例。

	class CommandChain {
	  private $_commands = array();

	  public function addCommand( $cmd ) {
		$this->_commands []= $cmd;
	  }

	  public function runCommand( $name, $args ) {
		foreach( $this->_commands as $cmd ) {
		  if ( $cmd->onCommand( $name, $args ) )
			return;
		}
	  }
	}

	class UserCommand implements ICommand {
	  public function onCommand( $name, $args ) {
		if ( $name != 'addUser' ) return false;
		echo( "UserCommand handling 'addUser'\n" );
		return true;
	  }
	}

	class MailCommand implements ICommand {
	  public function onCommand( $name, $args ) {
		if ( $name != 'mail' ) return false;
		echo( "MailCommand handling 'mail'\n" );
		return true;
	  }
	}

	$cc = new CommandChain();
	$cc->addCommand( new UserCommand() );
	$cc->addCommand( new MailCommand() );
	$cc->runCommand( 'addUser', null );
	$cc->runCommand( 'mail', null );

# Reference
- [php-pattern]

[php-pattern]: http://www.ibm.com/developerworks/cn/opensource/os-php-designptrns/
