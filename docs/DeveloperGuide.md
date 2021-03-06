# ToDoApp - Developer Guide

By : `CS2103JAN2017-F12-B2`  &nbsp;&nbsp;&nbsp;&nbsp; Since: `Mar 2017`  &nbsp;&nbsp;&nbsp;&nbsp; Licence: `MIT`

---

1. [Setting Up](#setting-up)
2. [Design](#design)
3. [Implementation](#implementation)
4. [Testing](#testing)
5. [Dev Ops](#dev-ops)

* [Appendix A: User Stories](#appendix-a--user-stories)
* [Appendix B: Use Cases](#appendix-b--use-cases)
* [Appendix C: Non Functional Requirements](#appendix-c--non-functional-requirements)
* [Appendix D: Glossary](#appendix-d--glossary)
* [Appendix E : Product Survey](#appendix-e--product-survey)


## 1. Setting up

### 1.1. Prerequisites

1. **JDK `1.8.0_60`**  or later<br>

    > Having any Java 8 version is not enough. <br>
    This app will not work with earlier versions of Java 8.

2. **Eclipse** IDE
3. **e(fx)clipse** plugin for Eclipse (Do the steps 2 onwards given in
   [this page](http://www.eclipse.org/efxclipse/install.html#for-the-ambitious))
4. **Buildship Gradle Integration** plugin from the Eclipse Marketplace
5. **Checkstyle Plug-in** plugin from the Eclipse Marketplace


### 1.2. Importing the project into Eclipse

0. Fork this repo, and clone the fork to your computer
1. Open Eclipse (Note: Ensure you have installed the **e(fx)clipse** and **buildship** plugins as given
   in the prerequisites above)
2. Click `File` > `Import`
3. Click `Gradle` > `Gradle Project` > `Next` > `Next`
4. Click `Browse`, then locate the project's directory
5. Click `Finish`

  > * If you are asked whether to 'keep' or 'overwrite' config files, choose to 'keep'.
  > * Depending on your connection speed and server load, it can even take up to 30 minutes for the set up to finish
      (This is because Gradle downloads library files from servers during the project set up process)
  > * If Eclipse auto-changed any settings files during the import process, you can discard those changes.

### 1.3. Configuring Checkstyle
1. Click `Project` -> `Properties` -> `Checkstyle` -> `Local Check Configurations` -> `New...`
2. Choose `External Configuration File` under `Type`
3. Enter an arbitrary configuration name e.g. ToDoApp
4. Import checkstyle configuration file found at `config/checkstyle/checkstyle.xml`
5. Click OK once, go to the `Main` tab, use the newly imported check configuration.
6. Tick and select `files from packages`, click `Change...`, and select the `resources` package
7. Click OK twice. Rebuild project if prompted

> Note to click on the `files from packages` text after ticking in order to enable the `Change...` button

### 1.4. Troubleshooting project setup

**Problem: Eclipse reports compile errors after new commits are pulled from Git**

* Reason: Eclipse fails to recognize new files that appeared due to the Git pull.
* Solution: Refresh the project in Eclipse:<br>
  Right click on the project (in Eclipse package explorer), choose `Gradle` -> `Refresh Gradle Project`.

**Problem: Eclipse reports some required libraries missing**

* Reason: Required libraries may not have been downloaded during the project import.
* Solution: [Run tests using Gradle](UsingGradle.md) once (to refresh the libraries).


## 2. Design

### 2.1. Architecture

<img src="images/Architecture.png" width="600"><br>
_Figure 2.1.1 : Architecture Diagram_

The **_Architecture Diagram_** given above explains the high-level design of the App.
Given below is a quick overview of each component.

> Tip: The `.pptx` files used to create diagrams in this document can be found in the [diagrams](diagrams/) folder.
> To update a diagram, modify the diagram in the pptx file, select the objects of the diagram, and choose `Save as picture`.

`Main` has only one class called [`MainApp`](../src/main/java/seedu/todoapp/MainApp.java). It is responsible for,

* At app launch: Initializes the components in the correct sequence, and connects them up with each other.
* At shut down: Shuts down the components and invokes cleanup method where necessary.

[**`Commons`**](#common-classes) represents a collection of classes used by multiple other components.
Two of those classes play important roles at the architecture level.

* `EventsCenter` : This class (written using [Google's Event Bus library](https://github.com/google/guava/wiki/EventBusExplained))
  is used by components to communicate with other components using events (i.e. a form of _Event Driven_ design)
* `LogsCenter` : Used by many classes to write log messages to the App's log file.

The rest of the App consists of four components.

* [**`UI`**](#ui-component) : The UI of the App.
* [**`Logic`**](#logic-component) : The command executor.
* [**`Model`**](#model-component) : Holds the data of the App in-memory.
* [**`Storage`**](#storage-component) : Reads data from, and writes data to, the hard disk.

Each of the four components

* Defines its _API_ in an `interface` with the same name as the Component.
* Exposes its functionality using a `{Component Name}Manager` class.

For example, the `Logic` component (see the class diagram given below) defines it's API in the `Logic.java`
interface and exposes its functionality using the `LogicManager.java` class.<br>
<img src="images/LogicClassDiagram.png" width="800"><br>
_Figure 2.1.2 : Class Diagram of the Logic Component_

#### Events-Driven nature of the design

The _Sequence Diagram_ below shows how the components interact for the scenario where the user issues the
command `delete 1`.

<img src="images\SDforDeletePerson.png" width="800"><br>
_Figure 2.1.3a : Component interactions for `delete 1` command (part 1)_

>Note how the `Model` simply raises a `ToDoAppChangedEvent` when the ToDoApp data are changed,
 instead of asking the `Storage` to save the updates to the hard disk.

The diagram below shows how the `EventsCenter` reacts to that event, which eventually results in the updates
being saved to the hard disk and the status bar of the UI being updated to reflect the 'Last Updated' time. <br>
<img src="images\SDforDeletePersonEventHandling.png" width="800"><br>
_Figure 2.1.3b : Component interactions for `delete 1` command (part 2)_

> Note how the event is propagated through the `EventsCenter` to the `Storage` and `UI` without `Model` having
  to be coupled to either of them. This is an example of how this Event Driven approach helps us reduce direct
  coupling between components.

<!-- @@author A0114395E -->
The Activity Diagram below shows the flow when a Recurring Task exists in ToDoApp<br><br>
<img src="images/ToDoApp_Recurring-Activity-Diagram.png" width="600"><br>
_Figure 2.1.5 : Component interactions for commands_
> Note how if a command has tags `daily`, `weekly`, `monthly`, `yearly`, it will qualify as a recurrent task.
> RecurrentTaskManager, implemented as a singleton, will check the whole list whenever there's a new command, and move expired task to its' next expected schedule.

The Activity Diagram below shows the flow when a Command is being executed in ToDoApp<br><br>
<img src="images/ToDoApp_Activity-Diagram.png" width="600"><br>
_Figure 2.1.6 : Component interactions for commands_

The Sequence Diagram below shows the flow when `add`, `delete`, `edit`, `mark`, `unmark`, `clear`  Command is being executed in ToDoApp<br><br>
<img src="images/ToDoApp_Seq-Diag-AddDelEdit.png" width="800"><br>
_Figure 2.1.7 : Sequence diagram for commands_

> Note how if a command is `add`, `delete`, `edit`, `mark`, `unmark`, `clear` we will parse the inverse of it's command to be stored as well.

The Sequence Diagram below shows how ToDoApp handles `undo` and `redo` requests from the user.<br><br>
<img src="images/ToDoApp_Seq-Diag-UndoRedo.png" width="800"><br>
_Figure 2.1.8 : Sequence diagram for `undo` & `redo` commands_

> Note how `StateManager` is implemented as a Singleton. This is by design, as we do not want more than one instance of a `StateManager` to handle undo/redo states. The `StateManager` consists of 2 stacks - `undoStack` & `redoStack`, holding a `StateCommandPair` object. <br>
> The `StateCommandPair` class contains of 2 commands. The Command itself, and the inverse of it's Command. The inverse command is evaluated during the parsing of the actual command. <br><br>
> When `undo` command is invoked, we pop the `StateCommandPair` from `undoStack` and put it on `redoStack`. We invoke the `undoCommand` from the `StateCommandPair`. <br>
> Vice versa, when `redo` command is invoked, we pop the `StateCommandPair` from `redoStack` and put it on the `undoStack`. We then invoke the `executeCommand` from `StateCommandPair`. <br><br>
> When the user performs any action, the redo stack is cleared. <br>
> Undoability and Redoability are defined by whether either stack is empty, as well as if an action is undo-able, i.e only `add`, `edit`, `delete`, `mark`, `unmark`, `clear` commands.

he Sequence Diagram below shows how ToDoApp handles the event where a user `select` or clicks on a task <br><br>
<img src="images/ToDoApp_Seq-Diag-GMaps.png" width="800"><br>
_Figure 2.1.9 : Sequence diagram for `select` / click on task_
> On user interaction, whether it being a click or a `select` command, `BrowserPanel` will retrieve the venue from the Task, and does a Google search for the location of the venue and render the webpage accordingly.
<!-- @@author -->

The sections below give more details of each component.

### 2.2. UI component

Author: Alice Bee

<img src="images/UiClassDiagram.png" width="800"><br>
_Figure 2.2.1 : Structure of the UI Component_

**API** : [`Ui.java`](../src/main/java/seedu/todoapp/ui/Ui.java)

The UI consists of a `MainWindow` that is made up of parts e.g.`CommandBox`, `ResultDisplay`, `PersonListPanel`,
`StatusBarFooter`, `BrowserPanel` etc. All these, including the `MainWindow`, inherit from the abstract `UiPart` class.

The `UI` component uses JavaFx UI framework. The layout of these UI parts are defined in matching `.fxml` files
 that are in the `src/main/resources/view` folder.<br>
 For example, the layout of the [`MainWindow`](../src/main/java/seedu/todoapp/ui/MainWindow.java) is specified in
 [`MainWindow.fxml`](../src/main/resources/view/MainWindow.fxml)

The `UI` component,

* Executes user commands using the `Logic` component.
* Binds itself to some data in the `Model` so that the UI can auto-update when data in the `Model` change.
* Responds to events raised from various parts of the App and updates the UI accordingly.

### 2.3. Logic component

Author: Bernard Choo

<img src="images/LogicClassDiagram.png" width="800"><br>
_Figure 2.3.1 : Structure of the Logic Component_

**API** : [`Logic.java`](../src/main/java/seedu/todoapp/logic/Logic.java)

1. `Logic` uses the `Parser` class to parse the user command.
2. This results in a `Command` object which is executed by the `LogicManager`.
3. The command execution can affect the `Model` (e.g. adding a person) and/or raise events.
4. The result of the command execution is encapsulated as a `CommandResult` object which is passed back to the `Ui`.

Given below is the Sequence Diagram for interactions within the `Logic` component for the `execute("delete 1")`
 API call.<br>
<img src="images/DeleteTaskSdForLogic.png" width="800"><br>
_Figure 2.3.1 : Interactions Inside the Logic Component for the `delete 1` Command_

### 2.4. Model component

Author: Cynthia Dharman

<img src="images/ModelClassDiagram.png" width="800"><br>
_Figure 2.4.1 : Structure of the Model Component_

**API** : [`Model.java`](../src/main/java/seedu/todoapp/model/Model.java)

The `Model`,

* stores a `UserPref` object that represents the user's preferences.
* stores the ToDoApp data.
* exposes a `UnmodifiableObservableList<ReadOnlyPerson>` that can be 'observed' e.g. the UI can be bound to this list
  so that the UI automatically updates when the data in the list change.
* does not depend on any of the other three components.

### 2.5. Storage component

Author: Darius Foong

<img src="images/StorageClassDiagram.png" width="800"><br>
_Figure 2.5.1 : Structure of the Storage Component_

**API** : [`Storage.java`](../src/main/java/seedu/todoapp/storage/Storage.java)

The `Storage` component,

* can save `UserPref` objects in json format and read it back.
* can save the ToDoApp data in xml format and read it back.

### 2.6. Common classes

Classes used by multiple components are in the `seedu.todoapp.commons` package.

## 3. Implementation

### 3.1. Logging

We are using `java.util.logging` package for logging. The `LogsCenter` class is used to manage the logging levels
and logging destinations.

* The logging level can be controlled using the `logLevel` setting in the configuration file
  (See [Configuration](#configuration))
* The `Logger` for a class can be obtained using `LogsCenter.getLogger(Class)` which will log messages according to
  the specified logging level
* Currently log messages are output through: `Console` and to a `.log` file.

**Logging Levels**

* `SEVERE` : Critical problem detected which may possibly cause the termination of the application
* `WARNING` : Can continue, but with caution
* `INFO` : Information showing the noteworthy actions by the App
* `FINE` : Details that is not usually noteworthy but may be useful in debugging
  e.g. print the actual list instead of just its size

### 3.2. Configuration

Certain properties of the application can be controlled (e.g App name, logging level) through the configuration file
(default: `config.json`):

## 4. Testing

Tests can be found in the `./src/test/java` folder.

**In Eclipse**:

* To run all tests, right-click on the `src/test/java` folder and choose
  `Run as` > `JUnit Test`
* To run a subset of tests, you can right-click on a test package, test class, or a test and choose
  to run as a JUnit test.

**Using Gradle**:

* See [UsingGradle.md](UsingGradle.md) for how to run tests using Gradle.

We have two types of tests:

1. **GUI Tests** - These are _System Tests_ that test the entire App by simulating user actions on the GUI.
   These are in the `guitests` package.

2. **Non-GUI Tests** - These are tests not involving the GUI. They include,
   1. _Unit tests_ targeting the lowest level methods/classes. <br>
      e.g. `seedu.todoapp.commons.UrlUtilTest`
   2. _Integration tests_ that are checking the integration of multiple code units
     (those code units are assumed to be working).<br>
      e.g. `seedu.todoapp.storage.StorageManagerTest`
   3. Hybrids of unit and integration tests. These test are checking multiple code units as well as
      how the are connected together.<br>
      e.g. `seedu.todoapp.logic.LogicManagerTest`

#### Headless GUI Testing
Thanks to the [TestFX](https://github.com/TestFX/TestFX) library we use,
 our GUI tests can be run in the _headless_ mode.
 In the headless mode, GUI tests do not show up on the screen.
 That means the developer can do other things on the Computer while the tests are running.<br>
 See [UsingGradle.md](UsingGradle.md#running-tests) to learn how to run tests in headless mode.

### 4.1. Troubleshooting tests

 **Problem: Tests fail because NullPointException when AssertionError is expected**

 * Reason: Assertions are not enabled for JUnit tests.
   This can happen if you are not using a recent Eclipse version (i.e. _Neon_ or later)
 * Solution: Enable assertions in JUnit tests as described
   [here](http://stackoverflow.com/questions/2522897/eclipse-junit-ea-vm-option). <br>
   Delete run configurations created when you ran tests earlier.

## 5. Dev Ops

### 5.1. Build Automation

See [UsingGradle.md](UsingGradle.md) to learn how to use Gradle for build automation.

### 5.2. Continuous Integration

We use [Travis CI](https://travis-ci.org/) and [AppVeyor](https://www.appveyor.com/) to perform _Continuous Integration_ on our projects.
See [UsingTravis.md](UsingTravis.md) and [UsingAppVeyor.md](UsingAppVeyor.md) for more details.

### 5.3. Publishing Documentation

See [UsingGithubPages.md](UsingGithubPages.md) to learn how to use GitHub Pages to publish documentation to the
project site.

### 5.4. Making a Release

Here are the steps to create a new release.

 1. Generate a JAR file [using Gradle](UsingGradle.md#creating-the-jar-file).
 2. Tag the repo with the version number. e.g. `v0.1`
 2. [Create a new release using GitHub](https://help.github.com/articles/creating-releases/)
    and upload the JAR file you created.

### 5.5. Converting Documentation to PDF format

We use [Google Chrome](https://www.google.com/chrome/browser/desktop/) for converting documentation to PDF format,
as Chrome's PDF engine preserves hyperlinks used in webpages.

Here are the steps to convert the project documentation files to PDF format.

 1. Make sure you have set up GitHub Pages as described in [UsingGithubPages.md](UsingGithubPages.md#setting-up).
 1. Using Chrome, go to the [GitHub Pages version](UsingGithubPages.md#viewing-the-project-site) of the
    documentation file. <br>
    e.g. For [UserGuide.md](UserGuide.md), the URL will be `https://CS2103JAN2017-F12-B2.github.io/addressbook-level4/docs/UserGuide.html`.
 1. Click on the `Print` option in Chrome's menu.
 1. Set the destination to `Save as PDF`, then click `Save` to save a copy of the file in PDF format. <br>
    For best results, use the settings indicated in the screenshot below. <br>
    <img src="images/chrome_save_as_pdf.png" width="300"><br>
    _Figure 5.4.1 : Saving documentation as PDF files in Chrome_

### 5.6. Managing Dependencies

A project often depends on third-party libraries. For example, ToDoApp depends on the
[Jackson library](http://wiki.fasterxml.com/JacksonHome) for XML parsing. Managing these _dependencies_
can be automated using Gradle. For example, Gradle can download the dependencies automatically, which
is better than these alternatives.<br>
a. Include those libraries in the repo (this bloats the repo size)<br>
b. Require developers to download those libraries manually (this creates extra work for developers)<br>

<!-- @@author A0114395E -->
### 5.7. Git Workflow
Before each sprint, a staging branch will be prepared, named in the form of `master_V[VERSION_NUMBER]`. <br/>
New features will be worked on in a new branch, named in the manner `features/FEATURE_DESCRIPTION`. When complete, the developers will do a pull request into `master_V[VERSION_NUMBER]`, e.g. `master_V0.5`. <br/>
For release into production - we will merge `master_V[VERSION_NUMBER]` into `master` branch, and tag it as `V[VERSION_NUMBER]`.
<!-- @@author -->

## Appendix A : User Stories

Priorities: High (must have) - `* * *`, Medium (nice to have)  - `* *`,  Low (unlikely to have) - `*`


Priority | As a ... | I want to ... | So that I can...
-------- | :-------- | :--------- | :-----------
`* * *` | new user | see usage instructions | refer to instructions when I forget how to use the App
`* * *` | user | add a new task |
`* * *` | user | edit a new task | change information of a task
`* * *` | user | delete a task | remove tasks that I no longer need to track
`* * *` | user | list out all uncompleted tasks | To view outstanding tasks
`* * *` | user | block out a period of time | keep a period of time unavailable
`* * *` | user | release a blocked period of time | make a period of time available again
`* * *` | user | find upcoming tasks by date | can track of what is dued soon
`* * *` | user | mark a task as completed | differentiate between completed and uncompleted tasks
`* * *` | user | unmark a task as undone | differentiate between completed and uncompleted tasks
`* * *` | user | set a deadline to a task | easily keep track of deadline for a certain task to be completed
`* * *` | user | get more information about a command | learn how to use various commands
`* * *` | user with different kind of tasks| tag a task | so that I can add labels associated with the task
`* * *` | user | retrieve tasks by tag | so that I can see tasks associated with a label
`* * *` | user | retrieve tasks due on certain date | so that I can see tasks due on specified date
`* * *` | user | assign priority to tasks | so that I can keep track of the priority of tasks
`* * *` | user | retrieve tasks based on priority | so that I can see tasks ranked by priority
`* * *` | user | undo a command | correct mistakes
`* * *` | user | redo a command | correct mistakes
`* *` | advanced user | use shorter versions of a command to type faster | more quickly use the app
`* *` | complex user | break a task into subtasks | keep track of complex tasks
`* *` | frequent user | add a recurring task | keep track of task that needs to be done many times
`* *` | user | get a week view of tasks | plan for my week
`* *` | frequent user | access app with shortcut | quickly access the app
`* *` | user | change the reccurence setting of a task | update if the task is recurring
`* *` | user | specify the location of the storage file | save the data to the location of my preference
`* *` | user | import data from a specified file
`* ` | user | create a copy of task in google calendar | refer to tasks outside of app
`* ` | user | set task reminders | remind myself of certain tasks
`* ` | user | sync the  to my google calendar | sync the tasks with the calendar and get notified
`* ` | international user | specify the timezone of task | keep track of tasks due across different timezones

{More to be added}

## Appendix B : Use Cases

(For all use cases below, the **System** is the `TodoApp` and the **Actor** is the `user`, unless specified otherwise)

#### Use case: Add a task

**MSS**

1. User inputs command to add a new task
2. TodoApp adds said task, and shows the task's details
Use case ends

#### Use case: Delete a task

**MSS**

1. User requests to list tasks
2. TodoApp shows a list of tasks
3. User requests to delete a specific task in the list
4. TodoApp deletes the task
Use case ends

**Extensions**

2a. List is empty

> Use case ends

3a. Invalid index given to delete

> 3a1. TodoApp shows an error message
  Use case resumes at step 2

#### Use case: Edit a task

**MSS**

1. User requests to list tasks
2. TodoApp shows a list of tasks
3. User requests to edit the details of a specific task in the list
4. TodoApp edits the details of that task
Use case ends

**Extensions**

2a. List is empty

> Use case ends

3a. Invalid index given to edit

> 3a1. TodoApp shows an error message
  Use case resumes at step 2

3b. The attribute given is invalid

> 3b1. TodoApp shows an error message
  Use case resumes at step 2

3c. Value of the attribute is invalid

> 3c1. TodoApp shows an error message
  Use case resumes at step 2

3d. Value of the attribute is the same as the previous value
> 3d1. TodoApp does nothing

>Use case ends

<!-- @@author A0124591H -->

#### Use case: Mark or unmark a task

**MSS**

1. User requests to list tasks
2. TodoApp shows a list of tasks
3. User requests to mark/unmark a specific task in the list
4. TodoApp sets completion of the task to true/false

**Extensions**

2a. List is empty

> Use case ends

3a. Invalid index given to delete

> 3a1. TodoApp shows an error message
  Use case resumes at step 2

3b. Value of completion is the same as the previous value

> 3b1. TodoApp does nothing

>Use case ends

<!-- @@author A0124591H -->

#### Use case: Retrieve a task

**MSS**

1. User requests to retrieve tasks by name/priority/deadline/completion
2. TodoApp shows a list of tasks that matches
Use case ends

1a. List is empty

> Use case ends

1b. Given inputs not found

> Use case ends

<!-- @@author -->

#### Use case: List task

**MSS**

1. User input command to list task
2. TodoApp shows a list of tasks sorted by due dates
Use case ends

**Extensions**

2a. List is empty

> 2a1. TodoApp shows a message to indicate that the list is empty
Use case ends

#### Use case: Set deadline to a task

**MSS**

1. User requests to list tasks
2. TodoApp shows a list of tasks
3. User requests to set a deadline to a specific task
4. TodoApp sets a deadline for the task and displays the task
Use case ends

**Extensions**

2a. List is empty

> Use case ends

3a. The given index is invalid

> 3a1. TodoApp shows an error message
Use case resumes at step 2

3b. The date input by user is invalid

> 3b1. TodoApp shows an error message
Use case resumes at step 2

#### Use case: Undo a command

**MSS**

1. User does `add`, `edit`, `delete`, `mark`, `unmark`, `clear` command
2. TodoApp executes command
3. User wants to undo previous command, executes `undo`
4. ToDoApp reverts the command
5. Use case ends

**Extensions**

3a. No command to revert

> Use case ends

#### Use case: Redo a command

**MSS**

1. User executes an `undo` command
2. User wants to undo previous `undo` command, executes `redo`
4. ToDoApp undoes the `undo` command
5. Use case ends

**Extensions**

2a. No command to redo

> Use case ends

<!-- @@author A0124591H -->

#### Use case: Specify a new path command

**MSS**

1. User executes a `cd` command
2. ToDoApp creates a clone of the data file in specified path
3. Use case ends

**Extensions**

1a. Invalid file path

> 1a1. ToDoApp shows an error message

> Use case ends

<!-- @@author A0124591H -->

#### Use case: Import command

**MSS**

1. User executes an `import` command
2. ToDoApp imports data from specified file into current ToDoApp
3. Use case ends

**Extensions**

1a. Invalid file path

> 1a1. ToDoApp shows an error message

> Use case ends

## Appendix C : Non Functional Requirements

1. Should work on any [mainstream OS](#mainstream-os) as long as it has Java `1.8.0_60` or higher installed.
2. Should be able to hold up to 1000 tasks without a noticeable sluggishness in performance for typical usage.
3. A user with above average typing speed for regular English text (i.e. not code, not system admin commands)
   should be able to accomplish most of the tasks faster using commands than using the mouse.
4. Should include well-written guides for both users and developers
5. Should follow good OOP principles
6. Should be integrated with automated tests

{More to be added}

## Appendix D : Glossary

##### Mainstream OS

> Windows, Linux, Unix, OS-X

##### Private contact detail

> A contact detail that is not meant to be shared with others

## Appendix E : Product Survey

**Trello**

Author: Lim Jing Rong

Pros:

* Simple to use
* Intuitive UI
* Cloud support
* Good integration (Github, etc)
* Modular and flexible based on user
* Can be fitlered by tags & users

Cons:

* Tasks have to be small enough to be described in a few words
* Okay for specific projects/todo lists. Bad if you have multiple 'big projects'

**Wunderlist**

Author: Lim Jing Rong

Pros:

* Straight forward to use
* Multi-platform (Desktop application, Chrome ext, mobile app, web app, etc)
* Cloud support
* Great UI/ graphics
* Free
* Good search

Cons:

* Not much integrations
* No options for subtasks
* No repeat options

<!-- @@author A0124591H -->

**Google Calendar**

Author: Lim Huan Hock

Pros:

* Straight forward to use
* Multi-platform (Mobile app, web app, etc)
* Cloud support
* Free
* API-friendly

Cons:

* No options for subtasks
* No repeat options
* Plain UI/ graphics
