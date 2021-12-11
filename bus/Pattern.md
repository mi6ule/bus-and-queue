# Bus Pattern
The only one question below always exists in our head: How to separate the messages in our system?
#### command-bus
![[command-bus.png]]
In this pattern, there are three classes that is utilized.

1.  `Command` class
    
    Command class is only a class that contains the needed data to execute our action. It is like the Data Transfer Object(`DTO`) pattern.
    
    The relationship between Command and CommandHandler is **one-to-one**. It means that one Command will only be handled by one CommandHandler.
    
    In Command class, we can do some simple validation for its data, and Command object is an *immutable object*.
    
2.  `CommandHandler` class
    
    CommandHandler’s responsibility is to execute the appropriate domain behavior when taking a specific Command object as input. **The CommandHandler should not be doing any domain logic itself.** If we need to do more than this, we should define a ***service*** to wrap that logic.
    
    The domain behavior should be executed in an AggregateRoot. So, CommandHandler class will load an `AggregateRoot` by using the repository, and pass its data to an AggregateRoot.
	
    ![[aggrigate-root.jpg]]
3.  `CommandBus` class
    
    After receiving a Command object, CommandBus will route it to the suitable CommandHandler.
    
    To extend the functionality of CommandBus class, we can use `Decorator pattern` or `Proxy pattern` with it. In the case of Decorator pattern, there are some samples for CommandBus’s extension:
    
    -   Wrap CommandBus in a database transaction.
    -   Wrap CommandBus with logging, measuring time for CommandHandler.
    
    With Proxy pattern, wraps CommandBus to check permissions before handling our command.