---
layout: single
title:  "DSL for tests: before"
date:   2019-01-20
categories: programming
---
One of the reasons why I love Kotlin so much is that thanks to Kotlin we managed
to dramatically improve tests in [EduTools plugin](https://plugins.jetbrains.com/plugin/10081-edutools). 
For me writing tests turned from torture to routine stage of development.

Let me start by giving you some context about the project.

We're a plugin based on [IntelliJ Platform](https://www.jetbrains.com/opensource/idea/). 
If you'd like to learn more about how IJ is being tested, you'd better check [Dev Guide](https://www.jetbrains.org/intellij/sdk/docs/basics/testing_plugins.html),
because I'm going to really simplify things here.

Our plugin allows to take and create courses right inside an IDE.
The workflow is the following: you join a course, open a task, fill in answer 
placeholders and click "check" button. Take a look at the picture (green box in the editor - correctly
filled answer placeholder):
<img src="../../assets/images/edutools-kotlin.png">

From the plugin developer's point of view we have <code>project</code> object 
which has <code>Course</code> attribute. Using course content we perform various IDE actions.

{% highlight kotlin %}
class Course {
  val tasks: List<Task>
}

class Task {
 val files: List<EduFile>
}

class EduFile {
 val placeholders: List<Placeholder>
}

class Placeholder {
  val offset: Int
  val length: Int
  val text: String
}

class AwesomeAction: Action() {
 fun actionPerformed(event: ActionEvent) {
   val course = event.project.course
   if (course.hasAtLeastTwoTasks()) {
     showMessage("You get to solve more than two tasks!")
   }
 }
}
{% endhighlight %}

For testing we use JUnit 3, so I'll stick to it, but
testing framework details don't really matter.

Let me show you a typical test:

{% highlight kotlin %}
class TypicalTest: EduTestCase {
  private lateinit var project: Project
  override fun setUp() {
    project = createProject()  
  }
  
  fun `test typical thing`() {
    val course = createCourse()
    project.course = course
    createFileStructureForCourseOnDisk()
    doActualTest()
  }

  override fun tearDown() {
    project.close()
    project.dispose()
  }
}
{% endhighlight %}

The real pain here is hidden behind `createCourse` and `createFileStructureForCourseOnDisk()`
functions. 
Before DSL was introduced we used to have several ways of creating course object.

**Creating from json**

This way when you wanted to add a new test you had to provide a json
file describing a test course structure. These json files were stored in testData directory. Of course, you weren't supposed
to write these json files by hand: you could generate them with the help
of the plugin but you needed to actually perform actions like create new course,
add tasks, files and placeholders in the UI.

In tests we had smth like this:
{% highlight kotlin %}
fun createCourse(): Course {
  val jsonFile = getJsonFile("path to json")
  return deserializeCourse(jsonFile)  
}
{% endhighlight %}

Looks quite simple, but actually very inconvenient: you needed a running IDE
in order to generate test data, you needed to navigate to test data in order
to understand how the course in particular test looks like, you needed to 
update all the tests if some changes affecting json format were made.

**Creating from files**

This way you didn't need any json files, instead you created only one
file and a course with only one task and file was created from it.

Placeholders were specified in file in the form of tags: you just surrounded 
text with tags and then we parsed offset, length and text for a placeholder.
{% highlight kotlin %}
fun createCourse(): Course {
  val course = Course()
  val task = Task()
  course.addTask(task)
  val file = EduFile()
  file.text = setTextFromFile("path to file")
  file.placeholders = parsePlaceholders("path to file")
  task.addFile(file)
  return course  
}
{% endhighlight %}

It wasn't as painful as generating json files, but this approach has an obvious
downside: you couldn't create a course with several tasks or several files.  
So we've created several helpers that also allowed you to copy entire
directories to test project.

The last approach doesn't look that bad, but in reality you needed to call several helpers
and exchange objects between them in order to "compose" a suitable course
object for testing.

I'm going to publish "after" part of the story next week, so you'll be able to compare this approach with kotlin DSL that we eventually 
introduced and see how much visual and easy to write and read it is. 

You can read the second part [here](https://medvector.github.io/programming/dsl-after/)







