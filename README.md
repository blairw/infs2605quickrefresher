# INFS2605 Quick Refresher
This is a quick refresher of [INFS2605](https://www.business.unsw.edu.au/degrees-courses/course-outlines/INFS2605) (_Intermediate Business Programming_) for UNSW students who are taking later courses that assume INFS2605 as prerequisite knowledge, but might need some help jogging their memory. 😅

These notes will reacquaint you with some concepts that you have previously studied in depth, but these notes will not fully explain these concepts for you. For that, please refer to your notes from when you studied INFS2605.

**Contents:**

- [💻 1. Code and Version Control](#codeandversioncontrol)
- [📒 2. Data and Databases](#data)
- [🍱 3. UI and UX](#uiux)
- [☕️ 4. JavaFX](#javafx)
- [⚙️ 5. Parallel Processing](#parallelprocessing)
- [🧨 6. Errors and Exceptions](#errorsandexceptions)
- [✅ 7. Quality Assurance](#qualityassurance)
- [💭 8. Intellectual Property](#intellectualproperty)
- [💡 9. Tips and Tricks](#tipsandtricks)

_(Itemised based on concepts, not teaching weeks.)_

---

<a name="codeandversioncontrol"></a>
## 💻&nbsp;&nbsp;1. Code and Version Control

Where to store your code?
- **Your laptop:** Is vulnerable to device loss/damage and cannot facilitate team collaboration.
- **USB flash storage:** Keeping things in sync is difficult.
- **Purely online platform:** Limited functionality and performance (e.g. for developing GUI programs).
- **Cloud file sync:** Compile-time cached files can cause problems.

This is why we use Git! You just need to learn some commands:
- `git clone URL` &mdash; get a copy of an existing _repository_ (like this one!), from `URL`.
- `git pull` &mdash; get the latest updates. You should do this often.
- `git add` &mdash; make Git aware of the latest changes (this does not happen automatically).
- `git commit -avm MESSAGE` &mdash; record this set of changes as a _commit_, with a `MESSAGE`.
- `git push` &mdash; send the latest commit _upstream_.

You should also know about **commit logs** (like [this one](https://github.com/blairw/infs2605quickrefresher/commits/master)) and diffs.

<a name="data"></a>
## 📒&nbsp;&nbsp;2. Data and Databases

### Data Structures

- **Lists (e.g. ArrayLists)** are ordered, you can access any item at any time, and you can have duplicates.
- **Sets (e.g. HashSets)** are not ordered, you can access any item at any time, and duplicates are ignored.
- **Queues** are ordered but you can only work with the items on one end.
- **Deques** are like Queues, but you can work with both ends (deque = double-ended queue).

### JDBC and SQLite

JDBC is the standard technique for connecting your Java programs to databases. Databases can be _relational_ (e.g. SQL-based) or _non-relational_ (e.g. Hadoop), _client-server_ (e.g. Oracle 11g, mySQL) or _embedded_ (e.g. SQLite).

### SQLite Metadata/Create = `CREATE` statement

```java
public static void connect() throws ClassNotFoundException, SQLException {
	Connection conn = DriverManager.getConnection("jdbc:sqlite:mydatabase.db");

	Statement st = conn.createStatement();
	String createQuery = "CREATE TABLE students"
		+ "(zid INTEGER PRIMARY KEY AUTOINCREMENT"
		+ ", fname TEXT NOT NULL"
		+ ", lname TEXT NOT NULL"
		+ ", email TEXT NOT NULL"
		+ ", w1_tute_score REAL"
		+ ", w2_tute_score REAL"
		+ ", w3_tute_score REAL"
		+ ")";
	st.execute(createQuery);
	st.close();

	conn.close();
}
```

### SQLite Data/Create = `INSERT` statement(s)

```java
public static void inserts() throws SQLException {
	Connection conn = DriverManager.getConnection("jdbc:sqlite:mydatabase.db");

	Statement st = conn.createStatement();
	ArrayList<String> insertStatements = new ArrayList<String>();
	insertStatements.add(
		"INSERT INTO students (zid, fname, lname, email) "
		+ "VALUES (5000001, 'Ronald', 'Plump', 'r.plump.@notunsw.edu.au')"
	);
	insertStatements.add(
		"INSERT INTO students (zid, fname, lname, email) "
		+ "VALUES (5000002, 'Brock', 'Ohana', 'b.ohana.@notunsw.edu.au')"
	);
	for (String thisStatement : insertStatements) {
		st.execute(thisStatement);
	}
	st.close();

	conn.close();
}
```

### SQLite Data/Update = `UPDATE` statement(s)

```java
public static void updateScores() throws SQLException {
	Connection conn = DriverManager.getConnection("jdbc:sqlite:mydatabase.db");

	String myPreparedSt = "UPDATE student SET w1_tute_score = ? WHERE zid = ?";
	PreparedStatement st = conn.prepareStatement(myPreparedSt);

	int[] zids = { 5000001, 50000002 };
	double[] scores = { 0.75, 0.50 };
	for (int i = 0; i < zids.length; i++) {
		st.setDouble(1, scores[i]);
		st.setInt(2, zids[i]);
		st.executeUpdate();
	}
	st.close();

	conn.close();
}
```

_Ensure that you use `PreparedStatement` instead of `Statement` in order to sanitise your database input to protect against SQL injections. See [xkcd #327](https://xkcd.com/327/)._

### SQLite Data/Read = `SELECT` statement

```java
public static void printAll() throws SQLException {
	Connection conn = DriverManager.getConnection("jdbc:sqlite:mydatabase.db");

	Statement st = conn.createStatement();
	String selectQuery = "SELECT fname, lname, zid FROM students";
	ResultSet rs = st.executeQuery(selectQuery);
	while (rs.next()) {
		System.out.println(
			"Student " + rs.getString(1) + " "
			+ rs.getString(2) + " has zID " + rs.getString(3)
		)
	}
	st.close();

	conn.close();
}
```

### Java Dates

Convert from `String` to `LocalDate`:

```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
LocalDate myLocalDate = LocalDate.parse("23/09/2019", formatter);
```

Convert from `LocalDate` to `String`:

```java
String myDateString = myLocalDate.format(formatter);
```

### Generating UUIDs
We use UUIDs because auto-increment integer primary keys have the following drawbacks:
- Need to be centrally maintained
- Can be easily guessed
- Cannot easily handle concurrent access

You can recognise UUIDs from their distinct format, e.g.
```
E3E136E6-0F48-4946-A78E-0846A7DF1A15
28597B36-6868-41D7-AE5D-DD6DE21DF365
C13D475D-9928-4250-89A7-4C63A57148F6
```

Here's how you generate UUIDs in Java:
```java
import java.util.UUID;
public class BlairUUIDDemo {
	public static void main(String[] args) {
		for (int i = 0; i < 2605; i++) {
			UUID newUUID = UUID.randomUUID();
			System.out.println(newUUID);
		}
	}
}
```

<a name="uiux"></a>
## 🍱&nbsp;&nbsp;3. UI and UX

### Essential definitions

- **User Interface:** the part(s) of a machine that facilitate interactions with human users
- **Command Line Interface (CLI):** UI in which human users interact with the program through text commands such as `cd`, `ls`, `java`, etc.
- **Text User Interface (TUI):** UI in which text is used to generate graphical structures such as tables, menus, etc. Works with a mouse.
- **Graphical User Interface (GUI):** UI in which users interact with graphical icons and visual indicators with minimal typing for navigation.

### Heuristics

- [**Fitts’ Law:**](https://en.wikipedia.org/wiki/Fitts%27s_law#Implications_for_UI_design) Difficulty of moving quickly to a target on a screen depends on the target’s width and how far away it is.
- [**Norman's Concepts:**](https://medium.com/@sachinrekhi/don-normans-principles-of-interaction-design-51025a2c0f33) Natural mapping, perceived affordances, and feedback. But be warned:
	- Natural mapping can mutate into _skeuomorphism_.
	- Perceived affordances can mutate into _clutter_.
	- Feedback can mutate into _information overload_.
- [**Nielsen's Ten Heuristics:**](https://en.wikipedia.org/wiki/Heuristic_evaluation#Nielsen)
	- Visibility of System Status
	- Match Between System and Real World
	- User Control and Freedom
	- Consistency and Standards
	- Error Prevention
	- Recognition Rather Than Recall
	- Flexibility and Efficiency of Use
	- Aesthetic and Minimalist Design
	- Help Users Recognise, Diagnose and Recover from Errors
	- Help and Documentation
- [**Three-Click Rule:**](https://en.wikipedia.org/wiki/Three-click_rule) "a user should be able to find anything with no more than three clicks". But be warned:
	- Clicking was way worse in the old days, especially for websites, when every click resulted in a new web browser transaction (i.e. before [AJAX](https://en.wikipedia.org/wiki/Ajax_(programming))).
	- Thanks to touch screens and faster storage media, clicking is much easier than it used to be.
	- To satisfy the three-click rule, we often introduce clutter!

<a name="javafx"></a>
## ☕️&nbsp;&nbsp;4. JavaFX

JavaFX supersedes Abstract Window Toolkit (AWT) and Swing. It follows the Model-View-Controller (MVC) design pattern, consisting of:

- **Model:** Defiens data model (variables) and applies business logic (methods), e.g. `ContactPerson.java`.
- **View:** Defines layout of GUI screens, using procedural code or markup, e.g. `ContactList.fxml`.
- **Controller:** Defines the behaviour of the corresponding View, linking actions on elements with action handlers, e.g. `ContactListController.java`.

Reasons to use MVC:
- Complex program requires extensive modularisation
- Many developers using version control system like Git
- Complex business logic
- Consistent GUI template (e.g. scenes in SceneBuilder)

Reasons to use something else:
- Data model and business logic are very different &rarr; use 3-tier architecture (data, logic, presentation).
- The program is very simple &rarr; no need to modularise so much.
- The program will change a lot &rarr; MVC will require lots of refactoring across files.

For JavaFX code samples, please refer to:

- https://github.com/blairw/infs2605fxstarterkit
- https://github.com/blairw/javafx-planets-dbdemo

<a name="parallelprocessing"></a>
## ⚙️&nbsp;&nbsp;5. Parallel Processing

In the past, classic procedural programming meant writing a sequence of instructions for the computer's processor to execute one after another after another. We are no longer living in the past. We now have processors with multiple cores, multiple threads. Effective programming often means making use of multple threads to handle tasks concurrently.

Java provides classes `Thread` and `Runnable` for this. Most of the time, you want to be writing your code into a class that `implements Runnable` for dispatch onto a Thread.

Some important `Thread` methods to understand:
- `start()` - of course.
- `isAlive()` - to determine if the thread is alive (typically for synchronisation purposes)
- `interrupt()` - asks the thread to stop.
- `join()` - if thread X calls this on thread Y, i.e. in thread X we have `threadY.join()`, then it means that thread X will stop execution for now and only continue once thread Y is completed.

<a name="errorsandexceptions"></a>
## 🧨&nbsp;&nbsp;6. Errors and Exceptions

An **Exception** is an event that occurs during program execution that interrupts the program's normal flow such that we get **unusual circumstances**. Exceptions are part of a broader category of **Throwables** (disruptions to normal flow). The other kind of Throwable is an **Error** (serious problems). Exceptions are supposed to be caught and handled so that the program can recover from exceptions; however, the program is not required to be able to recover from Errors (e.g. `OutOfMemoryError`) as these are beyond our control.

If a piece of code is doing something that could generate an Exception, it either "catches" it or "throws" it.

```java
// Catching example
public void catchingExample() {
	try {
		dangerousThing();
	} catch (Exception e) {
		e.printStackTrace();
	}
}

// Throwing example
public void throwingExample() throws Exception {
	dangerousThing();
}
```

You can also nest Exceptions:

```java
public void doStuff() {
   try {
      doThing();
      try {
         doAnotherThing();
      } catch (SQLException sqle) {
         System.out.println("2nd Task Failed, keep going!");
      } finally {
         doCoolExtraThing();
      }
   } catch (Exception e) {
      System.out.println("Something went bad!");
      // report what's wrong in great detail
      e.printStackTrace();
   } finally {
      doCleanupActivities();
   }
}
```

There are two types of Exceptions:

- **Checked Exceptions:** Can be anticipated because you’re doing something risky, so the compiler checks that you have handled them (and doesn’t allow you to compile if you haven’t), e.g. `ClassNotFoundException`, `SQLException`, `IOException`.

- **Unchecked Exceptions:** Cannot be anticipated, can happen any time anywhere, so it would be unreasonable to force you to always handle it, e.g. `NullPointerException`, `IndexOutOfBoundsException`.

Exceptions are object-oriented (there's an Exception class that you could even extend if you want, just need to call `super(errorMessage)` in your constructor). All Exception objects have a `printStackTrace()` method which send diagnostic information to `System.err` (compare with `System.in`, `System.out`).


<a name="qualityassurance"></a>
## ✅&nbsp;&nbsp;7. Quality Assurance

Five defintions of quality:
- **Transcendent:** Quality is “uncompromising standards and high achievement” but “cannot be defined precisely”.
- **User-based:** Quality is how much the software satisfies users and their needs but users can’t articulate this well.
- **Manufacturers' definition:** Quality is how much the software conforms to pre-defined specifications but specifications aren’t perfect.
- **Product-based:** Quality is quantity: number of features made available to the user even the ones less-often used.
- **Value-based:** Quality is how much the software delivers value to stakeholders. In a sense, this subsumes all the above concerns.

Defects vs. Incidents:
- **Defect (bug):** An undesirable aspect of a software’s quality (IBM, 2018).
- **Incident:** An unplanned interruption to IT service or reduction in quality of an IT service (ITIL, 2011).

Different types of testing:
- **Unit Testing:** Software engineers individually testing components during the development process.
- **System Integration Testing (SIT):** Testing to ensure that different isolated components are able to work together.
- **User Acceptance Testing (UAT):** Testing to ensure that different types of users are satisfied with the product.
- **Deployment Testing:** Testing that delivery of latest release to the production environment goes smoothly.

Typical defect lifecycle:
- **Created:** Logging a problem in a defect register (usually bug-tracking software).
- **Assigned:** A team member takes responsibility of investigating the problem. As part of investigation, this team member must _triage_ the problem incl. verify that the problem can be replicated (reproduced).
- **Fixed:** A possible solution (patch, fix) has been developed and will be tested.
- **Tested:** The solution works!
- **Closed:** The solution has been applied to the codebase.


<a name="intellectualproperty"></a>
## 💭&nbsp;&nbsp;8. Intellectual Property

Once upon a time, information was excludable and rivalrous (e.g. in the form of paper books). Now information is stored and shared digitally, making it easy for non-paying users to access it (i.e., no longer easily excludable) and making it easy to distribute at almost zero cost (i.e., no longer clearly rivalrous). **Software is not immune to these issues.**

**🎁&nbsp;&nbsp;Solution #1 - Artificial Scarcity.**
- Information can be treated as excludable and rivalrous again. Traditional goods-based revenue models are protected.
- Protections are put in place for ideas (using patents), names (using trademarks), and the software code itself (using copyright).
- Famous cases include [Apple v. Microsoft](https://en.wikipedia.org/wiki/Apple_Computer,_Inc._v._Microsoft_Corp.) (1995), [Oracle v. Google](https://en.wikipedia.org/wiki/Google_v._Oracle_America) (since 2010, currently delayed due to COVID-19).
- Ongoing case: [Epic Games v. Apple](https://en.wikipedia.org/wiki/Epic_Games_v._Apple) (2020).

**🏝&nbsp;&nbsp;Solution #2 - Information as a Public Good.**
- Information treated as non-excludable and non-rival. Revenue models are based on services and other kinds of value-add on top of the public component.
- Public component is not always public domain: typically Creative Commons or open-source license (e.g. MIT, GPL, Apache, etc.).
- Examples: VLC Media Player (originally an academic project), Mozilla Firefox (originally funded by AOL, now funded by donations), and GNU/Linux (contributions from various sources).
- Java was originally conceived as open-source but has been affected by a licensing change (cf. [KTH](https://intra.kth.se/en/it/programvara-o-system/system/oracle-java-new-license-1.862964), [University of Minnesota](https://it.umn.edu/planned-changes/oracle-java-transition-project), etc.) - please use OpenJDK unless you've paid required licensing fees to Oracle.

<a name="tipsandtricks"></a>
## 💡&nbsp;&nbsp;9. Tips and Tricks

- 📝&nbsp;&nbsp;Sketch out the program before building it in JavaFX, but be aware of what JavaFX can and cannot do.
- 🏭&nbsp;&nbsp;Make use of tools like the NetBeans code generators!
- 🏗️&nbsp;&nbsp;Ensure your foundations are solid (data model, basic layouts) before playing with fun extras (CSS, etc).
- 🎀&nbsp;&nbsp;Fulfil users' underlying needs within the limits of current technology.
- 🍀&nbsp;&nbsp;Good luck with your studies!