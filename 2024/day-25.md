**Author:** Tuna

---

# Command Pattern

The Command pattern is a way to turn a request into a standalone object. Instead of directly telling your program “do this,” you create an object that contains all the information needed to perform that action later.

![Command pattern icon](https://iamtuna.org/assets/apple-touch-icon.png)

Source: https://iamtuna.org/2024-12-22/command-pattern

---

Most developers are familiar with writing code that runs right away - you call a function and it does something immediately. But sometimes, we need a more flexible way to handle actions in our programs. This is where the Command pattern comes in.

While it’s one of the most widely used design patterns in software, many developers overlook it because it works differently than what they’re used to. Instead of the usual direct approach of “do this now,” the Command pattern introduces a way to package actions into objects that can be used later, modified, or even undone.

### What is the Command pattern?

The Command pattern is a way to turn a request into a standalone object. Instead of directly telling your program “do this,” you create an object that contains all the information needed to perform that action later. This object knows what needs to be done and how to do it.

Here’s a simple conceptual example to illustrate the Command pattern:

```ts
// Command interface
interface Command {
  execute(): void;
}

// Receiver
class Document {
  save() {
    console.log("Document saved");
  }
}

// Concrete Command
class SaveDocumentCommand implements Command {
  constructor(private doc: Document) {}

  execute() {
    this.doc.save();
  }
}

// Invoker
class Button {
  constructor(private command: Command) {}

  click() {
    this.command.execute();
  }
}

// Usage
const doc = new Document();
const saveCommand = new SaveDocumentCommand(doc);
const saveButton = new Button(saveCommand);

saveButton.click(); // "Document saved"
```

---

#### Why we often miss this pattern

The Command pattern is actually all around us in software development. Take a common example: handling a mouse click in your application. When a user clicks a button, the UI framework creates an event - and that’s actually the Command pattern in action! But most developers don’t recognize it because of how we typically handle these events. Usually, we write code that either:

- Responds to the click immediately right there in the event handler
- Passes the event up to a parent component through callbacks or event bubbling

This immediate or direct handling makes the Command pattern “invisible.” We don’t see that the click event is actually a command object containing information about what happened (like click position, which mouse button was used, and when it occurred). We just process it and move on.

---

### Drawbacks of the Command Pattern

Before we dive into the benefits of the Command pattern, let’s discuss some of its drawbacks. Like any design pattern, the Command pattern isn’t a one-size-fits-all solution. It has its own set of trade-offs that you should consider before using it in your projects.

#### Unintuitive Indirect Handling

When we handle actions indirectly through commands, the code becomes less straightforward to follow. Instead of seeing a direct connection between trigger and action like:

```text
button.click() -> saveData()
```

you have to trace through multiple layers: the command creation, where it’s stored, and where it’s finally executed. This can make code harder to understand, especially for developers who are new to the codebase or the pattern itself.

#### Limited Context and Information Loss

Because a command is a data model object, which means it can be retained long before it’s handled, even stored within the persistent storage. In another word, a command’s lifetime can be longer than the source that emit the command itself. Therefore, when we package an action into a command object, we need to be careful about what information we include to avoid memory leaks. This means:

- We can’t keep a direct reference to the source object  
- Some context about where and why the action was triggered gets lost  
- Adding more context means larger command objects and more memory usage  
- Debugging becomes harder because the stack trace doesn’t show the complete picture  

#### Additional Complexity

There are a few more drawbacks worth considering:

- **More boilerplate code:** You need to create separate command classes for different actions  
- **Increased memory usage:** Each action becomes an object, which uses more memory than direct method calls  
- **Potential performance overhead:** The indirect nature of command execution adds a small performance cost  
- **Command management:** You need additional infrastructure to handle command objects (storing them, executing them, cleaning them up)  

---

### Benefits of the Command Pattern

Despite these drawbacks, the Command pattern offers several benefits that make it a valuable tool in your design patterns toolbox.

#### Flexible Execution Control

The Command pattern transforms how we can handle program actions. Rather than executing everything immediately, we gain the power to decide when and how actions run. This opens up possibilities like queuing tasks for later, running multiple commands as a batch for better performance, or even spreading work across different processes. This flexibility becomes particularly valuable in complex applications where timing and resource management are crucial.

#### Undo/Redo Functionality

Perhaps the most celebrated benefit of the Command pattern is its elegant support for undo and redo operations. Since each action is captured as an object, we can keep track of everything that happens. Each command can know not only how to perform its action but also how to reverse it.

What’s particularly interesting is how one of the pattern’s apparent drawbacks - losing detailed context about the source - actually becomes a strength for undo/redo implementation. Since commands are self-contained and don’t maintain references to their original triggers, the undo system becomes significantly simpler. We don’t need to worry about whether the original button still exists, if the user who initiated the action is still logged in, or if the triggering component has been destroyed. The command object contains exactly what it needs to perform or reverse the action on the main state of the application, nothing more.

#### Better Testing and Debugging

Converting actions into command objects creates new opportunities for testing and debugging. We can inspect commands before they run, create mock versions for testing, and even record sequences of commands to reproduce bugs. This visibility into what’s happening makes it much easier to understand and fix issues in complex applications. You can even replay a user’s exact actions to see what led to a problem.

#### Decoupling and Modularity

The Command pattern naturally encourages better code organization by separating the code that requests an action from the code that performs it. This separation makes your application more flexible and easier to maintain. You can change how commands are handled without touching the code that creates them, or add new types of commands without disrupting existing functionality. This becomes especially valuable in large applications where different teams might work on different parts of the system.

Decoupling also helps reduce the number of public APIs we need to expose from a component, or this is one tool to make the module deeper (see *Chapter 4. Modules Should Be Deep - A Philosophy of Software Design*). In the diagram at the beginning of this article, the:

```text
handle(action)
```

method is the only public API we need to expose. This method can handle multiple types of actions, so we don’t need to create separate methods for each action type. This reduces the surface area of our public API and makes it easier to manage and maintain.

#### Transaction Management

In systems that need to handle complex operations reliably, the Command pattern shines. By treating actions as objects, we can group them into transactions, ensure they all succeed or all fail together, and even implement retry mechanisms for failed operations. This becomes particularly valuable in financial systems or anywhere else where data consistency is crucial. If something goes wrong halfway through a complex operation, you can roll back all the previous steps to keep your system in a consistent state.

---

### Ending thoughts

The Command pattern is powerful and you can see it almost in every application you use, from Database transactions to the Undo/Redo feature in your text editor. It’s a great tool to have in your design patterns toolbox, but like any tool, it’s not always the right one for the job. Use it wisely and you’ll find that it can simplify complex problems and make your code more flexible and maintainable.

The implementation of the Command pattern can vary depending on the purpose of your use-case. It can be simple as turning multiple public methods into a single method that accepts an action type, or as complex as implementing a full-fledged undo/redo system. The key is to understand the benefits and drawbacks of the pattern and decide if it’s the right fit for your project.

This writing does not include how to implement the Command pattern, you can find many resources online that explain how to implement it in different use-cases. I hope this writing gives you an overview of the benefits and drawbacks of the pattern and hope it nails down the concept of the Command pattern in your mind.

---

### Read more

- [Command pattern - Refactoring Guru](https://refactoring.guru/design-patterns/command)  
- [Command pattern - Wikipedia](https://en.wikipedia.org/wiki/Command_pattern)  
- [A Philosophy of Software Design - John Ousterhout](https://www.goodreads.com/book/show/58665335)